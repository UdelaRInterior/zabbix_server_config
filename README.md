Role zabbix_server_config
=========

Ansible role to manage users, groups, templates, actions and mediatypes on Zabbix servers.

Requirements
------------

Ansible >= 2.9

Role Variables
--------------

```yaml
zabbix_server_config_become_on_localhost: false
zabbix_server_config_remove_apache_default_vhost: no

### API Login
zabbix_server_install_zabbix_api_pkg: true
zabbix_server_config_url: "https://{{ invertory_hostname }}"
zabbix_server_config_validate_certs: false

# Zabbix Super Admin default credentials
zabbix_server_config_default_superadmin_user: Admin
zabbix_server_config_default_superadmin_pass: zabbix

# If you wish to disable the default 'Zabbix Super Admin', you must
# give the credentials of another user with super admin privileges
zabbix_server_config_disable_default_superadmin_user: true
zabbix_server_config_custom_superadmin_user: super_user
zabbix_server_config_custom_superadmin_pass: strong-password

## Users groups
zabbix_server_config_custom_users_groups:
  - name: Test group
    state: present
    rights:
      - host_group: Linux servers
        permission: RO      # Choices: denied - RO - RW
    # users:                # Optional
    #     - user1
    #     - user2

## Users
zabbix_server_config_custom_users:
  - name: My User name              # Optional
    surname: My User surname        # Optional
    alias: "{{ zabbix_server_config_custom_superadmin_user }}"
    passwd: "{{ zabbix_server_config_custom_superadmin_pass }}"
    usrgrps:
      - Zabbix administrators
    type: Zabbix super admin        # Optional. Choices: 'Zabbix user' - 'Zabbix admin' - 'Zabbix super admin'
    state: present                  # Optional
    override_passwd: false          # Optional
    lang: en_GB                     # Optional
    theme: default                  # Optional. Choices: 'default' - 'blue-theme' - 'dark-theme'
  - alias: api_user
    passwd: strong-password
    usrgrps:
      - Guests


### Templates
zabbix_server_config_custom_templates_src: zabbix_custom_templates
zabbix_server_config_custom_templates_xml:
  - "{{ zabbix_server_config_custom_templates_src }}/zbx_exports_templates.xml"


### Mediatypes
zabbix_server_config_alertscript_path: /usr/lib/zabbix/alertscripts/
zabbix_server_config_custom_mediatypes:
  - name: Telegram
    type: script
    status: enabled
    script_src: telegram.sh.j2      # Path in local project
    script_name: telegram.sh
    script_params:
      - '{ALERT.SENDTO}'
      - '{ALERT.SUBJECT}'
      - '{ALERT.MESSAGE}'
    state: present

## For included Telegram mediatype example. Vars for telegram.sh.j2 template:
zabbix_server_config_telegram_username: 'telegram-bot'
zabbix_server_config_telegram_password: 'your-super-secret-pass'
zabbix_server_config_telegram_usergroup: 'ReadOnly-telegram'
zabbix_server_config_telegram_bot_token: 'your-token'
##


### Actions
zabbix_server_config_custom_actions:
  - name: Send alerts via Telegram
    event_source: trigger
    state: present
    status: enabled
    esc_period: 60
    default_subject: "{{ zabbix_server_config_actions_default_subject }}"
    default_message: "{{ zabbix_server_config_actions_default_message }}"
    recovery_default_subject: "{{ zabbix_server_config_actions_recovery_default_subject }}"
    recovery_default_message: "{{ zabbix_server_config_actions_recovery_default_message }}"
    conditions:
      - type: trigger_severity
        operator: '>='
        value: Average
    operations:
      - type: send_message
        media_type: Telegram
        send_to_users:
          - telegram-bot
    recovery_operations:
      - type: send_message
        media_type: Telegram
        send_to_users:
          - telegram-bot
```

Dependencies
------------

This role depends of `zabbix-api` Python package, but performs the installation himself.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
  - hosts: servers
    roles: zabbix_server_config
    zabbix_server_config_become_on_localhost: false
    zabbix_server_install_zabbix_api_pkg: true
    zabbix_server_config_url: "https://{{ invertory_hostname }}"
    zabbix_server_config_default_superadmin_user: Admin
    zabbix_server_config_default_superadmin_pass: zabbix
    zabbix_server_config_custom_users:
      - alias: api_user
        passwd: strong-password
        usrgrps:
          - Guests
    zabbix_server_config_custom_templates_xml:
     - "{{ zabbix_server_config_custom_templates_src }}/zbx_exports_templates.xml"
```

License
-------

(c) Universidad de la República (UdelaR), Red de Unidades Informáticas de la UdelaR en el Interior.

Licenced under GPL-v3

Author Information
------------------

[@vamgnu](https://github.com/vamgnu)
[@santiagomr](https://github.com/santiagomr)
[@UdelaRInterior](https://github.com/UdelaRInterior)
https://proyectos.interior.edu.uy/
