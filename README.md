Im0.cisco_upgrader
==================

This role assists with deploying new images onto a range of Cisco network devices.


Requirements
------------

* Ansible 2.7+
* Ansible Network Engine
* Suitable Cisco router or switch to upgrade
* Suitable ios/catos images for your device
* Python pip `scp` module installed


Role Variables
--------------

* `Image`: The image filename.
* `ImagePath`: The path to image filename.  Default: `./`.
* `State`: Either `stage` the image for future manual reboot (and boot settings), or, `upgrade` will stage and reboot the device.
* `UpgradeType`: Upgrade types are either `ios-bin-upgrade`, `cat-software-install` or `auto`.
* `BackupImage`: True of False, backup the existing image from the switch.  `ios-bin-upgrade` only.  Default: `False`.


Important Notes
---------------

*Warning*: This role makes changes on your networking device, and, reloads the device.  *It will cause an outage*.  
Ensure you have thoroughly test against a lab device before running against production devices.  Use at own risk.

*Timeouts*: You will probably find it necessary to adjust some ansible configuration variables, specifically:

```
timeout = 60

[persistent_connection]
command_timeout = 1800
persistent_connect_timeout = 1800

```

Otherwise, various tasks such as the md5 checksum calculation may timeout with an error like this:

```
The full traceback is:
WARNING: The below traceback may *not* be related to the actual failure.
  File "/tmp/ansible_ios_command_payload_VB1GqY/ansible_ios_command_payload.zip/ansible/module_utils/network/ios/ios.py", line 145, in run_commands
    return connection.run_commands(commands=commands, check_rc=check_rc)
  File "/tmp/ansible_ios_command_payload_VB1GqY/ansible_ios_command_payload.zip/ansible/module_utils/connection.py", line 182, in __rpc__
    raise ConnectionError(to_text(msg, errors='surrogate_then_replace'), code=code)

fatal: [device]: FAILED! => {
    "changed": false,
    "invocation": {
        "module_args": {
            "auth_pass": null,
            "authorize": null,
            "commands": [
                "verify /md5 flash:c2960-lanbasek9-mz.150-2.SE11.bin"
            ],
            ... omitted ...
        }
    },
    "msg": "command timeout triggered, timeout value is 10 secs.\nSee the timeout setting options in the Network Debug and Troubleshooting Guide."

```


*scp*: The image is currently transfered using `scp`.  `scp` is enabled on the device
if it is not already enabled.  Future updates may enable `tftp` as a transfer
method.


*Stack switches*: Upgrading of stack switches has not specifically been tested, or,
provisioned for.

*MD5 Checksum*: The integrity of the provided binary image is assumed.  It is your
responsibility to ensure the image you're using is not compromised.

At time of execution a local MD5 checksum is captured, and, then a post transfer md5
checksum is compared to ensure data transfer was successful.


Example Playbook
----------------

Stage an image for manual reboot:

```
    - hosts: switches
      gather_facts: no
      roles:
         - { role: Im0.cisco_upgrader, 
               Image: 'c2960-lanbasek9-mz.122-55.SE12.bin',
               State: stage,
               ImagePath: '/tmp',
               UpgradeType: 'ios-bin-upgrade',
               BackupImage: True
            }
```

Upgrade device and reload:

```
- name: Upgrade device
  include_role:
    name: Im0.cisco_upgrader
  vars:
    Image: 'c2960-lanbasek9-mz.122-55.SE12.bin'
    State: 'upgrade'
    ImagePath: '/tmp'
    UpgradeType: 'ios-bin-upgrade'

```


UpgradeType details
-------------------

There are multiple methods for upgrading Cisco networking devices, depending, on make and model.

Currently supported upgrade styles:
* `ios-bin-upgrade`, follows upgrade instructions based from Cisco's website: https://www.cisco.com/c/en/us/support/docs/routers/3800-series-integrated-services-routers/49044-sw-upgrade-proc-ram.html / https://www.cisco.com/c/en/us/support/docs/switches/catalyst-2950-series-switches/41542-191.html
* `cat-software-install`, follows upgrade instructions based from Cisco's website:  https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst3850/software/release/16-3/release_notes/ol-16-3-3850.html#pgfId-1110759


Currently unsupported:
* `nxos-upgrade`
* `ios-xr-upgrade`

*Special mention*:
* `auto`, attempts to select the appropriate upgrade method based on tested/known models. Currently: C881, C2960, C3750



Limitations
-----------

Consider the following limitations as a todo list.  Current limitations include:

* Doesn't support filesystem paths other than `flash:`.
* Only supports `ios` and `catos` upgrades.
* Only supports `scp` copying method.
* Auto detect upgrade method currently not implemented.
* Needs better control over wait times for the device to come back online.  Introduce a new parameter to control this.


License
-------

GPLv3


Author Information
------------------

* John Imison
* Matthew Eason
