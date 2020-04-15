.. _intro-overview:

============================
appimage-builder at a glance
============================

`appimage-builder` is a tool for packaging other applications into AppImages. Any kind of
application can be packaged using this tool and unlike `other AppImage creation tools`_ it can be
used in modern systems and the resulting bundle will be backward compatible.

.. _other AppImage creation tools: https://github.com/linuxdeploy/

**NOTICE**: Only GNU/Linux distributions that contains the **APT** package manager are supported. In
the future other package managers will be added.

Walk-through of an example `appimage-builder` recipe
====================================================

`appimage-builder` uses a recipe to configure the AppImage creation process. Here's an example of
a recipe for building a Bash AppImage:


.. code-block::

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
        arch: amd64
        sources:
          - sourceline: 'deb [arch=amd64] http://archive.ubuntu.com/ubuntu/ bionic main'
            key_url: 'http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3b4fe6acc0b21f32'

        include:
          - bash
          - coreutils
        exclude:
          - dpkg

      test:
        centos:
          image: centos:6
          command: "./AppRun -c \"ls\""
          use_host_x: True


    AppImage:
      update-information: None
      sign-key: None
      arch: x86_64

Put this in a file named `appimage-builder.yml` and run the tool using the following command:

.. code-block::

    appimage-builder --recipe appimage-builder.yml

When this finishes you will have in current working directory an AppImage file like this: `bash-4.4.20-x86_64.AppImage`

To execute it just do:

.. code-block::

    ./bash-4.4.20-x86_64.AppImage


What just happened?
===================

When you ran the command ``appimage-builder --recipe appimage-builder.yml`` the tool read the recipe file and executed
the following tasks:

1. APT configuration
--------------------

An APT configuration will be generated in the appimage-builder-cache directory. This directory will hold
a cache of the resources used to build the AppImage. In the listed sources will be configured as APT
sources and the keys will be added to an internal keyring.

Then ``apt update`` will be executed using the newly created configuration.

The packages listed in the `AppDir\apt\exclude` section will be set as 'Installed' to prevent their inclusion.

2. Binaries deployment
----------------------

The packages listed in the `AppDir\apt\included` along with their dependencies will be downloaded and deployed
into de `AppDir` `path`. Only the `glibc` packages will be deployed to an special location on `opt/libc` so they
can be easily ignored at runtime.

3. Runtime Setup
----------------

This step has the purpose of making all the embed resources available to the application at runtime. Therefore
it's aid by a set of helpers that are activated depending on whether some binaries are found. Those helpers will
add configuration files to the bundle and set the required environment variables to the `.env` file.

By example the Qt helper will be used if `libQt5Core.so.5` is found. This Qt helper will create the required
`qt.conf` files to ensure that the Qt plugins are properly resolved.

Finally the AppRun and libapprun_hooks.so files are added. The first one loads the `.env` file and executes the
application. The other makes sure that the environment configuration that is required to execute your AppImage
doesn't propagate to other applications executed.

4. Bundling
-----------

Finally the whole AppDir is compressed into an squashfs file and appended to a runtime binary. This binary does
the function of mounting the bundle at runtime and calling the AppRun in it. It also contains the update
information and signature of the AppImage.

To perform this tasks appimagetool is used. If everything went OK, the output should be a nice AppImage file.


What else?
==========

You have seen how to make recipe for Bash and how it's used to build an AppImage. But this is just the surface.
With appimage-builder you can create recipes for almost any kind of glibc based applications. We invite you to
check the examples sections to see other recipes for different frameworks and technologies.

Also it's important to say that contents of your bundle are not limited to those resources available in some
APT repository. You can also include self build binaries, check the script section in the recipe specification
for more details.


What’s next?
============

The next steps for you is to `install appimage-builder`_, `follow through the tutorial`_ to learn how to create
recipes for more complex applications and join the `appimage community`_. Thanks for your interest!
