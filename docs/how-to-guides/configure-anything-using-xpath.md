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

## Obtain the XPath and element required

Use the PAN-OS API browser, found at `https://your-device/api`, or an exported configuration from your device to ascertain the XPath required, and the element required to be inserted at the XPath.

This example creates a new VSYS.

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

## Do.....

```yaml
    - name: Create VSYS
      paloaltonetworks.panos.panos_config_element:
        provider: "{{ provider }}"
        xpath: '/config/devices/entry[@name="localhost.localdomain"]/vsys'
        element: '<entry name="{{ vsys_id_2 }}"><display-name>{{ vsys_name_2 }}</display-name></entry>'
```
