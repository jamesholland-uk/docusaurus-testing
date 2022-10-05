---
sidebar_position: 3
---

# Configure Anything Using XPath

In this guide we will make configuration changes to a firewall using a generic module capable of configuring anything in PAN-OS using the XPath. This is especially useful for configuring features for which there is no predefined module in the PAN-OS Ansible Collection.

This guide assumes a working installation of Ansible with the PAN-OS collection installed, working connectivity to the firewall, and administrative credentials capable of performing configuration actions on the firewall.

## Create playbook files and define connectivity to the firewall

Create a new Ansible yaml file named `configure-with-xpath.yml`, establish a variable block called `provider` for the firewall, and reference the PAN-OS collection:

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

## Identify the XPath and element required

Use the PAN-OS API browser found at `https://your-device/api`, or an exported XML configuration from your device, to ascertain:

- the XPath required
- the element required to be inserted at the XPath

This XML configuration file, shortened for brevity, highlights the XPath and element required to define a VSYS:

```xml
<config version="10.1.0" urldb="paloaltonetworks">
  <mgt-config>
    ...
  </mgt-config>
  <shared>
    ...
  </shared>
  <devices>
    <entry name="localhost.localdomain">
      <network>
        ...
      </network>
      <deviceconfig>
        ...
      </deviceconfig>
      <vsys>
        <entry name="vsys1">
          <display-name>First-VSYS</display-name>
          ...
        </entry>
      </vsys>
```

By following the (indented) elements in the XML configuration, we can see that the XPath required for a VSYS is:

```
/config/devices/entry[@name="localhost.localdomain"]/vsys
```

Then beneath the `vsys` element is the VSYS `entry` itself. When creating a VSYS, we only need to define the VSYS ID number, but we also include a display name for human readability within the configuration later on:

```xml
<entry name="vsys1">
  <display-name>First-VSYS</display-name>
</entry>
```

## Define the VSYS creation task

With the XPath and element, we can now define a task that will create a new VSYS, passing in the XPath and element to the task:

```yaml
  tasks:
    - name: Create VSYS
      paloaltonetworks.panos.panos_config_element:
        provider: "{{ provider }}"
        xpath: '/config/devices/entry[@name="localhost.localdomain"]/vsys'
        element: '<entry name="vsys2"><display-name>Second-VSYS</display-name></entry>'
```

## Final playbook

Putting all the sections together, the playbook in entirety looks like this:

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

  tasks:
    - name: Create a VSYS
      paloaltonetworks.panos.panos_config_element:
        provider: "{{ provider }}"
        xpath: '/config/devices/entry[@name="localhost.localdomain"]/vsys'
        element: '<entry name="vsys2"><display-name>Second-VSYS</display-name></entry>'
```
