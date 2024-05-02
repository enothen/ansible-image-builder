# ansible-image-builder
A collection of Ansible playbooks to manage lifecycle of qcow2 images in OpenStack.

## Prerequisites
Somewhere to run ansible-playbooks, such as Ansible Automation Platform, and an OpenStack platform where to upload and rotate the images.

## Example playbooks
The playbooks mentioned in the following examples use the image definitions in `group_vars/all/image_definitions.yaml` to create or delete images. No image will be created, uploaded or delete if the name doesn't match the definitions.
When deleting composes from image builder, only those with an `image_name` matching the definitions will be deleted. The rest will be ignored.

### lifecycle-images.yaml
This playbook manages end-to-end lifecycle of a qcow image, from image builder to OpenStack. It will:

1. Create new images using Red Hat's [image builder](https://console.redhat.com/insights/image-builder) service

2. Wait until the creation of images is complete, then download the images.

3. Upload the images to OpenStack based: Use the [OpenStack credential type](https://access.redhat.com/documentation/en-us/red_hat_ansible_automation_platform/2.4/html/automation_controller_user_guide/controller-credentials#ref-controller-credential-openstack) in Ansible Automation Platform for this.

4. Remove the older versions of the images from OpenStack

```
$ ansible-playbook lifecycle-images.yaml

PLAY [This playbook creates qcow2 image(s) using the image-builder service] ****

TASK [image_builder : Get refresh_token from offline_token] ********************
ok: [localhost]

TASK [Check if image exists] ***************************************************
ok: [localhost] => (item=rhel-8.4-base-ib)
ok: [localhost] => (item=rhel-9.2-base-ib)
ok: [localhost] => (item=rhel-9-lamp-ib)

TASK [Show file status] ********************************************************
ok: [localhost] => (item=./rhel-8.4-base-ib-rel-20240430v2.qcow2) => {
    "msg": {
        "exists": false
    }
}
ok: [localhost] => (item=./rhel-9.2-base-ib-rel-20240430v2.qcow2) => {
    "msg": {
        "exists": false
    }
}
ok: [localhost] => (item=./rhel-9-lamp-ib-rel-20240430v2.qcow2) => {
    "msg": {
        "exists": false
    }
}

TASK [image_builder : Request creation of images] ******************************
ok: [localhost] => (item=rhel-8.4-base-ib)
ok: [localhost] => (item=rhel-9.2-base-ib)
ok: [localhost] => (item=rhel-9-lamp-ib)

TASK [image_builder : Get available composes] **********************************
skipping: [localhost]

TASK [image_builder : Identify required composes from the list] ****************
skipping: [localhost]

TASK [image_builder : Set fact with compose_request] ***************************
skipping: [localhost]

TASK [image_builder : Verify compose request is finished] **********************
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

TASK [image_builder : Download images] *****************************************
changed: [localhost] => (item=./rhel-8.4-base-ib-rel-20240430v2.qcow2)
changed: [localhost] => (item=./rhel-9.2-base-ib-rel-20240430v2.qcow2)
changed: [localhost] => (item=./rhel-9-lamp-ib-rel-20240430v2.qcow2)

TASK [qcow_image : Upload images to rhosp] *************************************
changed: [localhost] => (item=rhel-8.4-base-ib)
changed: [localhost] => (item=rhel-9.2-base-ib)
changed: [localhost] => (item=rhel-9-lamp-ib)

TASK [qcow_image : Get current list of images] *********************************
ok: [localhost] => (item=rhel-8.4-base-ib)
ok: [localhost] => (item=rhel-9.2-base-ib)
ok: [localhost] => (item=rhel-9-lamp-ib)

TASK [qcow_image : Delete older image versions] ********************************
skipping: [localhost] => (item=rhel-8.4-base-ib, 5ca06b32-4367-48cd-a03c-622985ca5fe8) 
changed: [localhost] => (item=rhel-8.4-base-ib, acd5845f-7272-4b85-9805-eb2c987fba04)
skipping: [localhost] => (item=rhel-9.2-base-ib, 1a289610-6d67-4e0b-aba1-5ec18797e9b2) 
changed: [localhost] => (item=rhel-9.2-base-ib, c6dff39e-76f3-404a-8037-944cbb714799)
skipping: [localhost] => (item=rhel-9-lamp-ib, 008b3291-091c-45b7-a6c8-3a83aa5ba141) 
changed: [localhost] => (item=rhel-9-lamp-ib, 1baa67d0-59e0-4849-9fd4-ebf52b6f66c9)

PLAY RECAP *********************************************************************
localhost                  : ok=9    changed=3    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
```

### cleanup-composes.yaml
This playbook will delete compose requests in image builder, where the `image_name` matches the definitions.

```
PLAY [Playbook to delete old composes from image builder] **********************

TASK [image_builder : Get refresh_token from offline_token] ********************
ok: [localhost]

TASK [image_builder : Get available composes] **********************************
ok: [localhost]

TASK [Deleting these composes] *************************************************
skipping: [localhost] => (item=253bda2a-56f5-4c76-bf64-cb36d4266e01) 
ok: [localhost] => (item=008b3291-091c-45b7-a6c8-3a83aa5ba141) => {
    "msg": "rhel-9-lamp-ib"
}
ok: [localhost] => (item=1a289610-6d67-4e0b-aba1-5ec18797e9b2) => {
    "msg": "rhel-9.2-base-ib"
}
ok: [localhost] => (item=5ca06b32-4367-48cd-a03c-622985ca5fe8) => {
    "msg": "rhel-8.4-base-ib"
}
skipping: [localhost] => (item=766388f9-7454-458b-bdfb-b886803c2534) 
skipping: [localhost] => (item=51253728-5c73-4080-a342-a23ec43bd7fd) 
skipping: [localhost] => (item=c8602a4f-43a5-4a29-b719-5f2e0adceeb7) 
skipping: [localhost] => (item=e3b7bf4b-536a-4b37-aa00-958e3425a866) 
skipping: [localhost] => (item=893eddfd-6b27-41db-9f69-e238a3f7dc3d) 
skipping: [localhost] => (item=7d39d75c-f035-4fbc-a0f6-ab1b3c5ad279) 
skipping: [localhost] => (item=e993885f-5db5-439c-84b1-8125c5610b53) 
skipping: [localhost] => (item=8b8b23e8-6b1f-4902-b3a6-315e039478c7) 
skipping: [localhost] => (item=abf21d44-8c44-44c1-9169-b1e6dd8008fc) 
skipping: [localhost] => (item=c658ad17-bc22-4143-8c0d-e0140187d447) 
skipping: [localhost] => (item=031ab188-1f63-4139-946c-c3d914fb81a6) 
skipping: [localhost] => (item=b78182d7-1590-46a7-8ccd-7b130e42fd0a) 
skipping: [localhost] => (item=7cd9cc6f-8193-49f7-b2ff-eeef45123f0c) 

TASK [Delete compose] **********************************************************
skipping: [localhost] => (item=253bda2a-56f5-4c76-bf64-cb36d4266e01) 
ok: [localhost] => (item=008b3291-091c-45b7-a6c8-3a83aa5ba141)
ok: [localhost] => (item=1a289610-6d67-4e0b-aba1-5ec18797e9b2)
ok: [localhost] => (item=5ca06b32-4367-48cd-a03c-622985ca5fe8)
skipping: [localhost] => (item=766388f9-7454-458b-bdfb-b886803c2534) 
skipping: [localhost] => (item=51253728-5c73-4080-a342-a23ec43bd7fd) 
skipping: [localhost] => (item=c8602a4f-43a5-4a29-b719-5f2e0adceeb7) 
skipping: [localhost] => (item=e3b7bf4b-536a-4b37-aa00-958e3425a866) 
skipping: [localhost] => (item=893eddfd-6b27-41db-9f69-e238a3f7dc3d) 
skipping: [localhost] => (item=7d39d75c-f035-4fbc-a0f6-ab1b3c5ad279) 
skipping: [localhost] => (item=e993885f-5db5-439c-84b1-8125c5610b53) 
skipping: [localhost] => (item=8b8b23e8-6b1f-4902-b3a6-315e039478c7) 
skipping: [localhost] => (item=abf21d44-8c44-44c1-9169-b1e6dd8008fc) 
skipping: [localhost] => (item=c658ad17-bc22-4143-8c0d-e0140187d447) 
skipping: [localhost] => (item=031ab188-1f63-4139-946c-c3d914fb81a6) 
skipping: [localhost] => (item=b78182d7-1590-46a7-8ccd-7b130e42fd0a) 
skipping: [localhost] => (item=7cd9cc6f-8193-49f7-b2ff-eeef45123f0c) 

PLAY RECAP *********************************************************************
localhost                  : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```