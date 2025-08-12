# Example playbooks
The playbooks in this repository use the variable `images` (list). No compose request will be created if the images variable is undefined or empty. No compose will be deleted if it's name doesn't match the definitions in yaml. 

## Playbooks to run from command line or ansible-navigator
### create-and-download.yaml
The easiest and quickest way to request image builds, wait for them to finish and download to current directory (or where defined by `storage_dir`).
```
$ ansible-playbook example/playbooks/create-and-download.yaml

PLAY [Create and download images at once] ****************************************************************

TASK [enothen.image_builder.check_images_exist : Set fact with release_str if release_id is set] *********
skipping: [localhost]

TASK [enothen.image_builder.check_images_exist : Check if image exists] **********************************
ok: [localhost] => (item=./rhel-9-ib.qcow2)
ok: [localhost] => (item=./rhel-10-ib.qcow2)

TASK [enothen.image_builder.check_images_exist : Show file status] ***************************************
ok: [localhost] => (item=./rhel-9-ib.qcow2) => {
    "msg": "File exists: False"
}
ok: [localhost] => (item=./rhel-10-ib.qcow2) => {
    "msg": "File exists: False"
}

TASK [enothen.image_builder.get_refresh_token : Get refresh_token from offline_token] ********************
ok: [localhost]

TASK [enothen.image_builder.request_image_creation : Request creation of images] *************************
ok: [localhost] => (item=rhel-9-ib)
ok: [localhost] => (item=rhel-10-ib)

TASK [enothen.image_builder.verify_compose_finished : Set retry counter] *********************************
ok: [localhost]

TASK [Get compose requests ids if undefined] *************************************************************
skipping: [localhost]

TASK [enothen.image_builder.verify_compose_finished : Verify compose request is finished] ****************
FAILED - RETRYING: [localhost]: Verify compose request is finished (15 retries left).
FAILED - RETRYING: [localhost]: Verify compose request is finished (14 retries left).
FAILED - RETRYING: [localhost]: Verify compose request is finished (13 retries left).
FAILED - RETRYING: [localhost]: Verify compose request is finished (12 retries left).
FAILED - RETRYING: [localhost]: Verify compose request is finished (11 retries left).
FAILED - RETRYING: [localhost]: Verify compose request is finished (10 retries left).
FAILED - RETRYING: [localhost]: Verify compose request is finished (9 retries left).
FAILED - RETRYING: [localhost]: Verify compose request is finished (8 retries left).
FAILED - RETRYING: [localhost]: Verify compose request is finished (7 retries left).
FAILED - RETRYING: [localhost]: Verify compose request is finished (6 retries left).
FAILED - RETRYING: [localhost]: Verify compose request is finished (5 retries left).
ok: [localhost] => (item=rhel-9-ib: compose_status: success, upload_status: success)
ok: [localhost] => (item=rhel-10-ib: compose_status: success, upload_status: success)

TASK [enothen.image_builder.download_images : Set fact with release_str if release_id is set] ************
skipping: [localhost]

TASK [enothen.image_builder.download_images : Start image download] **************************************
changed: [localhost] => (item=./rhel-9-ib.qcow2)
changed: [localhost] => (item=./rhel-10-ib.qcow2)

TASK [enothen.image_builder.download_images : Confirm image download completed] **************************
FAILED - RETRYING: [localhost]: Confirm image download completed (20 retries left).
FAILED - RETRYING: [localhost]: Confirm image download completed (19 retries left).
FAILED - RETRYING: [localhost]: Confirm image download completed (18 retries left).
FAILED - RETRYING: [localhost]: Confirm image download completed (17 retries left).
changed: [localhost] => (item=rhel-9-ib)
changed: [localhost] => (item=rhel-10-ib)

PLAY RECAP ***********************************************************************************************
localhost          : ok=8    changed=2    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   

$
```

