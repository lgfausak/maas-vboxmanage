# maas-vboxmanage - maas management of remote virtualbox

Manage a remote virtualbox with MaaS

## Overview

This is a very simple hack of the MaaS source code with accomplishes a couple of things.

* A new power type is added called vbox-manage which presents on the MaaS node's power type screen.
* The new type requires a machine name, virtualbox hostname and virtualbox username as parameters.
* The new power type has the ability to control power on, others can be added, I just haven't done it.

## Installation

### Install the new power type in the power_schema.py MaaS file
I changed the python file which describes the input screen for node power type.  This file can be overwritten, but, it might be out of date.  This file (power_schema.py) is from MaaS 1.8 on launchpad.  It would be the safest thing to just edit the file and insert the new array element as I have listed below.  I include the entire file here just as a reminder to me of what worked. So, with my current version of MaaS I do the overwrite method.  Again, I recommend you just edit the file and insert the new stanza.

Overwrite:
```
cp power_schema.py /usr/lib/python2.7/dist-packages/provisioningserver/power_schema.py
```

This is a better way. You can edit that file and insert this section in the JSON_POWER_TYPE_PARAMETERS array:

Edit method:
```
find out where the JSON_POWER_TYPE_PARAMETERS is, then add this as the first element to the array.

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

After installing the new power type the application (apache) needs to be restarted.  I simply reboot.  You can verify the new type is there by looking at this screen:

![alt text][powernode]


### Install new powertype template

Also, there is a new file which needs to be copied to the MaaS template directory.  This is a completely new file with handles the power stuff for the new power type:

Template:
```
cp vboxmanage.template /etc/maas/templates/power/.
```

Then, as mentioned before, you need to make sure the maas user can ssh to the vbox_user@vbox_address.  That is an ssh authorized_keys thing, here are some cheat sheet notes how to do that. Your mileage may vary.

* log (su to) in as maas user (might have to chsh from /bin/false)
* ssh-keygen
* cat ~/.ssh/*pub
* cut the output from the cat and copy it. Then ssh to the remote virtualbox machine.
* ssh vbox_user@vbox_address (you will be asked for password)
** cat > ~/.ssh/authorized_keys
** (paste the previously captured key)
** log out of remote machine.
* try the ssh again (you should get in without password)
* you may want to log out of the maas user, then chsh the shell back to /bin/false

That is about it.  There are 3 fields.  The first is the Machine Name.  This is the name of the machine you made up when you created the virtual machine in the first place.  That virtual machine should be set to boot from lan.

[alt text][bootfromlan]


## Notes
This work is inspired from https://github.com/izoratti/MAAS_VirtualBox. I would have used that, except I needed the ether_wake type to function rather than override it.

[powernode]: https://github.com/lgfausak/maas-vboxmanage/raw/master/images/powernode.png "Example Node Power Edit Screen"
[bootfromlan]: https://github.com/lgfausak/maas-vboxmanage/raw/master/images/bootfromlan.png "Example Boot From Lan"
