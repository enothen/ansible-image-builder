Role Name
=========
A role to monitor an image build until the job is completed.

Requirements
------------
See requirements here: https://github.com/enothen/ansible-image-builder/blob/main/README.md#prerequisites

Role Variables
--------------
api_endpoint: The url to interact with Insights image builder.
compose_check_retries: How many times to check the status of image builds. Defaults to 15 retries per loop.
compose_check_wait_secs: For how long to wait between each image build status check. Defaults to 60 seconds.
compose_check_max_loops: How many loops of `compose_check_retries` to do, while renewing token in between loops (if expired). Defaults to 5 loops.

Dependencies
------------

Example Playbook
----------------
See the example here: https://github.com/enothen/ansible-image-builder/blob/main/example/playbooks/create-and-download.yaml

License
-------
GPL-3.0

Author Information
------------------
Eric Nothen, https://github.com/enothen/ansible-image-builder
