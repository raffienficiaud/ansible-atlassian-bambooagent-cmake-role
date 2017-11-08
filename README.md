Ansible Atlassian Bambooagent CMake role
========================================

Installs a particular version of `cmake` into the remotes and declares it on the bamboo capabilities.

Requirements
------------

Currently working only on Ubuntu, OSX and Windows.

Role Variables
--------------

| variable | default | meaning |
|----------|---------|---------|
|cmake_installation| **required**| the installation definition|
|bambooagent_install_root|**required on linux**| the bamboo agent root folder for local programs installation. |
|bamboo_capabilities|**required**| capabilities. |

### cmake_installation

* Linux: On Linux, a tar file is deflated on the remote. The fields `file`, `subfolder` and `version` should be defined. Example:

  ```yaml
  bamboo_cmake_version:
    major: 3
    minor: 4
    patch: 1

  bamboo_cmake_installation:
    file: 'cmake-{{bamboo_cmake_version.major}}.{{bamboo_cmake_version.minor}}.{{bamboo_cmake_version.patch}}-Linux-x86_64.tar.gz'
    subfolder: cmake-{{bamboo_cmake_version.major}}.{{bamboo_cmake_version.minor}}.{{bamboo_cmake_version.patch}}-Linux-x86_64
    version: "{{bamboo_cmake_version}}"
  ```

* OSX: on OSX, it uses the DMG installer, to which 'version' should be added. Example:
  ```yaml
  bamboo_cmake_version:
    major: 3
    minor: 4
    patch: 1

  bamboo_cmake_installation:
    file: 'cmake-{{bamboo_cmake_version.major}}.{{bamboo_cmake_version.minor}}.{{bamboo_cmake_version.patch}}-Darwin-x86_64.dmg'
    install_cmd: 'rm -rf /Applications/CMake.app && cp -R -f "${mount}/CMake.app" /Applications/'
    remove_interactive: True
    version: "{{ bamboo_cmake_version }}"
  ```

### bambooagent_install_root

The variable is required due to the fact that the `cmake` installation on Linux goes to a folder `{{ bambooagent_install_root }}/usr/local` that is local to the bamboo agent. This folder takes precedence over the system path when the agent starts.

### Capabilities declared by the role

Once run, this role declares the following capabilities onto the Bamboo agent instance:

| capability | value |
|----------|---------|
|system.builder.command.cmake(/ctest/cpack)| location of the commands|
|cmake_version| version of cmake |

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
    - bamboo_cmake_version:
        major: 3
        minor: 7
        patch: 1

    - bamboo_cmake_installation:
      file: "/path/to/cmake/installers/cmake-{{bamboo_cmake_version.major}}.{{bamboo_cmake_version.minor}}.{{bamboo_cmake_version.patch}}-Darwin-x86_64.dmg"
      install_cmd: 'rm -rf /Applications/CMake.app && cp -R -f "${mount}/CMake.app" /Applications/'
      remove_interactive: True
      version: "{{ bamboo_cmake_version }}"

    pre_tasks:
      - name: '[BAMBOO] empty capabilities declaration'
        set_fact:
          bamboo_capabilities: {}

    roles:

      # Installs cmake
      - role: raffienficiaud.atlassian-bambooagent-cmake-role
        cmake_installation: "{{ bamboo_cmake_installation }}"
        bambooagent_install_root: "/home/bambooagent/"
  ```

License
-------

BSD

Author Information
------------------

Any comments on the Ansible, PR or bug reports are welcome from the corresponding Github project.
