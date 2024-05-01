# ansible-image-builder
A collection of Ansible playbooks to manage lifecycle of qcow2 images in OpenStack.

## Prerequisites
Somewhere to run ansible-playbooks, such as Ansible Automation Platform, and an OpenStack platform where to upload and rotate the images.

## Example playbooks

### lifecycle-images.yaml
Will use the image definitions in `group_vars/all/image_definitions.yaml` to:

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
