Ansible Atlassian Bambooagent CMake role
========================================

Installs a specific version of `cmake` into the remotes and registers the `cmake/cpack/ctest` as system builders for the Bamboo agent's capabilities.
Also registers `cmake` on the `PATH` on Windows and registers the cmake version and paths in a fact `bamboo_capabilities`.

Requirements
------------

Currently working only on Ubuntu, OSX and Windows. Requires the official installation packages on disk. On Linux, this should be the tar file provided by
the official CMake download page. On Windows and OSX, those are respectively the `.exe` and `.dmg` packaged installers.

Role Variables
--------------

| variable | default | meaning |
|----------|---------|---------|
|`cmake_installation`| **required** | a dictionary describing the installation, see below.|
|`linux_install_prefix`| `''` | Installation prefix for Linux (defaults to empty string). |
|`bamboo_capabilities`| (empty dict) | fact dictionary holding the agent's capabilities. The dictionary will contain additional keys after the run.|

### cmake_installation

* On Linux: the cmake tar file is deflated on the remote under the `{{ linux_install_prefix }}/usr/local/cmake-version-specific`. The fields `file`, `subfolder` and `version` of `cmake_installation` should be defined.
Example:

  ```yaml
  local_cmake_version:
    major: 3
    minor: 4
    patch: 1

  local_cmake_installation:
    file: 'cmake-{{ local_cmake_version.major }}.{{ local_cmake_version.minor }}.{{ local_cmake_version.patch }}-Linux-x86_64.tar.gz'
    subfolder: cmake-{{ local_cmake_version.major }}.{{ local_cmake_version.minor }}.{{ local_cmake_version.patch }}-Linux-x86_64
    version: "{{ local_cmake_version }}"
  ```

* On OSX: the role uses the DMG installer, to which 'version' should be added. Example:

  ```yaml
  bamboo_cmake_version:
    major: 3
    minor: 4
    patch: 1

  local_cmake_installation:
    file: 'cmake-{{ local_cmake_version.major }}.{{ local_cmake_version.minor }}.{{ local_cmake_version.patch }}-Darwin-x86_64.dmg'
    install_cmd: 'rm -rf /Applications/CMake.app && cp -R -f "${mount}/CMake.app" /Applications/'
    remove_interactive: True
    version: "{{ local_cmake_version }}"
  ```

### linux_install_prefix

The variable is used to install `cmake` as a relocatable package under this particular prefix. It can go for instance
`{{ bambooagent_install_root }}/usr/local` that is local to the build agent, and which isolates the binaries being in use from the rest of the
operating system. This folder should takes precedence over the system path when the agent starts.

### Capabilities declared by the role

Once run, this role declares the following capabilities onto the Bamboo agent instance:

| capability | value |
|----------|---------|
|`system.builder.command.cmake(/ctest/cpack)`| location of the commands|
|`cmake_version`| version of cmake |

Dependencies
------------

This role relies on the DMG installation role `ansible-atlassian-bambooagent-install-dmg-role`.

Example Playbook
----------------

Here is an example on how to use the role for installing on OSX agents.

  ```yaml
  - hosts: osx-agents
    vars:

    # declares the cmake version and installer
    - play_cmake_version:
        major: 3
        minor: 7
        patch: 1

    - play_cmake_installation:
        file: "/path/to/cmake/installers/cmake-{{ play_cmake_version.major }}.{{ play_cmake_version.minor }}.{{ play_cmake_version.patch }}-Darwin-x86_64.dmg"
        install_cmd: 'rm -rf /Applications/CMake.app && cp -R -f "${mount}/CMake.app" /Applications/'
        remove_interactive: True
        version: "{{ bamboo_cmake_version }}"

    pre_tasks:
      - name: '[BAMBOO] empty capabilities declaration'
        set_fact:
          bamboo_capabilities: {}

    roles:

      # Installs cmake
      - role: atlassian-bambooagent-cmake-role
        vars:
          cmake_installation: "{{ play_cmake_installation }}"
          linux_install_prefix: "/home/bambooagent/"
  ```

License
-------

BSD

Author Information
------------------

Any comments on the Ansible, PR or bug reports are welcome from the corresponding Github project.
