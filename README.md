# enothen.image-builder
A collection to manage virtualization images using declarative image definitions through the [API interface](https://developers.redhat.com/api-catalog/api/image-builder) of Red Hat's [Insights Image Builder](https://console.redhat.com/insights/image-builder).

- [enothen.image-builder](#enothenimage-builder)
  - [Prerequisites](#prerequisites)
  - [Configuration](#configuration)
  - [Image types](#image-types)
    - [Downloadable images](#downloadable-images)
    - [Shareable images](#shareable-images)
  - [Creating image definitions](#creating-image-definitions)
    - [Image Builder customizations](#image-builder-customizations)
      - [Repositories](#repositories)
      - [Filesystems](#filesystems)
      - [Directories](#directories)
      - [Firewall](#firewall)
      - [Restrictions by image\_type](#restrictions-by-image_type)
    - [Sharing images with cloud providers](#sharing-images-with-cloud-providers)
      - [Amazon Web Services](#amazon-web-services)
      - [Azure](#azure)
      - [Google Cloud Platform](#google-cloud-platform)
      - [Oracle Cloud Infrastructure](#oracle-cloud-infrastructure)
    - [OpenScap profiles](#openscap-profiles)
  - [Setting a release id](#setting-a-release-id)


## Prerequisites
To use this collection you need:
1. A user account on Red Hat's customer portal
2. An offline token, which you can generate [here](https://access.redhat.com/management/api)

## Configuration
1. Install the collection:
```
$ ansible-galaxy collection install enothen.image_builder
```

2. Create and populate a vault (then put your vault password in a file or provide it on the ansible-playbook command line):
```
$ echo 'vault_offline_token: "<your offline token here>"' > group_vars/all/vault
$ ansible-vault encrypt group_vars/all/vault
```

## Image types
Insights image builder supports different types of images, which can be classified in two groups: the ones you can download, and the ones you can push to public cloud providers.
You can use `ansible-image-builder` to build all of the image types supported by Insights Image Builder.

### Downloadable images
These types of images are copied to an AWS S3 bucket after the build completes successfully, so that they can be downloaded to be used in private clouds, virtualization environments, etc.
- edge-commit
- edge-installer
- guest-image
- rhel-edge-commit
- rhel-edge-installer

The actual file format depends on the image type. For example, the `guest-image` type will create a `.qcow2` file, and the `vsphere` type will create a `.vmdk` file. The list of all extensions matching the image types are defined [here](https://github.com/enothen/ansible-image-builder/blob/main/roles/download_images/vars/main.yml#L8-L18).

### Shareable images
These types of images are built for a public cloud provider and shared with specified tenants or accounts after the build completes successfully, which means the image definition may require parameters such as tenant, subscription, resource group, etc. See [example/definitions/main.yaml](https://github.com/enothen/ansible-image-builder/blob/main/example/definitions/main.yaml) for more details.
- ami
- aws
- azure
- gcp
- oci
- vhd

## Creating image definitions
Create image definitions in yaml, where the `images` variable contains an array of images to manage. Each of the entries is a dictionary with the following format:

```
  - name: String (max 100 char, required)
    distribution: String, limited to specific options, required
    description: String (max 250 char), optional
    customizations: Object<Customizations>, optional
```

The schema of the ComposeRequest and Customizations objects, as well as the rest of all possible customizations are defined [here](https://developers.redhat.com/api-catalog/api/image-builder#content-schemas).

Unless overriden in the image definition, the `image_type` defaults to `guest-image` and the `architecture` to `x86_64`.

### Image Builder customizations
There are multiple customization methods available, most of them documented in the [example definitions](https://github.com/enothen/ansible-image-builder/blob/main/example/definitions/). 
Here's a brief definition example: a RHEL 9.2 image that installs Apache, MariaDB and PHP, customizing firewall, services and filesystems:
```
images:
  - name: rhel-9.2-lamp
    distribution: rhel-92
    description: "RHEL 9.2 with Apache, MariaDB and PHP"
    customizations:
      files:
        - path: /etc/sudoers.d/dbagroup
          mode: '0600'
          user: root
          group: root
          data: |
            # Sudo rules for database administrators
            %dbas ALL= /usr/bin/systemctl start mysqld.service
            %dbas ALL= /usr/bin/systemctl stop mysqld.service
            %dbas ALL= /usr/bin/systemctl restart mysqld.service
            %dbas ALL= /usr/bin/systemctl reload mysqld.service
      firewall:
        services:
          enabled:
            - ssh
            - http
            - https
      filesystems:
        - mountpoint: /var/www/html
          min_size: 2048
        - mountpoint: /var/lib/mysql
          min_size: 2048
      packages:
        - httpd
        - mariadb-server
        - php
        - php-mysqlnd
      services:
        enabled:
          - firewalld
          - httpd
          - mariadb
```
See more advanced example definitions, including other image formats, architectures and customizations in [example definitions/](https://github.com/enothen/ansible-image-builder/blob/main/example/definitions/).

Some customization methods may have their own restrictions, limitations or dependencies. Some notes are provided below for awareness.

#### Repositories
You may want to use custom content in yout images, either during the build of the image or afterwards. For this, you can use one or both of the following customization methods:

- `payload_repositories`: A list of repositories to use during the image build. Use this to install custom packages (not available in Red Hat repositories).
- `custom_repositories`: A list of repositories to be defined inside the image, but not defined or used during the image build. This makes it possible to define an internal repository that is not available to image builder, but is available internally once an instance has been created from the image.

Here's a minimal example that uses both customization methods:
```
customizations:
  packages:
    - <name of rpm in custom repository>
  payload_repositories:
    - id: <id of the custom repository in the hybrid cloud console>
      check_gpg: <True|False>
      check_repo_gpg: <True|False>
      rhsm: false
  custom_repositories:
    - name: <name of the custom, internal repository in the target environment where the image is going to be used>
      id: <id of the custom repository>
      baseurl:
        - <the URL of the repository>
      filename: <name of the file to create in /etc/yum.repos.d>
      enabled: <True|False>
      check_gpg: <True|False>
```

#### Filesystems
You can define a list of filesystems to create in the image, providing the mountpoint and the size like this:
```
customizations:
  filesystem:
    - mountpoint: <where the filesystem is mounted>
      min_size: <size of the filesystem in MB>
```

Note, however, that ISO and OSTree images don't support filesystems. If you have a customization of type `filesystem` and a type of `edge-installer` or `edge-commit`, the compose request is going to be rejected. See valid image types for Insights Image Builder [here](https://developers.redhat.com/api-catalog/api/image-builder#schema-ImageTypes), and documentation of image types [here](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html-single/composing_a_customized_rhel_system_image/index#specifying-a-custom-filesystem-configuration_creating-system-images-with-composer-command-line-interface).

#### Directories
You can define new directories or directory structures and define the user, password and mode of the directory, as long as the directory's path is under /etc. If you add directories outside of /etc, the image build request is going to be accepted, but the build is going to fail. This is documented [here](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/composing_a_customized_rhel_system_image/creating-system-images-with-composer-command-line-interface_composing-a-customized-rhel-system-image#specifying_customized_directories_in_the_blueprint).

#### Firewall
Sligthly different to what the `firewall-cmd --add-port` command expects, adding ports to the firewall in image builder is done with the `:` separator, therefore a list of `<port>:<protocol>` is required. This is documented [here](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/composing_a_customized_rhel_system_image/creating-system-images-with-composer-command-line-interface_composing-a-customized-rhel-system-image#customizing-firewall_creating-system-images-with-composer-command-line-interface).

When adding any customization of type firewall (either ports or services), the firewalld package must also be explicitly added to the list of packages to install, or the image build would fail because the command `firewall-offline-cmd` is not available on the image. Idealy, you would also list `firewalld` on the enabled section of the services customization, in order for the service to start at boot.

#### Restrictions by image_type
Some customization methods are not supported in combination with specific image types. For example, the image types `rhel-edge-commit` and `rhel-edge-installer` don't support the `kernel` customization method.

### Sharing images with cloud providers
Insights image builder can share images with cloud providers after a successfull build, provided the yaml definition has all required fields. See minimal requirements below, or complete definitions in [example/definitions/public-cloud.yaml](https://github.com/enothen/ansible-image-builder/blob/main/example/definitions/public-cloud.yaml).
This is also documented in detail [here](https://docs.redhat.com/en/documentation/red_hat_insights/1-latest/html-single/deploying_and_managing_rhel_systems_in_hybrid_clouds/index#assembly_launching-customized-rhel-images-to-the-cloud-platforms-with-image-builder_host-management-services).

#### Amazon Web Services
Images shared with AWS require the definition to include the following fields:
```
    requests:
      image_type: <ami|aws>
      upload_request:
        type: aws
        options:
          share_with_accounts:
            - <AWS Account ID (12 digits)>
```
Alternatively, if the account is already configured as a source in the hybrid cloud console, you can use the source id and `share_with_sources` instead of `share_with_accounts`:
```
    requests:
      image_type: <ami|aws>
      upload_request:
        type: aws
        options:
          share_with_sources:
            - <Source ID (6 digits)>
```
To find the source ID, in the hybrid cloud console navigate to Integrations > Cloud, select filter `Type` and `Amazon Web Services`, then click on the name of the source you are looking for and the id will be shown in the url as `.../integrations/detail/<source id>?...`.

#### Azure
Images shared with Azure require the definition to include the following fields:
```
    requests:
      image_type: <azure|vhd>
      upload_request:
        type: azure
        options:
          tenant_id: <Your tenant ID>
          resource_group: <Your resource group>
          subscription_id: <Your subscription ID>
          hyper_v_generation: <V1 (BIOS) | V2 (UEFI)>
```
Note that you will have to authorize Image Builder to push images to the resource group, or the image build will fail. Example using the az cli:
```
$ az role assignment create \
    --assignee df4c7ed6-d52b-4995-b6b7-7506f4c9c051 \
    --role Contributor \
    --scope /subscriptions/<Your subscription ID>/resourceGroups/<Your resource group>
```

#### Google Cloud Platform
To build a GCP image and share it with a Google account, add the following block in the image definition:
```
    requests:
      image_type: gcp
      upload_request:
        type: gcp
        options:
          share_with_accounts:
            - "user:account@domain.com"
```
Alternatively, to build a GCP image that is shared with Red Hat Insights only, remove the `options` key, and the whole structure below it.

#### Oracle Cloud Infrastructure
To build an OCI image use the following structure:
```
    requests:
      image_type: oci
      upload_request:
        type: oci.objectstorage
```
When the build finishes, a link and instructions are provided on the Hybrid Cloud Console to import the image in OCI.

### OpenScap profiles
Insights Image Builder support many OpenScap profiles. Each of these profiles will contain a number of customizations, such as filesystems, packages, services, etc. To see the customizations required to comply with each OpenScap profiles, see the examples in the file [example/definitions/openscap.yaml](https://github.com/enothen/ansible-image-builder/blob/main/example/definitions/openscap.yaml).

## Setting a release id
The example in this repository uses event information from Github in order to identify the pull request that triggers an image build. If you are running the image lifecycle playbook from command line, unset `release_id` from the images definitions file so that a timestamp is used instead.
