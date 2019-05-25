upgrade_ios ansible playbooks
=============================

These playbooks are being developed to upgrade IOS devices.


Notes
-----

* These playbooks make use of SCP to upload images to target devices


Requirements
------------

* The remote device must be running Cisco IOS
* The remote device must support SSH
* The ansible host must have the pip scp package installed
* The `Images/` folder must be populated with the images being used

How to run
----------

You must populate the required variables at the start of the playbook:

```yaml
- name: IOS > Upgrade device
  hosts: switch
  gather_facts: no
  vars:
    new_image: 'c3750-ipservicesk9-mz.122-55.SE12.bin'
    new_image_md5: 'b28cf0ed5cc0d1928ea4f6656e1c8dde'
```

`new_image` is the image name you wish to deploy to a device.  `hosts` is
the inventory group of switches you want to upgrade.

Below will upgrade a switch and reboot:

```bash
ansible-playbook upgrade_device.yml --tags pre_req,check,upload,reboot
```

If you just want to do some checks and pre-stage:
```bash
ansible-playbook upgrade_device.yml --tags pre_req,upload
```


Authors
-------

* Matthew Eason
* John Imison
