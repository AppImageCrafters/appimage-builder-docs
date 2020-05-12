.. _intro-tutorial:

""""""""
Tutorial
""""""""

In this page is explained how to build an AppImage for a simple Qt/Qml application. The tutorial is meant to be
performed in a Ubuntu system where **appimage-builder** have been installed. Check the :ref:`intro-install`
for instructions. The application code can be found `here`_.

.. _here: https://www.opencode.net/azubieta/qt-appimage-template

===================
Build Configuration
===================

To pack an application as AppImage we need to configure it as it were to be installed in a regular GNU/Linux
system. That implies using a ``Release`` configuration and setting as ``/usr`` as installation prefix. But
instead of installing it to our root dir ``/``, it's going to be installed to ``AppDir``. This directory will
be used later to also deploy the application dependencies.

It's recommended that the applications install procedure also deploys a desktop entry and a icon for the
application, as in a regular installation. These resources will be used to infer the bundle metadata.

.. code-block:: shell

    git clone https://www.opencode.net/azubieta/qt-appimage-template.git
    cd qt-appimage-template
    cmake . -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr
    make install DESTDIR=AppDir


=================
Recipe Generation
=================

Once the application binaries are deployed to an ``AppDir`` we proceed to run ``appimage-builder --generate``. This
will proceed to read the application executable from the desktop entry and launch it. The application will shows up
you can proceed to close it. In complex applications where external plugins are used it's recommended to test all
the features before closing, this will make sure that all runtime dependencies are identified.

Now the tool proceeds to confirm the bundle metadata. At this point you can customize the application id, name,
icon, version, executable path, execution arguments and the target architecture.


.. code-block:: shell

    ? ID [Eg: com.example.app] : QtQuickControls2Application
    ? Application Name : Qt AppImage Template
    ? Icon : QtQuickControls2Application
    ? Version : latest
    ? Executable path relative to AppDir [usr/bin/app] : usr/bin/qt-appimage-template
    ? Arguments [Default: $@] : $@
    ? Architecture :  amd64

Once done the recipe is printed to the standard output and to a file named ``AppImageBuilder.yml``

Filling the gaps
================

If you open the ``AppImageBuilder.yml`` file you will find along the apt source line configuration
a set of empty ``key_url`` entries. Those cannot be resolved currently by the tool and must be filled
manually.

Apt repositories key urls are usually located in the repository root. In the case of the Ubuntu
official repositories and PPAs they can be found in the `Ubuntu keyserver`_.

.. _Ubuntu keyserver: http://keyserver.ubuntu.com/

There is no need to set a ``key_url`` for each ``sourceline`` if it was set before as all the keys are
added to the same keyring. Let's remove all the empty ``key_url`` entries to make apt complain about
the missing keys and run ``appimage-builder``.

You will see something like this, check the line ends:

.. code-block:: text

    INFO:apt-get:Err:2 http://archive.ubuntu.com/ubuntu bionic-updates InRelease
    INFO:apt-get:The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 3B4FE6ACC0B21F32


To make an ubuntu keyring url you can use the following snippet, replace ``<KEY ID>`` by the value that comes
after the NO_PUBKEY in the `appimage-builder` output:

.. code-block:: text

    http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x<KEY ID>
    http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3B4FE6ACC0B21F32


Deploying dependencies
======================

Run ``appimage-builder --skip-test --skip-appimage`` to deploy the dependencies into to ``AppDir``. Notice we all
intentionally skipping the test and AppImage generation steps. Once the process complete should have a working
``AppDir``. To do a quick test you can run ``AppDir/AppRun`` and the application will show up.


==================
Testing the AppDir
==================

So far we should have a functional AppDir, which can be tested by running ``AppDir/AppRun``. This will let's know if
the bundle will run on our system. To know if it will run on other systems we can make use of docker. Official docker
images tend to be minimal so they are great to test our bundle.

appimage-builder provides the means to automate the :ref:`recipe_version_1_test` process. All that you have to do is
specify the docker image to be used and the command to start the application. In case of an application with a
graphical interface set ``use_host_x`` to ``True``. This will share the host X11 server with the docker container.

The generated recipe already comes with a set of tests configured, to run then use:

.. code-block:: text
   appimage-builder --skip-build --skip-appimage

**NOTE**: If the docker images are not in your system it may take a while to download.

Once all the tests cases are completed successfully your ``AppDir`` is ready to be transformed into an AppImage.

*Two important notes on testing inside docker*:

- use docker images that include X11 libraries when testing graphic applications, like the `ones here`_.
- applications with graphical interface will stay running after they are started, therefore you will
  have to manually close then to proceed with the next test case.

.. _ones here: https://hub.docker.com/repository/docker/appimagecrafters/tests-env

============================
Bundling everything together
============================

You have made and tested and ``AppDir`` containing your application binaries and it's dependencies. The final step
is to generate the AppImage as follows:

.. code-block:: text
   appimage-builder --skip-build --skip-test


===========
What's next
===========

The next steps for you is to learn how to do :ref:`advanced-updates` and :ref:`advanced-signing`. You may also want
to check the recipe specification :ref:`recipe_version_1` for advanced tuning.

Thanks for your interest!

