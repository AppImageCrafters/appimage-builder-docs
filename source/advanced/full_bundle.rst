.. _advanced-full-bundle:

""""""""""""""""""""
Full AppImage bundle
""""""""""""""""""""

A full AppImage bundle is an AppImage that uses absolutely no libraries from the system. This allows to run it in
weird GNU/Linux or even in FreeBSD. A full bundle includes software components that are considered present in every
GNU/Linux distribution such as fonts config, libGL, libEGL and other related pieces of software.

Strong points & use cases
=========================

A full bundle is the best choice when we want to freeze a piece of software in time, as the only missing dependency
will be the Linux Kernel. It maximizes the portability/compatibility of the bundle allowing it to run in non
GNU/Linux system such as FreeBSD.

Draw backs
==========

Adding more binaries to the bundle means that it will be bigger, usually about 30 Mb more. Also there is an issue with
the NVidia drivers, they require the client and the kernel modules to have the same version. So if your software
requires graphic acceleration and your target user may have an NVidia card making a full bundle may not be a good idea.


Instructions
============

To make a full bundle you have to explicitly include those packages that are excluded by default. The recipe below
show how to create an full AppImage bundle for kcalc.


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

          # Full bundle requirements
          - libx11-6
          - libgl1
          - libglapi-mesa
          - libdrm2
          - libegl1
          - libxcb-shape0
          - libxcb1
          - libx11-xcb1
          - fontconfig-config
          - libfontconfig1
          - libfreetype6
          - libglx0
          - libxcb-xfixes0
          - libxcb-render0
          - libxcb-glx0
          - libxcb-shm0
          - libglvnd0
          - libxcb-dri3-0
          - libxcb-dri2-0
          - libxcb-present0

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
      update-information: None
      sign-key: None
      arch: x86_64
