# Shift-CTRL Space: Calibre

An [Ansible role](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html) for managing [Calibre content server](https://manual.calibre-ebook.com/generated/en/calibre-server.html)(s) with optional per-library configurations. Notably, this role has been tested with [Raspbian](https://www.raspbian.org/) on [Raspberry Pi](https://www.raspberrypi.org/) hardware. This role's purpose is to make it simple to deploy the Web server that ships with Calibre in a number of different configurations simultaneously.

## Configuring Calibre content servers

A Calibre Library is a folder containing, at a minimum, a `metadata.db` file. This file is created and managed by the Calibre software itself and represents a single collection of content. You can use the Calibre GUI or the [`calibredb`](https://manual.calibre-ebook.com/generated/en/calibredb.html) command-line utility to make modifications to this file, but you should never do so by manually editing yourself.

A Calibre content server is a process invoked by the [`calibre-server`](https://manual.calibre-ebook.com/generated/en/calibre-server.html) command that exposes a given Calibre Library's contents over a Web (HTTP) interface. For each Calibre Library, an associated content server is configured and managed by this role.

You can manage multiple Libraries (and thus server processes) on a single managed host, although in most cases you will only need one Calibre Library (and server) per host. In order to provide different Library configurations to different hosts, you should instead make use of [Ansible inventory group or host variables](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#hosts-and-groups) to override the role default variables on a per-group or per-host basis.

### Calibre Library configuration variables

Of this role's [default variables](defaults/main.yml), which you can override using any of [Ansible's variable precedence rules](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable), one of the most important is the `calibre_libraries` list. It describes the Calibre Libraries you want deployed on a given managed host. Each item in the list is a dictionary with the following keys:

* `name`: Name with which to manage the Calibre server for a given library. This becomes the systemd instance name. This key is required.
* `library_dir`: Path to the Calibre Library from which to serve content. This key is required.
* `restriction`: Name of a [virtual library](https://manual.calibre-ebook.com/virtual_libraries.html) or [saved search](https://manual.calibre-ebook.com/gui.html#saving-searches) in the Calibre Library to restrict served content to. This key is optional.
* `listen_port`: TCP port on which to listen for incoming HTTP requests. This key is optional when only one Library is defined. It is required if two or more Libraries are defined.
* `sync_source`: Directory containing a master copy of the Calibre Library content to use when synchronizing data from the Ansible controller to the managed host. This should be an `rsync(1)` style `SRC` specifier. (I.e., trailing slashes mean "contents of" rather than the directory itself.) This key is optional but required if you plan to use any of the Library management playbooks (such as [`synchronize.yml`](synchronize.yml)).

It may help to see a few examples:

1. Single Library running on default Calibre content server port (`8080`):
    ```yml
    calibre_libraries:
      - name: main
        library_dir: "{{ calibre_server_home_dir }}/Library"
    ```
1. Two content servers serving content from the same Library. The `main` process is bound to the default HTTP port (`80`) and restricts viewing to the "`Main Library`" virtual library, while another process serves an restricted view of library on the standard HTTP alternative port.
    ```yml
    calibre_libraries:
      - name: main
        library_dir: "{{ calibre_server_home_dir }}/Library"
        restriction: "Main Library"
        listen_port: 80
      - name: unrestricted
        library_dir: "{{ calibre_server_home_dir }}/Library"
    ```

## Additional role variables

In addition to configuring Calibre Libraries and their respective `calibre-server` processes, you can configure various aspects of the Calibre suite and this role's behavior. To do so, set any of the following role variables:

* `calibre_server_username`: The system user name under which the Calibre server will run. Defaults to `calibre`.
* `calibre_server_home_dir`: The home directory of the Calibre server's user account. Defaults to `/srv/{{ calibre_server_username }}`.
* `calibre_server_user_groups`: List of additional user groups to add the Calibre server user account to. Defaults to an empty list.