### list-distributions.yaml
Insights Image builder allows building images with speciifc minor OS versions, such as RHEL 8.4, or 9.2, but these options are not listed on the web interface. This playbook uses one of the API methods to list all possible options.
```
$ ansible-playbook example/playbooks/list-distributions.yaml 

PLAY [Playbook to list available distributions in Red Hat's image builder service] ***********************

TASK [enothen.image_builder.get_refresh_token : Get refresh_token from offline_token] ********************
ok: [localhost]

TASK [enothen.image_builder.get_distributions : Get available distributions] *****************************
ok: [localhost]

TASK [Show available distributions] **********************************************************************
ok: [localhost] => {
    "distributions.json | sort(attribute='name')": [
        {
            "description": "CentOS Stream 10",
            "name": "centos-10"
        },
        {
            "description": "CentOS Stream 9",
            "name": "centos-9"
        },
        {
            "description": "Red Hat Enterprise Linux (RHEL) 10",
            "name": "rhel-10"
        },
        {
            "description": "Red Hat Enterprise Linux (RHEL) 10 Beta",
            "name": "rhel-10-beta"
        },
        {
            "description": "Red Hat Enterprise Linux (RHEL) 10",
            "name": "rhel-10.0"
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
            "description": "Red Hat Enterprise Linux (RHEL) 9 beta",
            "name": "rhel-9-beta"
        },
        {
            "description": "Red Hat Enterprise Linux (RHEL) 9",
            "name": "rhel-9.6"
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
        },
        {
            "description": "Red Hat Enterprise Linux (RHEL) 9",
            "name": "rhel-95"
        }
    ]
}

PLAY RECAP ***********************************************************************************************
localhost          : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

$ 
```

### list-composes.yaml
This playbook lists basic information of all available composes on the organization of the Red Hat account associated to the offline token, whether they match the definitions in this directory or not.
```
$ ansible-playbook example/playbooks/list-composes.yaml 

PLAY [Playbook to list available composes in Red Hat's image builder service] ****************************

TASK [enothen.image_builder.get_refresh_token : Get refresh_token from offline_token] ********************
ok: [localhost]

TASK [enothen.image_builder.get_composes : Get available composes] ***************************************
ok: [localhost]

TASK [enothen.image_builder.get_composes : Identify required composes from the list] *********************
skipping: [localhost] => (item=f740bb5a-060c-4215-afff-11c2cf1e4194) 
ok: [localhost] => (item=aab72afb-067f-4eb2-8539-9b8d12e5be39)
skipping: [localhost] => (item=19e377a9-d5e4-4a16-bb0a-035d94f5f243) 
skipping: [localhost] => (item=5f200a90-0f9a-4644-b9ea-9b740c319c94) 
skipping: [localhost] => (item=bb43ebc7-7868-4d4b-8e69-9ed7d5ea1b83) 
ok: [localhost] => (item=f740bb5a-060c-4215-afff-11c2cf1e4194)
skipping: [localhost] => (item=aab72afb-067f-4eb2-8539-9b8d12e5be39) 
skipping: [localhost] => (item=19e377a9-d5e4-4a16-bb0a-035d94f5f243) 
skipping: [localhost] => (item=5f200a90-0f9a-4644-b9ea-9b740c319c94) 
skipping: [localhost] => (item=bb43ebc7-7868-4d4b-8e69-9ed7d5ea1b83) 

TASK [enothen.image_builder.get_composes : Set fact with compose_request] ********************************
ok: [localhost]

TASK [enothen.image_builder.list_composes : List composes] ***********************************************
ok: [localhost] => (item=id: f740bb5a-060c-4215-afff-11c2cf1e4194) => {
    "msg": [
        "created_at: 2025-08-12T09:19:30Z",
        "image_name: rhel-10-ib",
        "image_description: No description",
        "distribution: rhel-10"
    ]
}
ok: [localhost] => (item=id: aab72afb-067f-4eb2-8539-9b8d12e5be39) => {
    "msg": [
        "created_at: 2025-08-12T09:19:29Z",
        "image_name: rhel-9-ib",
        "image_description: No description",
        "distribution: rhel-9"
    ]
}
ok: [localhost] => (item=id: 19e377a9-d5e4-4a16-bb0a-035d94f5f243) => {
    "msg": [
        "created_at: 2025-08-11T22:41:58Z",
        "image_name: rhel-9-JIJI1",
        "image_description: ",
        "distribution: rhel-9"
    ]
}
ok: [localhost] => (item=id: 5f200a90-0f9a-4644-b9ea-9b740c319c94) => {
    "msg": [
        "created_at: 2025-08-11T22:19:51Z",
        "image_name: rhel-9-JIJI2",
        "image_description: ",
        "distribution: rhel-9"
    ]
}
ok: [localhost] => (item=id: bb43ebc7-7868-4d4b-8e69-9ed7d5ea1b83) => {
    "msg": [
        "created_at: 2025-08-11T22:14:13Z",
        "image_name: rhel-9-JIJI1",
        "image_description: ",
        "distribution: rhel-9"
    ]
}

PLAY RECAP ***********************************************************************************************
localhost          : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

$ 
```

