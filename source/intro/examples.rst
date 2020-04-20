.. _intro-examples:

""""""""""""""""""""""""""""""""
appimage-builder recipe examples
""""""""""""""""""""""""""""""""

Here you will find a collection of recipes for building AppImage from applications built using different technologies.
With each recipe are included a set of advices to ease their replication.

You can also find more examples in this address: https://github.com/AppImageCrafters/appimage-builder/tree/master/examples

========================
Shell application (BASH)
========================

This recipe will generate a aarch64 (arm64) AppImage for bash. It's cross-built from a amd64 system.

.. code-block:: yaml

    version: 1

    AppDir:
      path: ./AppDir

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

====================================
Gnome application (gnome-calculator)
====================================

**NOTE**: If your app uses svg images you should bundle ``librsvg2-common``

.. code-block:: yaml

    version: 1

    AppDir:
      path: ./AppDir

      app_info:
        id: org.gnome.Calculator
        name: gnome-calculator
        icon: gnome-calculator
        version: 3.28.0
        exec: usr/bin/gnome-calculator

      apt:
        arch: i386
        sources:
          - sourceline: 'deb [arch=i386] http://mx.archive.ubuntu.com/ubuntu/ bionic main restricted universe multiverse'
            key_url: 'http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3b4fe6acc0b21f32'

        include:
          - gnome-calculator
          - librsvg2-common
        exclude:
          - adwaita-icon-theme
          - humanity-icon-theme

      files:
        exclude:
          - usr/lib/x86_64-linux-gnu/gconv
          - usr/share/man
          - usr/share/doc/*/README.*
          - usr/share/doc/*/changelog.*
          - usr/share/doc/*/NEWS.*
          - usr/share/doc/*/TODO.*

      test:
        debian:
          image: appimage-builder/test-env:debian-stable
          command: "./AppRun"
          use_host_x: True
        centos:
          image: appimage-builder/test-env:centos-7
          command: "./AppRun"
          use_host_x: True
        arch:
          image: appimage-builder/test-env:archlinux-latest
          command: "./AppRun"
          use_host_x: True
        fedora:
          image: appimage-builder/test-env:fedora-30
          command: "./AppRun"
          use_host_x: True
        ubuntu:
          image: appimage-builder/test-env:ubuntu-xenial
          command: "./AppRun"
          use_host_x: True


    AppImage:
      arch: i686

==========================
Qt/Kde application (kcalc)
==========================

.. code-block:: yaml

    version: 1

    AppDir:
      path: ./AppDir

      app_info:
        id: org.kde.kcalc
        name: kcalc
        icon: accessories-calculator
        version: 17.12.3
        exec: usr/bin/kcalc

      apt:
        arch: amd64
        sources:
          - sourceline: 'deb [arch=amd64] http://archive.ubuntu.com/ubuntu/ bionic main restricted universe multiverse'
            key_url: 'http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3b4fe6acc0b21f32'
          - sourceline: 'deb [arch=amd64] http://archive.ubuntu.com/ubuntu/ bionic-updates main restricted universe multiverse'
          - sourceline: 'deb [arch=amd64] http://archive.ubuntu.com/ubuntu/ bionic-backports main restricted universe multiverse'

        include:
          - kcalc
          - libpulse0
        exclude:
          - core-packages
          - graphics-stack-packages
          - xclient-packages

          - phonon4qt5
          - libkf5service-bin
          - perl
          - perl-base
          - libpam-runtime

      files:
        exclude:
          - usr/lib/x86_64-linux-gnu/gconv
          - usr/share/man
          - usr/share/doc/*/README.*
          - usr/share/doc/*/changelog.*
          - usr/share/doc/*/NEWS.*
          - usr/share/doc/*/TODO.*
      runtime:
        env:
          APPDIR_LIBRARY_PATH: $APPDIR/lib/x86_64-linux-gnu:$APPDIR/usr/lib/x86_64-linux-gnu:$APPDIR/usr/lib/x86_64-linux-gnu/pulseaudio

      test:
        debian:
          image: appimage-builder/test-env:debian-stable
          command: "./AppRun"
          use_host_x: True
          env:
            - QT_DEBUG_PLUGINS=1
        centos:
          image: appimage-builder/test-env:centos-7
          command: "./AppRun"
          use_host_x: True
          env:
            - QT_DEBUG_PLUGINS=1
        arch:
          image: appimage-builder/test-env:archlinux-latest
          command: "./AppRun"
          use_host_x: True
          env:
            - QT_DEBUG_PLUGINS=1
        fedora:
          image: appimage-builder/test-env:fedora-30
          command: "./AppRun"
          use_host_x: True
          env:
            - QT_DEBUG_PLUGINS=1
        ubuntu:
          image: appimage-builder/test-env:ubuntu-xenial
          command: "./AppRun"
          use_host_x: True


    AppImage:
      update-information: None
      sign-key: None
      arch: x86_64

============================
Multimedia application (VLC)
============================


.. code-block:: yaml

    version: 1

    script:
      - rm -r ./AppDir || true

    AppDir:
      path: ./AppDir

      app_info:
        id: vlc
        name: VLC media player
        icon: vlc
        version: 3.0.8-0-gf350b6b5a7
        exec: usr/bin/vlc

      apt:
        arch: amd64
        sources:
          - sourceline: 'deb [arch=amd64] http://archive.ubuntu.com/ubuntu/ bionic main restricted universe multiverse'
            key_url: 'http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3b4fe6acc0b21f32'
          - sourceline: 'deb [arch=amd64] http://archive.ubuntu.com/ubuntu/ bionic-updates main restricted universe multiverse'

        include:
          - vlc

      test:
        debian:
          image: appimage-builder/test-env:debian-stable
          command: "./AppRun"
          use_host_x: True
        centos:
          image: appimage-builder/test-env:centos-7
          command: "./AppRun"
          use_host_x: True
        arch:
          image: appimage-builder/test-env:archlinux-latest
          command: "./AppRun"
          use_host_x: True
        fedora:
          image: appimage-builder/test-env:fedora-30
          command: "./AppRun"
          use_host_x: True
        ubuntu:
          image: appimage-builder/test-env:ubuntu-xenial
          command: "./AppRun"
          use_host_x: True

    AppImage:
      arch: x86_64
