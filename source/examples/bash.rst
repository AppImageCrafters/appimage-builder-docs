========================
Shell application (BASH)
========================

This recipe will generate a aarch64 (arm64) AppImage for bash. It's cross-built from a amd64 system.

.. code-block:: yaml

    version: 1

    AppDir:
      app_info:
        id: org.gnu.bash
        name: bash
        icon: utilities-terminal
        version: 4.4.20
        exec: bin/bash
        exec_args: $@

      apt:
        arch: arm64
        sources:
          - sourceline: 'deb [arch=arm64] http://ports.ubuntu.com/ubuntu-ports bionic main'
            key_url: 'http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3b4fe6acc0b21f32'

        include:
          - bash
          - coreutils
        exclude:
          - libpcre3


    AppImage:
      update-information: None
      sign-key: None
      arch: aarch64