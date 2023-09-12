===================
Flutter Application
===================

`Flutter`_ is Google’s UI toolkit for building natively compiled applications for mobile, web,
and desktop from a single codebase. In page you will learn how to pack a Flutter desktop project for Linux using
the AppImage format.

For the purpose we will be using a simple hello world application which is available at: https://github.com/AppImageCrafters/appimage-builder-flutter-example

.. _Flutter: https://flutter.dev/

Preparing your system
=====================

- `Install Flutter`_
- `Install appimage-builder`_

.. _Install Flutter: https://flutter.dev/docs/get-started/install/linux
.. _Install appimage-builder: https://appimage-builder.readthedocs.io/en/latest/intro/install.html

Building the flutter app
========================

We will use the linux desktop target of Flutter to generate our application binaries. This target is currently only
available in the beta channel therefore we need to enable it. Once it's enable we can generate the binaries.


.. code-block:: shell

    # enable desktop builds
    flutter channel beta
    flutter upgrade
    flutter config --enable-linux-desktop

    # build desktop release
    flutter build linux

Our application binaries should be somewhere inside the build dir, usually `build/linux/x64/release/bundle`. We will
copy this this folder to our work dir as AppDir:

`cp build/linux/x64/release/bundle $PWD/AppDir`



Generating the recipe
=====================

We will use the `--generate` method to draft an initial recipe for our project. In the process you'll be prompted
with a set of questions that will help the tool to process your project.

Notice that the application must run in order to properly analyse it's runtime dependencies.

.. code-block:: shell

    appimage-builder --generate

    Basic Information :
    ? ID [Eg: com.example.app] : com.example.flutter_hello
    ? Application Name : Flutter Hello
    ? Icon : utilities-terminal
    ? Version : latest
    ? Executable path relative to AppDir [usr/bin/app] : hello_flutter
    ? Arguments [Default: $@] : $@
    ? Update Information [Default: guess] : guess
    ? Architecture :  amd64

Generating the AppImage
=======================

At this point we should have a working recipe that can be used to generate an AppImage from our flutter project. To
do so execute `appimage-builder` and the packaging process will start.

After deploying the runtime dependencies to the AppDir and configuring then the tool will proceed to test the
application according to test cases defined in the recipe. This will give us the certainty that our app runs in the
target system. It's up to you to manually verify that all features work as expected. Once the tools completes its
execution you should find an AppImage file in your current work dir.

.. code-block:: shell

    appimage-builder --recipe AppImageBuilder.yml


Polishing the recipe
====================

Hooray! You should have now an AppImage that can be shipped to any GLibC based GNU/Linux distribution. But there is some
extra work to do. The recipe we have is made to freeze the current runtime which include certain parts of your system
(such as theme modules) that may not be required in the final bundle. Therefore we will proceed to remove them from
the recipe.

Grab your favourite text editor and open the `AppImageBuilder.yml` file.

Deploy binaries from the script section
---------------------------------------

Every time we run appimage-builder we need to first copy the application binaries into the AppDir. This step can
be made part of the recipe as using the script section as follows:

.. code-block:: yaml

    script:
     - rm -rf AppDir | true
     - cp -r build/linux/x64/release/bundle $APPDIR

Notice the usage of the APPDIR environment variable, this is exported by appimage-builder at runtime.


Refine the packages include list
--------------------------------


In the `apt > include` section you may find a list of packages. Those packages that are tightly related to your
desktop environment (in my case KDE) or to some external system service can be removed in order to save some space but
you will have to always validate the resulting bundle using the tests cases. You can even try to boil down your list
to only `libgtk-3-0` and manually add the missing libs (if any).



Final recipe
============

After following the tutorial you should end with a recipe similar to this one. It could be used as starting point if
you don't want to use the `--generate` method.

.. code-block:: yaml

    # appimage-builder recipe see https://appimage-builder.readthedocs.io for details
    version: 1
    script:
     - rm -rf AppDir || true
     - cp -r build/linux/x64/release/bundle AppDir
     - mkdir -p AppDir/usr/share/icons/hicolor/64x64/apps/
     - mv AppDir/lib/ AppDir/usr/
     - cp flutter-mark-square-64.png AppDir/usr/share/icons/hicolor/64x64/apps/
    AppDir:
      app_info:
        id: org.appimagecrafters.hello-flutter
        name: Hello Flutter
        icon: flutter-mark-square-64
        version: latest
        exec: hello_flutter
        exec_args: $@
      apt:
        arch: amd64
        allow_unauthenticated: true
        sources:
        - sourceline: deb http://archive.ubuntu.com/ubuntu/ bionic main restricted universe multiverse
        - sourceline: deb http://archive.ubuntu.com/ubuntu/ bionic-updates main restricted universe multiverse
        - sourceline: deb http://archive.ubuntu.com/ubuntu/ bionic-backports main restricted universe multiverse
        - sourceline: deb http://security.ubuntu.com/ubuntu bionic-security main restricted universe multiverse
        include:
        - libgtk-3-0
        exclude:
        - humanity-icon-theme
        - hicolor-icon-theme
        - adwaita-icon-theme
        - ubuntu-mono
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
      arch: x86_64
      update-information: guess
      sign-key: None
