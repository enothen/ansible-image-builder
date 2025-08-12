Role Name
=========
A role to download images after they have been built.

Requirements
------------
See requirements here: https://github.com/enothen/ansible-image-builder/blob/main/README.md#prerequisites

Role Variables
--------------
api_endpoint: The url to interact with Insights image builder.
storage_dir: Where to check if the image exists. Defaults to `.`.
image_type_to_extension: A dictionary that maps image types to file extensions.

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
