# Users and Roles
* [Installation and Dependencies](#installation-and-dependencies)
* [Tags](#tags)
* [Examples](#examples)

This roles manages OS users and groups.





## Installation and Dependencies

This role has no dependencies.

To install run `ansible-galaxy install DelphicOkami.users_and_groups` or add
this to your `roles.yml`

```YAML
- name: DelphicOkami.users_and_groups
  version: v1.0
```

and run `ansible-galaxy install -p ./roles -r roles.yml`




## Tags

This role uses two tags:
* `configuration` - Ensures users are created, and fully managed
* `install` - Ensures that all required directories are created




## Examples

Simple example for creating two users and two groups.

```YAML
- name: Configure User Access
  hosts: sandbox

  roles:
    - name: DelphicOkami.users_and_groups
      users_and_groups:
        groups:
          lorem: # This is the group name
            system: yes
          - name: ipsum
        users:
          lorem_ipsum:
            name: lorem.ipsum # This will override the key as the username if it is set
            groups:
              - ipsum
              - lorem
            authorized_keys_file: ./lorem.ipsum.pub
            dolor_sit:
              groups:
                - ipsum
                - lorem
              authorized_keys:
                - ssh-rsa APUBLICSSHKEY
          - name: dolor.ament
            groups:
              - ipsum
```

Creating a jailed SFTP user (cf [here](https://wiki.archlinux.org/index.php/SFTP_chroot) for a step-by-step guide):

```YAML
- name: Configure User Access
  hosts: sandbox

  roles:
    - name: DelphicOkami.users_and_groups
      users_and_groups:
        authorized_keys_dir: /etc/ssh/authorized_keys
        groups:
          sftp_only:
            - name: sftp_only
        users:
          sftp:
            group: sftp_only
            home: /mnt/sftp_vol
```

In most cases you would keep the list of users in external vars file or
group|host vars file.

```YAML
- name: Configure User Access
  hosts: sandbox

  vars_files:
    - "vars/sandbox/users.yml"

  roles:
    - name: DelphicOkami.users_and_groups
      users_and_groups:
        groups: "{{ base_image.os_groups }}"
        users: "{{ base_image.admins }}"

    - name: DelphicOkami.users_and_groups
      users_and_groups:
        users: "{{ developers }}"
```

Add selected group to sudoers

```YAML
- name: Configure User Access
  hosts: sandbox

  vars_files:
    - "vars/sandbox/users.yml"

  roles:
    - name: DelphicOkami.users_and_groups
      users_and_groups:
        groups: "{{ base_image.os_groups }}"
        users: "{{ base_image.admins }}"

    - name: DelphicOkami.users_and_groups
      users_and_groups:
        users: "{{ developers }}"

    - name: DelphicOkami.users_and_groups
      users_and_groups:
        sudoers:
          wheel:
            user: "%wheel"
            runas: "ALL=(ALL)"
            commands: "NOPASSWD: ALL"
```

Use whitelist groups option to allow users contextually.

Var file with users:

```YAML
---

# vars/users.yml

users_and_groups:
  groups:
    admins:
    developer_group_alpha:
    developer_group_beta:
  users:
    admin_user:
      name: admin.user
      group: admins
    alpha_user:
      name: alpha.user
      group: alpha_develops
    beta_user:
      name: beta.user
      group: developer_group_beta
```

In a base image:

```YAML
---

# playbooks/base_image.yml

- name: Base Image
  hosts: "{{ hosts }}"

  vars_files:
    - vars/users.yml

  roles:
    - role: DelphicOkami.users_and_groups
      users_and_groups:
        whitelist_groups:
          - admins

    - role: base_image
```

In a service role:

```YAML
---

# playbooks/alpha_service.yml

- name: Alpha Service
  hosts: "{{ hosts }}"

  vars_files:
    - vars/users.yml

  roles:
    - role: DelphicOkami.users_and_groups
      users_and_groups:
        whitelist_groups:
          - admins
          - developer_group_alpha

    - role: alpha_service
```
