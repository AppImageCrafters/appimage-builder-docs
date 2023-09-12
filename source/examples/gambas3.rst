===================
Gambas3 Application
===================

In this tutorial you will learn how to pack a Gambas3 application using the AppImage format. For the purpose we will
use an Ubuntu 18.04 system and the appimage-builder tool.

The tutorial includes several troubleshooting steps that are required when packaging interpreted languages and may be
a bit complicated for novice users. If want to get your application packaged quickly please go to the "Resume"
section, download the recipe template, replace the binary, update the information and repack.


Gambas_ is a free development environment and a full powerful development platform based on a Basic interpreter with
object extensions, as easy as Visual Basic™.

.. _Gambas: http://gambas.sourceforge.net/en/main.html

Requirements
============

- Ubuntu 18.04 system
- appimage-builder (see :ref:`intro-install`)
- git
- gambas3 (see `Gambas PPA Install instructions`_)

.. _Gambas PPA Install instructions: https://launchpad.net/~gambas-team/+archive/ubuntu/gambas3


Getting the source code
=======================

For this tutorial we will be using a simple "Hello World" application without any special modifications. Its source
code can be found at Github_.

.. _Github: https://github.com/AppImageCrafters/appimage-demo-gambas3


.. code-block:: shell

        git clone https://github.com/AppImageCrafters/appimage-demo-gambas3.git

Building
========

To build the gambas3 application we will use the commands gbc3 and gba3 as follows.

.. code-block:: shell

    cd appimage-demo-gambas3/
    gbc3 project/
    gba3 project/ -o appimage-demo-gambas3.gambas


Prepare the AppDir
==================

Once we have compiled the application we will proceed to prepare our AppDir. Which means coping the application binary
and the icon inside the AppDir as follows:

.. code-block:: shell

    mkdir -p AppDir/usr/bin
    cp appimage-demo-gambas3.gambas AppDir/usr/bin/
    mkdir -p AppDir/usr/share/icons/hicolor/32x32/apps/
    cp project/mapview.png AppDir/usr/share/icons/hicolor/32x32/apps/


Generating the recipe
=====================

Once we have the binary and the icon in place we proceed to call appimage-builder generate and answer the prompts as
follows, notice that the file extension is not being included.

.. code-block:: shell

    appimage-builder --generate

    Basic Information :
    ? ID [Eg: com.example.app] : org.appimagecrafters.gambas3-demo
    ? Application Name : Gambas3 Demo
    ? Icon : mapview
    ? Version : latest
    ? Executable path relative to AppDir [usr/bin/app] : usr/bin/appimage-demo-gambas3.gambas
    ? Arguments [Default: $@] : $@
    ? Update Information [Default: guess] : guess
    ? Architecture :  amd64

This command gathers the required information to generate a recipe, then it runs the application to find the runtime
dependencies. Once your application is started must close it in order to proceed with the recipe generation. In the
end a file name `AppImageBuilder.yml` will be created in the current working directory that should look like this:


.. code-block:: yaml

    # appimage-builder recipe see https://appimage-builder.readthedocs.io for details
    version: 1
    AppDir:
      app_info:
        id: org.appimagecrafters.gambas3-demo
        name: Gambas3 Demo
        icon: mapview
        version: latest
        exec: usr/bin/appimage-demo-gambas3.gambas
        exec_args: $@
      runtime:
        env:
          APPDIR_LIBRARY_PATH: $APPDIR/usr/lib/x86_64-linux-gnu:$APPDIR/usr/lib/x86_64-linux-gnu/gconv:$APPDIR/usr/lib/x86_64-linux-gnu/gtk-2.0/2.10.0/engines:$APPDIR/lib/x86_64-linux-gnu:$APPDIR/usr/lib/x86_64-linux-gnu/qt4/plugins/accessible:$APPDIR/usr/lib/gambas3:$APPDIR/usr/lib/x86_64-linux-gnu/qt4/plugins/accessiblebridge:$APPDIR/usr/lib/x86_64-linux-gnu/gdk-pixbuf-2.0/2.10.0/loaders
      apt:
        arch: amd64
        allow_unauthenticated: true
        sources:
        - sourceline: deb http://archive.ubuntu.com/ubuntu/ bionic main restricted universe multiverse
        - sourceline: deb http://archive.ubuntu.com/ubuntu/ bionic-updates main restricted universe multiverse
        - sourceline: deb http://security.ubuntu.com/ubuntu bionic-security main restricted universe multiverse
        - sourceline: deb http://archive.neon.kde.org/user bionic main
        - sourceline: deb http://ppa.launchpad.net/gambas-team/gambas3/ubuntu bionic main
        include:
        - gambas3-gb-qt4
        - gambas3-runtime
        - gtk2-engines-pixbuf
        - libaudio2
        - libexpat1
        - libgcrypt20
        - libgtk2.0-0
        - liblz4-1
        - liblzma5
        - libpcre3
        - libsm6
        - libsystemd0
        - libxau6
        - libxdmcp6
        - libxext6
        - libxfixes-dev
        - libxinerama1
        - libxrender1
        - libxt6
        - qt-at-spi
        exclude: []
      files:
        exclude:
        - usr/share/man
        - usr/share/doc/*/README.*
        - usr/share/doc/*/changelog.*
        - usr/share/doc/*/NEWS.*
        - usr/share/doc/*/TODO.*
      test:
        fedora:
          image: appimagecrafters/tests-env:fedora-30
          command: ./AppRun
          use_host_x: true
        debian:
          image: appimagecrafters/tests-env:debian-stable
          command: ./AppRun
          use_host_x: true
        arch:
          image: appimagecrafters/tests-env:archlinux-latest
          command: ./AppRun
          use_host_x: true
        centos:
          image: appimagecrafters/tests-env:centos-7
          command: ./AppRun
          use_host_x: true
        ubuntu:
          image: appimagecrafters/tests-env:ubuntu-xenial
          command: ./AppRun
          use_host_x: true
    AppImage:
      arch: null
      update-information: guess
      sign-key: None


