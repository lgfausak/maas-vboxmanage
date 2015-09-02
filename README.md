# maas-vboxmanage - maas management of remote virtualbox

Manage a remote virtualbox with MaaS

## Overview

This is a very simple hack of the MaaS source code with accomplishes a couple of things.

* A new type is added called vbox-manage which presents on the MaaS node's power type.  In addition to the likes of ether_wake, virsh and Intel AMT there is now a vbox-manage type.
* The new type requires a machine name, virtualbox hostname and virtualbox username as parameters.
* When commissioning, the virtualbox hostname+username is combined to do an ssh from the MaaS server to username@hostname (it must be reachable, you will probably need to add maas user credentials (ssh-keygen as a maas user), then ssh to the username@hostname from the MaaS server's maas user.  You will need to update the remote hostname's .ssh/authorized_keys with the new maas user key.
* only power_on is supported (lazy i guess)


## Installation

This is a hack. I changed the python file which describes the input screen for node power type.  This file can be overwritten, but, it might be out of date.  This file (power_schema.py) is from MaaS 1.8 on launchpad.  It would be the safest thing to just edit the file and insert the new array element as I have listed below.  I include the entire file here just as a reminder to me of what worked. So, with my current version of MaaS I do the overwrite method.

Overwrite:
```
cp power_schema.py /usr/lib/python2.7/dist-packages/provisioningserver/power_schema.py
```

This is a better way. You can edit that file and insert this section in the JSON_POWER_TYPE_PARAMETERS array:

Insert:
```
JSON_POWER_TYPE_PARAMETERS = [
    {
        'name': 'vboxmanage',
        'description': 'vbox-manage',
        'fields': [
            make_json_field('vbox_vmname', "Machine Name"),
            make_json_field('vbox_address', "Virtualbox hostname"),
            make_json_field('vbox_user', "Virtualbox username"),
        ],
    },

    ...
```

Also, there is a new file which needs to be copied to the MaaS template directory.  This is a completely new file with handles the power stuff for the new power type:

Template:
```
cp vboxmanage.template /etc/maas/templates/power/.
```

Then, as mentioned before, you need to make sure the maas user can ssh to the vbox_user@vbox_address.  That is an ssh authorized_keys thing, I'll leave that up to the reader.

That is about it.  There are 3 fields.  The first is the Machine Name.  This is the name of the machine you made up when you created the virtual machine in the first place.  That virtual machine should be set to boot from lan.

## Notes
This work is inspired from https://github.com/izoratti/MAAS_VirtualBox. I would have used that, except I needed the ether_wake type to function rather than override it.
