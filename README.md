# ansible-image-builder
A collection of Ansible playbooks to manage lifecycle of qcow2 images in OpenStack using [Red Hat's Image Builder SaaS](https://console.redhat.com/insights/image-builder)

## Prerequisites
1. A user account on Red Hat's customer portal
2. An offline token, which you can generate [here](https://access.redhat.com/management/api)
3. Somewhere to run ansible-playbooks, such as Ansible Automation Platform
4. An OpenStack platform where to upload and rotate the images
5. Optionally, somewhere to store the images persistently before uploading to OpenStack

## Example playbooks
The playbooks mentioned in the following examples use the image definitions in `group_vars/all/image_definitions.yaml` to create or delete images. No image will be created, uploaded or delete if the name doesn't match the definitions.
When deleting composes from image builder, only those with an `image_name` matching the definitions will be deleted. The rest will be ignored.

It is recommended to setup these playbooks as templates in Ansible Automation Platform. Each of them require:
1. A vault with your `vault_offline_token` (and therefore a credential of type vault with the password) 
2. A credential of [type OpenStack] (https://access.redhat.com/documentation/en-us/red_hat_ansible_automation_platform/2.4/html/automation_controller_user_guide/controller-credentials#ref-controller-credential-openstack) with all required details where to upload the images (Keystone url, user, password, etc.)

Alternatively, 
1. clone this repository to your undercloud server:
```
$ git clone https://github.com/enothen/ansible-image-builder
$ cd ansible-image-builder
$ source ~/overcloudrc
```

2. Create and populate a vault (then put your vault password in a file or provide on the ansible-playbook command line):
```
$ echo 'vault_offline_token: "<your offline token here>"' > group_vars/all/vault
$ ansible-vault encrypt group_vars/all/vault
```

3. Then source the overcloud credentials:
```
$ source ~/overcloudrc 
```

### lifecycle-images.yaml
This playbook manages end-to-end lifecycle of a qcow image, from image builder to OpenStack. It will:

1. Create new images using Red Hat's image builder service
2. Wait until the creation of images is complete, then download the images.
3. Customize the image with arbirtrary tasks using `virt-customize`
4. Upload the images to OpenStack
5. Remove the older versions of the images from OpenStack

```
$ ansible-playbook lifecycle-images.yaml

PLAY [This playbook creates qcow2 image(s) using the image-builder service] **************************************

TASK [image_builder : Get refresh_token from offline_token] ******************************************************

ok: [localhost]

TASK [Check if image exists] *************************************************************************************
ok: [localhost] => (item=rhel-8.4-base-ib)
ok: [localhost] => (item=rhel-9.2-base-ib)
ok: [localhost] => (item=rhel-9-lamp-ib)

TASK [Show file status] ******************************************************************************************
ok: [localhost] => (item=/mnt/persist/images/rhel-8.4-base-ib-rel-20240430v2.qcow2) => {
    "msg": {
        "exists": false
    }
}
ok: [localhost] => (item=/mnt/persist/images/rhel-9.2-base-ib-rel-20240430v2.qcow2) => {
    "msg": {
        "exists": false
    }
}
ok: [localhost] => (item=/mnt/persist/images/rhel-9-lamp-ib-rel-20240430v2.qcow2) => {
    "msg": {
        "exists": false
    }
}

TASK [image_builder : Request creation of images] ****************************************************************
ok: [localhost] => (item=rhel-8.4-base-ib)
ok: [localhost] => (item=rhel-9.2-base-ib)
ok: [localhost] => (item=rhel-9-lamp-ib)

TASK [image_builder : Get available composes] ********************************************************************
skipping: [localhost]

TASK [image_builder : Identify required composes from the list] **************************************************
skipping: [localhost]

TASK [image_builder : Set fact with compose_request] *************************************************************
skipping: [localhost]

TASK [image_builder : Verify compose request is finished] ********************************************************
FAILED - RETRYING: Verify compose request is finished (20 retries left).
FAILED - RETRYING: Verify compose request is finished (19 retries left).
FAILED - RETRYING: Verify compose request is finished (18 retries left).
FAILED - RETRYING: Verify compose request is finished (17 retries left).
FAILED - RETRYING: Verify compose request is finished (16 retries left).
FAILED - RETRYING: Verify compose request is finished (15 retries left).
FAILED - RETRYING: Verify compose request is finished (14 retries left).
FAILED - RETRYING: Verify compose request is finished (13 retries left).
FAILED - RETRYING: Verify compose request is finished (12 retries left).
FAILED - RETRYING: Verify compose request is finished (11 retries left).
FAILED - RETRYING: Verify compose request is finished (10 retries left).
FAILED - RETRYING: Verify compose request is finished (9 retries left).
ok: [localhost] => (item=rhel-8.4-base-ib: compose_status: success, upload_status: success)
ok: [localhost] => (item=rhel-9.2-base-ib: compose_status: success, upload_status: success)
ok: [localhost] => (item=rhel-9-lamp-ib: compose_status: success, upload_status: success)


```

### list-distributions.yaml
Image builder allows building images with speciifc minor OS versions, such as RHEL 8.4, or 9.2, but these options are not listed on the web interface. This playbook uses one of the API methods to list all possible options.

```
$ ansible-playbook list-distributions.yaml 

PLAY [Playbook to list available distributions in image builder] *************************************************

TASK [image_builder : Get refresh_token from offline_token] ******************************************************
ok: [localhost]

TASK [image_builder : Get available distributions] ***************************************************************
ok: [localhost]

TASK [Show available distributions] ******************************************************************************
ok: [localhost] => {
    "distributions.json | sort(attribute='name')": [
        {
            "description": "CentOS Stream 8",
            "name": "centos-8"
        },
        {
            "description": "CentOS Stream 9",
            "name": "centos-9"
        },
        {
            "description": "Red Hat Enterprise Linux (RHEL) 8",
            "name": "rhel-8"
        },
        {
            "description": "Red Hat Enterprise Linux (RHEL) 8",
            "name": "rhel-8.10"
        },
        {
            "description": "Red Hat Enterprise Linux (RHEL) 8",
            "name": "rhel-84"
        },
        {
            "description": "Red Hat Enterprise Linux (RHEL) 8",
            "name": "rhel-85"
        },
        {
            "description": "Red Hat Enterprise Linux (RHEL) 8",
            "name": "rhel-86"
        },
        {
            "description": "Red Hat Enterprise Linux (RHEL) 8",
            "name": "rhel-87"
        },
        {
            "description": "Red Hat Enterprise Linux (RHEL) 8",
            "name": "rhel-88"
        },
        {
            "description": "Red Hat Enterprise Linux (RHEL) 8",
            "name": "rhel-89"
        },
        {
            "description": "Red Hat Enterprise Linux (RHEL) 9",
            "name": "rhel-9"
        },
        {
            "description": "Red Hat Enterprise Linux (RHEL) 9",
            "name": "rhel-90"
        },
        {
            "description": "Red Hat Enterprise Linux (RHEL) 9",
            "name": "rhel-91"
        },
        {
            "description": "Red Hat Enterprise Linux (RHEL) 9",
            "name": "rhel-92"
        },
        {
            "description": "Red Hat Enterprise Linux (RHEL) 9",
            "name": "rhel-93"
        },
        {
            "description": "Red Hat Enterprise Linux (RHEL) 9",
            "name": "rhel-94"
        }
    ]
}

PLAY RECAP *******************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### list-composes.yaml
This playbook lists basic information of all available composes on the organization of the Red Hat account associated to the offline token, whether they match the definitions in this directory or not.

```
$ ansible-playbook list-composes.yaml 

PLAY [Playbook to list available composes in image builder] ******************************************************

TASK [image_builder : Get refresh_token from offline_token] ******************************************************
ok: [localhost]

TASK [image_builder : Get available composes] ********************************************************************
ok: [localhost]

TASK [List available composes] ***********************************************************************************
ok: [localhost] => (item=id: bc2f7700-bbd6-48ab-a423-5964e147cafc) => {
    "msg": [
        "created_at: 2024-05-31T19:04:40Z",
        "image_name: rhel-9-lamp-ib",
        "image_description: rel-20240430v2",
        "distribution: rhel-9"
    ]
}
ok: [localhost] => (item=id: 6f618bd5-dbfc-4ebf-979b-86a0ed728ac6) => {
    "msg": [
        "created_at: 2024-05-31T19:04:40Z",
        "image_name: rhel-9.2-base-ib",
        "image_description: rel-20240430v2",
        "distribution: rhel-92"
    ]
}
ok: [localhost] => (item=id: 00e457d5-5788-449c-a153-f74116c0240e) => {
    "msg": [
        "created_at: 2024-05-31T19:04:39Z",
        "image_name: rhel-8.4-base-ib",
        "image_description: rel-20240430v2",
        "distribution: rhel-84"
    ]
}
ok: [localhost] => (item=id: 619bd633-7261-412a-9157-91000d54f5cb) => {
    "msg": [
        "created_at: 2024-05-30T13:13:34Z",
        "image_name: No name",
        "image_description: No description",
        "distribution: rhel-94"
    ]
}
ok: [localhost] => (item=id: 22e98230-b276-4b3c-9248-1b6feccb322d) => {
    "msg": [
        "created_at: 2024-05-30T12:47:22Z",
        "image_name: No name",
        "image_description: No description",
        "distribution: rhel-91"
    ]
}
ok: [localhost] => (item=id: b3bc6771-6374-4057-8d70-0004f301136e) => {
    "msg": [
        "created_at: 2024-05-30T12:10:23Z",
        "image_name: No name",
        "image_description: No description",
        "distribution: rhel-94"
    ]
}
ok: [localhost] => (item=id: 25b7a5f5-587c-4c92-a981-f655a8a187f4) => {
    "msg": [
        "created_at: 2024-05-24T06:55:05Z",
        "image_name: test-test",
        "image_description: ",
        "distribution: rhel-89"
    ]
}
ok: [localhost] => (item=id: 5c171c5c-1f46-4198-9a31-c92dae4bdca6) => {
    "msg": [
        "created_at: 2024-05-19T05:27:47Z",
        "image_name: testrh9",
        "image_description: testrh9",
        "distribution: rhel-94"
    ]
}
ok: [localhost] => (item=id: 33633066-1fae-4a59-8a83-7580210e02e8) => {
    "msg": [
        "created_at: 2024-05-18T09:33:16Z",
        "image_name: ableisch-wsl-test",
        "image_description: RHEL8 image to be used for WSL",
        "distribution: rhel-89"
    ]
}

PLAY RECAP *******************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### cleanup-composes.yaml
This playbook will delete compose requests in image builder, where the `image_name` matches the definitions. Note that other people's composes, where the image name matches to definitions in this directory, will be deleted.

```

```
