.. _intro-tutorial:

""""""""""""""""""""""""""
AppImage creation tutorial
""""""""""""""""""""""""""

In this page is explained how to build an AppImage using appimage-builder from scratch. We will be covering the
following topics:

- application dependencies lockup
- recipe writing
- AppDir testing


========================
Application dependencies
========================

To build an appimage-builder recipe you need to know which packages provide the dependencies that are required by your
application. If you are the application author and your development environment is Debian, Ubuntu, or some other
distribution that uses the APT package manager you should be familiar with such packages. In the other hand, if you are
not the application author or you are not familiar with APT based system you will need to find out such packages. Here
we will describe how to do it.

Does the target application has an existent Debian package ?
============================================================

If the application you are targeting already has a Debian package the process is simple. You query the dependencies
using  ``apt-cache depends <package name>`` or you can manually download the package from the repository and run
``dpkg-deb --info <deb file name>``.

There you will have the list of packages your applications depends on.

Building the application in a Debian based system
=================================================

In this scenario you will need an APT based system to build your application. The build process will require you to
install both the application build dependencies and the runtime ones. This could be a bit complicated luckily there
is a tool to find what package provides a given file.

Use `apt-file`_ to find out which packages provides the files you need to build your application, for example,
``apt-file find cmake`` and ``apt-get install <package name>`` to install the package.

.. _apt-file: https://wiki.debian.org/apt-file


Runtime dependencies
====================

Once you have a working binary we will proceed to find which are files that are really needed by your application at
runtime. To do so, we will execute the application using the following instructions:

.. code-block:: shell

   APP=./my_app
   LIBRARIES=$(LD_DEBUG=libs $APP 2>&1 | grep 'init: ' | rev | cut -f 1 -d' ' | rev)
   for LIB in $LIBRARIES; do echo -n '    - '; dpkg-query -S $LIB | cut -d':' -f 1 ; done | sort -u

The above **Bash** code snippet will execute ``./my_app`` and will list all the libraries that are opened at runtime. Therefore
it's important that while the application is open you use/test almost every feature. This will guarantee that even such
libraries that are plugins are loaded and listed. Depending on the amount of dependencies your app has it may take a
while to complete.

Notice that we use ``dpkg-query -s`` instead of ``apt-file find`` to resolve package names. As the files are already in
installed in your system ``dpkg-query`` will provide a quicker result.

At this point you should have a package list like this (but with much more entries):

.. code-block:: shell

    - libc6
    - libpcre3
    - libselinux1

That's all you need to place in ``AppDir >> apt >> include``.

A note on X11 client and the graphics stack
-------------------------------------------

The `linux graphics stack`_ is a quite complicated and entangled thing and thanks to NVidea it's not portable between
different kernel versions. Therefore it shoul not be included in the AppImages (if included they will not work on NVidea
systems). The following packages most not be placed in the include list. Regular expresion were used to match several
packages.

Packages that should not be bundled:

- ``libxcb.*``
- ``libgl.*``
- ``libegl.*``
- ``libx11.*``

.. _linux graphics stack: https://blogs.igalia.com/itoral/2014/07/29/a-brief-introduction-to-the-linux-graphics-stack/

=======================
Writing down the recipe
=======================


Once all the `application dependencies`_ are identified we can proceed to write down our recipe file. Let's make an
AppImage for `qt demo app`_.

.. _qt demo app: https://www.opencode.net/azubieta/qt-appimage-template


Let's start by creating a folder named ``qt-demo-appimage`` and place inside a file name ``appimage-builder.yml``. The
first thing that we should place in the file is the appimage-builder format to be used, in this case ``1``.

.. code-block:: yaml

    version: 1


Building the application binaries
=================================

Then we proceed to add the application build instructions. For this purpose we will use the
:ref:`recipe_version_1_script` section. There you can execute all kinds of shell scripts.


.. code-block:: yaml

    script:
     - git clone https://www.opencode.net/azubieta/qt-appimage-template.git
     - cd qt-appimage-template; cmake . -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release
     - cd qt-appimage-template; DESTDIR=../AppDir make install

With those instructions we download the project source code. Then it's configured for installation and finally
it's installed to ``AppDir``.

Describing the application
==========================

In order to provide a consistent user experience it's required to fill some information about he application. This
must be placed inside the :ref:`recipe_version_1_app_info` section.

.. code-block:: yaml

    AppDir:
      app_info:
            id: net.appimage-builder.demo-app
            name: AppImage Builder Demo App
            icon: QtQuickControls2Application
            version: latest
            exec: usr/bin/qt-appimage-template
            exec_args: "$@"

