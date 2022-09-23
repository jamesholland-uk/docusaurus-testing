---
sidebar_position: 0
---

# Panorama Configuration, Commit and Push

In this guide we will make configuration changes on Panorama within a Device Group. We will then commit the changes to Panorama, then push the changes to the managed devices (firewalls).

This guide assumes a working installation of Ansible with the PAN-OS collection installed, working connectivity to Panorama, and administrative credentials capable of performing configuration, commit and push actions via Panorama.

## Create playbook files and define connectivity to Panorama

Create a new Ansible yaml file named `device-group-changes-commit-and-push.yml`, establish a variable block called `provider` for Panorama, and reference the PAN-OS collection:

```yaml
---
- hosts: '{{ target | default("panorama") }}'
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

## Make the configuration changes

Start the playbook tasks by defining the desired configuration state, some new configuration elements within the Device Group:

```yaml
  tasks:
    - name: Create address object
      panos_address_object:
        provider: "{{ provider }}"
        device_group: "{{ devicegroup }}"
        name: "Object-One"
        value: "1.1.1.1"

    - name: Create service object
      panos_service_object:
        provider: "{{ provider }}"
        device_group: "{{ devicegroup }}"
        name: "tcp-12345"
        destination_port: "12345"

    - name: Add security rule
      panos_security_rule:
        provider: "{{ provider }}"
        device_group: "{{ devicegroup }}"
        rule_name: "Allow SSH on port 12345"
        source_zone: ["internet"]
        source_ip: ["any"]
        destination_zone: ["dmz"]
        destination_ip: ["Object-One"]
        application: ["ssh"]
        service: ["tcp-22"]
        action: "allow"
```

## Commit to Panorama

Continue the tasks with a commit operation on Panorama for the configuration changes just made:

```yaml
    - name: Commit candidate configuration
      panos_commit_panorama:
        provider: "{{ provider }}"
      register: results
    - debug:
        msg: "Commit with job ID: {{ results.jobid }} had output: {{ results.details }}"
```

## Push to Managed Devices

Continue the tasks with a push (and commit) to the managed devices in the Device Group:

```yaml
    - name: Commit and Push template configuration
      panos_commit_push:
        provider: "{{ provider }}"
        style: "device group"
        name: "{{ devicegroup }}"
        include_template: no
      register: results
    - debug:
        msg: "Push with job ID: {{ results.jobid }} finished"
```

## Test importing code

import CodeBlock from '@theme/CodeBlock';
import MyComponentSource from '!!raw-loader!./panorama-config-commit-push.md';

<CodeBlock language="jsx">{MyComponentSource}</CodeBlock>
