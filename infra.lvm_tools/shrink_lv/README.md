# ansible-bash-shrink-lv
Ansible role for shrinking a logical volume.
The role contains the shell scripts to shrink the logical volume,
as well as the script wrapping it to run as part of the pre-mount step during the boot process.
The devices to shrink and their expected sizes should be passed via the `shrink_lv_devices` variable.
For example
```
vars:
  shrink_lv_devices:
  - device: /dev/mapper/test
    size: 4G
```

## Example of a playbook to run the role
The following yaml is an example of a playbook that runs the role against a group of hosts named `rhel` to extend their boot partition by 1G.
The boot partition is automatically retrieved by the role by identifying the existing mounted partition to `/boot` and passing the information to the script using the `kernel_opts`.

```yaml
- name: Shrink Logical Volumes playbook
  hosts: rhel
  debugger: on_failed
  vars:
    shrink_lv_devices:
      - device: /dev/mapper/test
        size: 4G
  pre_tasks:
    - name: Gather facts
      ansible.builtin.setup:
        gather_subset:
          - kernel
  roles:
    - shrink_lv

```

With an inventory file consisting of group [rhel]:
```ini
[rhel]
localhost

[rhel:vars]
ansible_user=root
ansible_port=2022
```

# Validate execution
The script will add an entry to the kernel messages (`/dev/kmsg` or `/var/log/messages`) with success or failure.
In case of failure, it may also include an error message retrieved from the execution of the script.

A successful execution will look similar to this:
```bash
[root@localhost ~]# cat /var/log/messages |grep Resizing -A 2 -B 2
Oct 16 17:55:00 localhost /dev/mapper/rhel-root: 29715/2686976 files (0.2% non-contiguous), 534773/10743808 blocks
Oct 16 17:55:00 localhost dracut-pre-mount: resize2fs 1.42.9 (28-Dec-2013)
Oct 16 17:55:00 localhost journal: Resizing the filesystem on /dev/mapper/rhel-root to 9699328 (4k) blocks.#012The filesystem on /dev/mapper/rhel-root is now 9699328 blocks long.
Oct 16 17:55:00 localhost journal:  Size of logical volume rhel/root changed from 40.98 GiB (10492 extents) to 37.00 GiB (9472 extents).
Oct 16 17:55:00 localhost journal:  Logical volume rhel/root successfully resized.
```
