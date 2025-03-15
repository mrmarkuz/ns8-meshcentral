# ns8-meshcentral

[MeshCentral](https://meshcentral.com/) is a full computer management web site. \
Documentation: https://ylianst.github.io/MeshCentral/meshcentral/

## Install

Install via Software center:

  - Add a Software repository pointing to `https://repo.mrmarkuz.com/ns8/updates/`, check out the [repo webpage](https://repo.mrmarkuz.com) how to do it.

Instantiate the module on CLI with:

    add-module ghcr.io/nethserver/meshcentral:latest 1

The output of the command will return the instance name.
Output example:

    {"module_id": "meshcentral1", "image_name": "meshcentral", "image_url": "ghcr.io/nethserver/meshcentral:latest"}

## Configure

The FQDN needs to be configured in the Web UI app settings. A valid certificate is recommended.

## First login

At the login page you can create an account. The created user is the admin user. After creating the admin user it's not possible to create another user at the login page.

## Mongo on Proxmox

To make mongodb work, the CPU needs to be set to "host" in the Proxmox VM hardware. I found it in the [Proxmox Forum](https://forum.proxmox.com/threads/enable-avx.129019/)

## Migration from NS7

### Export the data from NS7 to NS8 via scp:

Scp the last database backup from NS7 to NS8:
    
    scp /var/lib/nethserver/meshcentral/backup/backup.archive root@ns8host.domain.tld:

Scp the important folders from NS7 to NS8:
    
    scp -r /opt/meshcentral/meshcentral-{data,files} root@ns8host.domain.tld:

You could also use the autobackupfiles in `/opt/meshcentral/meshcentral-backups/` which contain the directories and the database.

### Import the data on NS8:

Stop Meshcentral:
    
    runagent -m meshcentral1 systemctl --user stop meshcentral
    
Import files:
    
    \cp -vr meshcentral-data/* /home/meshcentral1/.local/share/containers/storage/volumes/meshcentral-data/_data/
    \cp -vr meshcentral-files/* /home/meshcentral1/.local/share/containers/storage/volumes/meshcentral-files/_data/

Import database:
    
    mv backup.archive /home/meshcentral1/.config/state/

Set right owner:
    
    chown meshcentral1: /home/meshcentral1/.config/state/backup.archive
    chown -R meshcentral1: /home/meshcentral1/.local/share/containers/storage/volumes/meshcentral-data/_data/
    chown -R meshcentral1: /home/meshcentral1/.local/share/containers/storage/volumes/meshcentral-files/_data/

Enter meshcentral app user environment:

    runagent -m meshcentral1

Start mongo container named restore_db:
    
    podman run \
      --rm \
      --detach \
      --interactive \
      --network=none \
      --volume mongo-app:/data/db:Z \
      --replace --name=restore_db \
    ${MONGO_IMAGE}

Copy database to mongo container:
    
    podman cp backup.archive restore_db:/

Enter mongo container:
    
    podman exec -ti restore_db bash

Remove current database:

    mongosh --eval 'db.dropDatabase();' meshcentral

Restore the NS7 database:
    
    mongorestore --archive=/backup.archive

Change collation name from meshcentral to mesh:
    
    mongosh --eval 'db.meshcentral.renameCollection("mesh", { dropTarget: true });' meshcentral

Exit mongo container:
    
    exit

Stop mongo container:
    
    podman stop restore_db

Exit app user env:
    
    exit

Remove the old config.json so it will be recreated at next restart
    
    rm /home/meshcentral1/.local/share/containers/storage/volumes/meshcentral-data/_data/config.json

Click Save in the NS8 app settings to rewrite the config. After the service restart you should be able to login with the NS7 user, even 2FA should work.

## Uninstall

To uninstall the instance:

    remove-module --no-preserve meshcentral1

