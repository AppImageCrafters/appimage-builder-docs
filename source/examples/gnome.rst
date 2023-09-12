

====================================
Gnome application (gnome-calculator)
====================================

**NOTE**: If your app uses svg images you should bundle ``librsvg2-common``

.. code-block:: yaml

    version: 1

    AppDir:
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
          image: appimagecrafters/tests-env:debian-stable
          command: "./AppRun"
          use_host_x: True
        centos:
          image: appimagecrafters/tests-env:centos-7
          command: "./AppRun"
          use_host_x: True
        arch:
          image: appimagecrafters/tests-env:archlinux-latest
          command: "./AppRun"
          use_host_x: True
        fedora:
          image: appimagecrafters/tests-env:fedora-30
          command: "./AppRun"
          use_host_x: True
        ubuntu:
          image: appimagecrafters/tests-env:ubuntu-xenial
          command: "./AppRun"
          use_host_x: True


    AppImage:
      arch: i686