Fixing-up the recipe
====================

Sadly appimage-builder is not capable yet of generating perfect recipes in the first ron for gambas3 applications so we
will have to do some manual tuning.


AppImage architecture
---------------------

If we proceed to run `appimage-builder` without fixing anything we will be prompted with an error like this:

.. code-block:: shell

    schema.SchemaError: Key 'AppImage' error:
    Key 'arch' error:
    None should be instance of 'str'

Which means that the tool wasn't able to determine the right architecture, of the target application. This happens
because Gambas3 binaries are not regular ELF binaries. Therefore we must set it manually to "x86_64" (see `final recipe L95`_).

.. _final recipe L95: https://github.com/AppImageCrafters/appimage-demo-gambas3/blob/main/AppImageBuilder.yml#L95


Gambas3 environment
-------------------

Now we should be able to run `appimage-builder` but our project will fail on the test with the following error:

.. code-block:: shell

    gbr3: unable to load component: gb.image
    $ ./AppRun FAILED, exit code: 1
    ERROR:appimage-builder:Tests failed

It seems that some of the Gambas resources cannot be found. This is provably because it's expected they to be installed
in the system, but they are in the bundle. To correct this we use the "GB_PATH" environment variable that must point to
the gbc3 binary.

To define a :ref:`recipe_runtime` environment variable add the following to the recipe:

.. code-block:: yaml

  runtime:
    env:
      GB_PATH: $APPDIR/usr/bin/gbr3

Now we can run appimage-builder again. This time the application will start but it may be just an empty window. If this
happens provably we are missing some other gambas3 extensions. Those "should" be resolved by the package manager but
for some reasons the packages didn't include it. Therefore, we will have to manually include them.

In my case I had to add the following packages to the apt include list:

- gambas3-gb-form
- gambas3-gb-gtk3


Entrypoint
----------

Now when we run appimage-builder almost all test with the exception of Centos. `appimage-builder` uses a custom AppRun
binary that configures the bundle runtime environment before calling the application and when an external application
is called this configuration is removed. As the Gambas3 binaries are not ELF files the execution flow is going out at
some point and returning which cases that a part of those settings are lost. To prevent this we must use as the
Gambas3 interpreter as entrypoint for our bundle and pass the application binary as argument as follows:

.. code-block:: yaml

    AppDir:
      app_info:

        exec: usr/bin/gbr3
        exec_args: $APPDIR/usr/bin/appimage-demo-gambas3.gambas $@



With this fix our application should run in all the tests scenarios and is ready to be shipped.


Resume
======

To pack a Gambas3 application using the AppImage format we need to:

- deploy the application binary inside the AppDir

- include all the Gambas3 resources and plugins packages in the recipe

- set the environment variable "GB_PATH" to $APPDIR/usr/bin/gbr3

- set usr/bin/gbr3 as entry point and pass the application binary path as argument


A working recipe can be found `here`_. You can use it as starting point instead of going all the troubleshooting of
the above steps.

.. _here: https://github.com/AppImageCrafters/appimage-demo-gambas3/blob/main/AppImageBuilder.yml


What's next
-----------

You may also want to check the following sections:

- :ref:`advanced-updates`

- :ref:`advanced-signing`.

- :ref:`recipe`