### get-compose-details.yaml
Sometimes you just want to get the details of an existing compose. This is similar to the `Download compose request (json)` option on the web interface, but it shows the output on the screen rather than creating a separate file. Either provide the compose id as extra var, or use the var prompt.
```
$ ansible-playbook get-compose-details.yaml -e compose=e74eb649-20c9-49fa-80f4-0cc4eeb36ebb

PLAY [Playbook to show details about a specific compose] *********************************************************

TASK [image_builder : Get refresh_token from offline_token] ******************************************************
ok: [localhost]

TASK [Get compose details] ***************************************************************************************
ok: [localhost]

TASK [Debug compose details] *************************************************************************************
ok: [localhost] => {
    "compose_details.json": {
        "image_status": {
            "status": "building"
        },
        "request": {
            "client_id": "api",
            "customizations": {
                "filesystem": [
                    {
                        "min_size": 1024,
                        "mountpoint": "/boot"
                    },
                    {
                        "min_size": 10240,
                        "mountpoint": "/"
                    }
                ],
                "kernel": {
                    "append": "rd.auto"
                },
                "packages": [
                    "lvm2",
                    "mdadm",
                    "tar",
                    "bzip2"
                ]
            },
            "distribution": "rhel-9",
            "image_description": "RHEL 9.latest image customized for OCP4 on Hetzner",
            "image_name": "rhel9-hetzner-ocp4",
            "image_requests": [
                {
                    "architecture": "x86_64",
                    "image_type": "image-installer",
                    "upload_request": {
                        "options": {},
                        "type": "aws.s3"
                    }
                }
            ]
        }
    }
}

PLAY RECAP *******************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### cleanup-composes.yaml
This playbook will delete compose requests in Insights Image Builder, where the `image_name` matches the definitions. Note that other people's composes, where their image name matches your definitions in this directory, will be deleted as well.
```
$ ansible-playbook example/playbooks/cleanup-composes.yaml 

PLAY [Playbook to delete old compose requests from Red Hat Insights Image Builder] ***********************

TASK [enothen.image_builder.get_refresh_token : Get refresh_token from offline_token] ********************
ok: [localhost]

TASK [enothen.image_builder.get_composes : Get available composes] ***************************************
ok: [localhost]

TASK [enothen.image_builder.get_composes : Identify required composes from the list] *********************
skipping: [localhost] => (item=f740bb5a-060c-4215-afff-11c2cf1e4194) 
ok: [localhost] => (item=aab72afb-067f-4eb2-8539-9b8d12e5be39)
skipping: [localhost] => (item=19e377a9-d5e4-4a16-bb0a-035d94f5f243) 
skipping: [localhost] => (item=5f200a90-0f9a-4644-b9ea-9b740c319c94) 
skipping: [localhost] => (item=bb43ebc7-7868-4d4b-8e69-9ed7d5ea1b83) 
ok: [localhost] => (item=f740bb5a-060c-4215-afff-11c2cf1e4194)
skipping: [localhost] => (item=aab72afb-067f-4eb2-8539-9b8d12e5be39) 
skipping: [localhost] => (item=19e377a9-d5e4-4a16-bb0a-035d94f5f243) 
skipping: [localhost] => (item=5f200a90-0f9a-4644-b9ea-9b740c319c94) 
skipping: [localhost] => (item=bb43ebc7-7868-4d4b-8e69-9ed7d5ea1b83) 

TASK [enothen.image_builder.get_composes : Set fact with compose_request] ********************************
ok: [localhost]

TASK [enothen.image_builder.delete_compose : Debug list of composes to delete] ***************************
ok: [localhost] => (item=f740bb5a-060c-4215-afff-11c2cf1e4194) => {
    "msg": "rhel-10-ib"
}
ok: [localhost] => (item=aab72afb-067f-4eb2-8539-9b8d12e5be39) => {
    "msg": "rhel-9-ib"
}
skipping: [localhost] => (item=19e377a9-d5e4-4a16-bb0a-035d94f5f243) 
skipping: [localhost] => (item=5f200a90-0f9a-4644-b9ea-9b740c319c94) 
skipping: [localhost] => (item=bb43ebc7-7868-4d4b-8e69-9ed7d5ea1b83) 

TASK [enothen.image_builder.delete_compose : Delete compose] *********************************************
ok: [localhost] => (item=f740bb5a-060c-4215-afff-11c2cf1e4194)
ok: [localhost] => (item=aab72afb-067f-4eb2-8539-9b8d12e5be39)
skipping: [localhost] => (item=19e377a9-d5e4-4a16-bb0a-035d94f5f243) 
skipping: [localhost] => (item=5f200a90-0f9a-4644-b9ea-9b740c319c94) 
skipping: [localhost] => (item=bb43ebc7-7868-4d4b-8e69-9ed7d5ea1b83) 

PLAY RECAP ***********************************************************************************************
localhost          : ok=6    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

$ 
```
