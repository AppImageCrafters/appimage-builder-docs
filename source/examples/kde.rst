==========================
Qt/Kde application (kcalc)
==========================

.. code-block:: yaml

    version: 1

    AppDir:
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
          image: appimagecrafters/tests-env:debian-stable
          command: "./AppRun"
          use_host_x: True
          env:
            QT_DEBUG_PLUGINS: 1
        centos:
          image: appimagecrafters/tests-env:centos-7
          command: "./AppRun"
          use_host_x: True
          env:
            QT_DEBUG_PLUGINS: 1
        arch:
          image: appimagecrafters/tests-env:archlinux-latest
          command: "./AppRun"
          use_host_x: True
          env:
            QT_DEBUG_PLUGINS: 1
        fedora:
          image: appimagecrafters/tests-env:fedora-30
          command: "./AppRun"
          use_host_x: True
          env:
            QT_DEBUG_PLUGINS: 1
        ubuntu:
          image: appimagecrafters/tests-env:ubuntu-xenial
          command: "./AppRun"
          use_host_x: True


    AppImage:
      update-information: None
      sign-key: None
      arch: x86_64