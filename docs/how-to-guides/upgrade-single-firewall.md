---
sidebar_position: 2
---

# Upgrade Single Firewall

In this guide we will upgrad the PAN-OS softare on a single firewall. We will download the software, install the software, reboot the firewall, then check the firewall is ready again after the reboot.

This guide assumes a working installation of Ansible with the PAN-OS collection installed, working connectivity to the firewall, and administrative credentials capable of performing softare upgrade and reboot actions on the firewall.

## Create playbook files and define connectivity to the firewall

Create a new Ansible yaml file named `upgrade-single-firewall.yml`, establish a variable block called `provider` for the firewall, and reference the PAN-OS collection:

```yaml
---
- hosts: '{{ target | default("firewall") }}'
  connection: local

  vars:
    provider:
      ip_address: "{{ ip_address }}"
      username: "{{ username | default(omit) }}"
      password: "{{ password | default(omit) }}"
      api_key: "{{ api_key | default(omit) }}"

  collections:
    - paloaltonetworks.panos
```

## Initiate download and install of software, and a reboot

Start the playbook tasks with the `panos_software` module, which in one task will download the target version, install the target version, and then initiate a reboot:

```yaml
  tasks:
    - name: Install target PAN-OS version
      paloaltonetworks.panos.panos_software:
        provider: '{{ device }}'
        version: '{{ version }}'
        download: true
        install: true
        restart: true
```

## Wait for the reboot to finish

Continue the tasks by initially waiting for 30 seconds to allow PAN-OS to initiated the reboot and therefore move to a "not ready" state:

```yaml

    - name: Pause for restart
      pause:
        seconds: 30
```

We now want to wait until the firewall has rebooted and is ready to pass traffic again, using the `panos_check` module and recording the `result` of the task execution. The `until` keyword tells Ansible to repeat the `panos_check` task until the conditions are met; that the check has not failed and that the resulting message from the task confirms the device is ready. The task is re-executed every 15 seconds up to a maximum of 100 times:

```yaml
    - name: Check if PAN-OS appliance is ready
      paloaltonetworks.panos.panos_check:
        provider: "{{ device }}"
      changed_when: false
      register: result
      until: result is not failed and result.msg == 'Device is ready.'
      retries: 100
      delay: 15
```

## Final message

The final task returns the last value of the result message, which should confirm the device is ready after the upgrade and reboot, but in the case of any issues, will confirm the status of the firewall at the end of the playbook execution:

```yaml
    - name: Display output
      debug:
        msg: "{{ result.msg }}"
```
