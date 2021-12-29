# Alerts

Alert is the most effective tool for infrastructure managers to evaluate the information they have and identify the problem before the outage occurs. For this reason, it is vital that the data we collect can be processed and converted into alerts.

The following items are important in order to be able to generate correct alerts and properly manage reproducing alerts.

### Tagging

Although the basic tool of monitoring tools was to generate alarms and visualize data in the first period of their emergence, today this purpose has become more comprehensive and has acquired a new purpose; automation.

The information required for the correct functioning of automation processes is usually carried between monitoring systems and automation systems through these tags.

Every device in our Infrastructure needs the following three tags by default.

* __environment__
* __application-type__
* __teams__

Example scenario; You want to clean the disk of the servers in the development environment. Make sure you have ansible installed on the proxy server.

1. Create your Ansible playbook.

```yaml
- name: Create Inventory
  hosts: localhost
  connection: local
  tasks:
    - name: Test SSH Connectivity
      local_action: shell timeout 5 ssh -o ConnectTimeout=2 -o PasswordAuthentication=no -o StrictHostKeyChecking=no {{ ansible_user }}@{{ target_host }} "echo success"
      register: ssh_enabled
      ignore_errors: yes

    - name: End Play If Type Agent and Cannot Connect with SSH
      meta: end_play

    - name: Create in-memory Ansible inventory
      add_host:
        name: "{{ target_host }}"
        groups: target

- name: Disk Alert Handler
  hosts: target
  become: "{{ need_root | default('no') }}"
  tasks:
    - name: Find files
      find:
        paths: "{{ path }}"
        recurse: yes
        age: "{{ older_then }}"
        size: "{{ greater_then }}"
        use_regex: yes
        patterns: "{{ patterns.split(',') }}"
      register: target_files

    - name: Take an action
      command: |
      	rm -rf {{ item.path }}
      with_items: "{{ target_files.files }}"
```

2. Move file to proxy server and authorize for zabbix user.

```bash
# Copy file to remote server
scp disk-cleaner.yaml <user>@<proxy-server>:<path-to-zabbix-users-home>

# Change owner to zabbix
chown zabbix. disk-cleaner.yaml
```

3. Now, create the script required for this job on the system. You can see how to create it [here](../components/scripts). The script should be as follows.

```bash
ansible-playbook <path-to-zabbix-users-home>/disk-cleaner.yaml --extra-vars "target_host={HOST.IP} path={EVENT.TAGS.path} older_then={EVENT.TAGS.older-then} greater_then={EVENT.TAGS.older-then} patterns='*.log,*.out'"
```

4. Finally, add the alert trigger. In this way, you will both receive an alarm and run automation. See [Trigger Actions](trigger-actions)