Deploying dependencies
======================

Dependencies are deployed according to the :ref:`recipe_version_1_apt` section specification. You need to specify
the binaries target architecture, the source lines as they are usually written in ``/etc/apt/sources.list``, and
the URLs of the sources keys.

Then, in the ``include`` section are list your application dependencies. You will not have to write every single
package you application depends on, instead you should only place those that are direct dependencies.

**NOTE**: It's important to specify the ``path`` entry inside the ``AppDir`` as this will point out where the downloaded packages
should be extracted.


.. code-block:: yaml

    AppDir:
      path: ./AppDir

      apt:
        arch: amd64
        sources:
          - sourceline: 'deb [arch=amd64] http://archive.ubuntu.com/ubuntu/ bionic main restricted universe multiverse'
            key_url: 'http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3b4fe6acc0b21f32'

        include:
            - libqt5core5a
            - libqt5dbus5
            - libqt5gui5
            - libqt5network5
            - libqt5printsupport5
            - libqt5qml5
            - libqt5quick5
            - libqt5quickcontrols2-5
            - libqt5quicktemplates2-5
            - libqt5svg5
            - libqt5texttospeech5
            - libqt5widgets5
            - libqt5x11extras5
            - libqt5xml5
            - qml-module-qtquick2
            - qml-module-qtquick-controls2
            - qml-module-qtquick-layouts
            - qml-module-qtquick-templates2
            - qml-module-qtquick-window2

Testing the AppDir
==================

So far we should have a functional AppDir, which can be tested by running ``AppDir/AppRun``. This will let's know if
the bundle will run on our system. To know if it will run on other systems we can make use of docker. Official docker
images tend to be minimal so they are great to test our bundle.

appimage-builder provides the means to automate the :ref:`recipe_version_1_test` process. All that you have to do is specify the docker image to
be used and the command to start the application. In case of an application with a graphical interface set
``use_host_x`` to ``True``. This will share the host X11 server with the docker container.

*Two important notes on testing inside docker*:

- use docker images that include X11 libraries when testing graphic applications.
- applications with graphical interface will stay running after they are started, therefore you will have to manually
  close then to proceed with the next test case.


.. code-block:: yaml

      test:
        centos:
          image: appimage-builder/test-env:ubuntu-xenial
          command: "./AppRun"
          use_host_x: True


Bundling everything together
============================

Once you have a running AppDir you can proceed to create the AppImage bundle. To do so just specify the architecture
of the AppImage runtime to be used and you're done. You can see which architectures are available by checking the
`AppImageKit project releases`_

.. _AppImageKit project releases: https://github.com/AppImage/AppImageKit/releases/

.. code-block:: yaml

    AppImage:
        arch: x86_64

===============
Complete recipe
===============

To test the whole recipe copy the following snippet in a file named ``appimage-builder.yml`` and run:
``appimage-builder --recipe appimage-builder.yml``.

Using a docker container is recommended so you don't mess your system with the application build dependencies. We have
some docker images ready for you to use check out: https://hub.docker.com/orgs/appimagecrafters


.. code-block:: yaml

    version: 1

    script:
     - git clone https://www.opencode.net/azubieta/qt-appimage-template.git
     - cd qt-appimage-template; cmake . -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release
     - cd qt-appimage-template; DESTDIR=../AppDir make install

    AppDir:
      path: ./AppDir

      app_info:
            id: net.appimage-builder.demo-app
            name: AppImage Builder Demo App
            icon: QtQuickControls2Application
            version: latest
            exec: usr/bin/qt-appimage-template
            exec_args: "$@"

      apt:
        arch: amd64
        sources:
          - sourceline: 'deb [arch=amd64] http://archive.ubuntu.com/ubuntu/ bionic main restricted universe multiverse'
            key_url: 'http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3b4fe6acc0b21f32'

        include:
            - libqt5core5a
            - libqt5dbus5
            - libqt5gui5
            - libqt5network5
            - libqt5printsupport5
            - libqt5qml5
            - libqt5quick5
            - libqt5quickcontrols2-5
            - libqt5quicktemplates2-5
            - libqt5svg5
            - libqt5texttospeech5
            - libqt5widgets5
            - libqt5x11extras5
            - libqt5xml5
            - qml-module-qtquick2
            - qml-module-qtquick-controls2
            - qml-module-qtquick-layouts
            - qml-module-qtquick-templates2
            - qml-module-qtquick-window2

      test:
        ubuntu-xenial:
          image: appimage-builder/test-env:ubuntu-xenial
          command: "./AppRun"
          use_host_x: True

    AppImage:
        arch: x86_64


