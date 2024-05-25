# ns8-meshcentral



## Install

Install via Software center:

  - Add a Software repository pointing to `https://repo.mrmarkuz.com/ns8/updates/`, check out the [repo webpage](https://repo.mrmarkuz.com) how to do it.
  - Install Guacamole via Software Center

Instantiate the module on CLI with:

    add-module ghcr.io/nethserver/meshcentral:latest 1

The output of the command will return the instance name.
Output example:

    {"module_id": "meshcentral1", "image_name": "meshcentral", "image_url": "ghcr.io/nethserver/meshcentral:latest"}

## Configure

The FQDN needs to be configured in the Web UI app settings. A valid certificate is recommended.

## First login

At the login page you can create an account. The created user is the admin user. After creating the admin user it's not possible to create another user at the login page.

## Uninstall

To uninstall the instance:

    remove-module --no-preserve meshcentral1

