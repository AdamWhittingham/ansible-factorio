# Factorio

[![Install from Ansible Galaxy](https://img.shields.io/badge/role-bplower.factorio-blue.svg)](https://galaxy.ansible.com/bplower/factorio/)

A role for creating Factorio servers
https://galaxy.ansible.com/bplower/factorio/

## Requirements

No requirements

## Role Variables

Variables can be roughly divided into two groups: deployment configurations and
Factorio configurations.

### Deployment Configurations

The deployment configurations are all related to the way in which ansible
installs the factorio server. These should be abstracted enough to allow
multiple factorio servers to be run simultaneously.

```
server_sources: "/opt/games/sources/factorio"
server_version: "0.14.23"
download_url: "https://www.factorio.com/get-download/{{ server_version }}/headless/linux64"
service_name: "factorio-server"
service_user: "factorio"
service_group: "factorio"
service_root: "/home/{{ service_user }}"
service_port: 34197
service_restart_permitted: true
factorio_default_save: "{{ service_root }}/factorio/saves/default-save.zip"
factorio_target_save: "{{ factorio_default_save }}"
```

More detailed information about these variables is as follows:

- Variable: `server_sources`<br>
  Default: `"/opt/games/sources/factorio"`<br>
  Comments: <br>
  Where to cache server binaries downloaded from the download_url

- Variable: `server_version`<br>
  Default: `"0.14.23"`<br>
  Choices:
  - "0.15.10"
  - "0.15.9"
  - "0.15.6"
  - "0.14.23"
  - "0.13.20"
  - "0.12.35"

  Comments:<br>
  This is used by the default `download_url` and the tasks related to
  uncompressing cached server downloads.

- Variable: `download_url`<br>
  Default: `"https://www.factorio.com/get-download/{{ server_version }}/headless/linux64"`<br>
  Comments:<br>
  The URL to download the server binary from. This will only be downloaded if
  the path `"{{ server_sources }}/factorio-{{ server_version }}.tar.gz"` does
  not exist.

- Variable: `service_name`<br>
  Default: `"factorio-server"`<br>
  Comments:<br>
  The name of the service to create. Multiple instances of factorio servers can
  be run on a single host by providing different values for this variable (See
  the examples section of this document).

- Variable: `service_user`<br>
  Default: `"factorio"`<br>
  Comments:<br>
  The user the service should be run as.

- Variable: `service_group`<br>
  Default: `"factorio"`<br>
  Comments:<br>
  The group the service user should be a member of.

- Variable: `service_root`<br>
  Default: `"/home/{{ service_user }}"`<br>
  Comments:<br>
  The directory in which to store the contents of the factorio zip file
  downloaded from the server. This will result in the factorio resources being
  stored at `{{ service_root }}/factorio/`.

- Variab: `service_port`<br>
  Default: `34197`<br>
  Comments:<br>
  The port to host the service on. This default is the factorio default value.

- Variable: `service_restart_permitted`<br>
  Default: `true`<br>
  Comments:<br>
  Setting this to `false` will prevent the service from being restarted if
  changes were applied. This allows settings to be applied in preparation for
  the next service restart without immediately causing service interruption.

- Variable: `factorio_default_save`<br>
  Default: `"{{ service_root }}/factorio/saves/default-save.zip"`<br>
  Comments:<br>
  The default save file used by the server.

- Variable: `factorio_target_save`<br>
  Default: `"{{ factorio_default_save }}"`<br>
  Comments:<br>
  The save file to be run by the server. This distinction is provided to
  facilitate switching between multiple save files.

### Factorio Configurations

Settings for various config files can be set in dictionaries loosely named after
the file. Each dictionary starts with `factorio_` followed by the filename
(excluding the filetype extension) where hyphens ( - ) are replaced by
underscores ( _ ). For example, the `server-settings.json` file is associated
with the dictionary variable `factorio_server_settings`.

The `default/` folder contains serveral files showing example dictionaries
representing the values provided by the Factorio servers various examples JSON
files.

The following is a list of config files that have been implemented:

- Filename: `server-settings.json`<br>
  Variable: `factorio_server_settings`<br>
  Example:
  ```
  factorio_server_settings:
    name: "My Public Server"
    max_players: 10
    game_password: "mypassword"
    visibility:
      public: true
      lan: true
  ```

- Filename: `server-whitelist.json`<br>
  Variable: `factorio_server_whitelist`<br>
  Example:
  ```
  factorio_server_whitelist:
  - Oxyd
  ```

- Filename: `map-settings.json`<br>
  Variable: `factorio_map_settings`<br>
  Example:
  ```
  factorio_map_settings:
    pollution:
      enabled: false
  ```

- Filename: `map-gen-settings.json`<br>
  Variable: `factorio_map_gen_settings`<br>
  Example:
  ```
  factorio_map_gen_settings:
    water: "high"
    autoplace_controles:
      coal:
        size: "very-low"
  ```

## Example Playbooks

An out of the box example might look as follows:

```
---
- name: Create a default factorio server
  hosts: localhost
  roles:
  - role: bplower.factorio
```

An example with a non-default port, and customized name:
```
---
- name: My slightly changed factorio server
  hosts: localhost
  roles:
  - role: bplower.factorio
    service_port: 12345
    factorio_server_settings:
      name: "My factorio server"
```

An example of multiple servers on a single host:
```
---
- name: Factorio farm
  hosts: localhost
  roles:
  - role: bplower.factorio
    service_port: 50001
    service_name: factorio_1
    service_root: /home/{{ service_user }}/{{ service_name }}
  - role: bplower.factorio
    service_port: 50002
    service_name: factorio_2
    service_root: /home/{{ service_user }}/{{ service_name }}
```

## License

GNU GPLv3

# Development

## Testing

Tests are run by applying example playbooks to docker instances. All testing
resources (save for the `test.sh` script in the root) can be found in the `tests`
directory. At this time, there is only one test playbook (`tests/test.yml`).

Before running tests, you will need to initialize your development environment
by running `init.sh`. At this time, all it does is create a vendor folder and
download the supported versions of the factorio server from www.factorio.com.
The url structure has been copied in `ventor/mock.factorio.local/` to use for
testing, that way tests don't hammer the production web server.
