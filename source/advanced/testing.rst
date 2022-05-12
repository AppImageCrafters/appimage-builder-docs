.. _advanced-testing:

"""""""
Testing
"""""""

`appimage-builder` provides you a simple way of testing the AppImages compatibility with different systems. It uses
docker containers to simulate the runtime environments and runs the applications inside.

==========================
appimage-builder AppImages
==========================

AppImages built using ``appimage-builder`` include almost every library and resource required by the bundled application
to run. This allows to execute it in almost every GNU/Linux with ``glibc`` system. But there are some libraries that
cannot be embed for technical reasons. The most relevant is ``libGL`` for NVidia, it's a requirement that the client
side driver version to be equal to the kernel side. Therefore the graphic stack libraries and others related to then
are excluded.

This leads us with a bundle that at runtime uses some libraries from the system and others from the bundle. If the
ABI or the implementations of those mixed libraries are not compatible the application will crash. Luckily the libraries
developers are careful enough to keep a good backward compatibility and the application works most of the times. But
the only way of being 100% sure is by testing.

There are other resources from the system that our applications uses. An incompatibility with them may also lead to a
crash. Here is a non-extensive list of those:

- icon themes
- fonts
- widgets themes (GTK/QT)
- Alsa / PulseAudio server
- X.Org server
- Wayland server


=========================
Tests in Docker container
=========================

`appimage-builder` provides a way of easily configuring a set of test using Docker containers to make sure your bundle
will work on a given system. Therefore you will need to have a working docker image with the system resources listed
above to make it work.

There is a list of pre-built docker images that you can use for your tests including the following GNU/Linux
distributions:

- Arch  ``appimagecrafters/tests-env:archlinux-latest``
- Fedora ``appimagecrafters/tests-env:fedora-30``
- Debian ``appimagecrafters/tests-env:debian-stable``
- Ubuntu ``appimagecrafters/tests-env:ubuntu-bionic``
- Centos ``appimagecrafters/tests-env:centos-7``

Those distributions are between the most populars or are base for others so if your app work there it has a high
provability to work on derivatives.

The whole docker images list can be found: https://hub.docker.com/r/appimagecrafters/tests-env

For details on how to setup the tests cases check the :ref:`recipe_test` specification.

==================
Recipe tests setup
==================

Tests cases can be described in the recipe file. Those are placed inside the ``AppDir >> test`` section. Bellow you
will find an example of a test case for Debian. To know more about this section check the :ref:`recipe_test`
specification:

.. code-block:: yaml

    AppDir:
        test:
            debian:
              image: appimagecrafters/tests-env:debian-stable
              command: "./AppRun"
              use_host_x: True
              env:
                QT_DEBUG_PLUGINS: 1

===================
Manual test running
===================

Any AppImage/AppDir can be also manually tested using the ``appimage-tester`` command. This is part of
``appimage-builder`` since ``v0.5.2``.

.. code-block:: shell

    usage: appimage-tester [-h] [--log LOGLEVEL] --docker-images DOCKER_IMAGE
                       [DOCKER_IMAGE ...] [--test] [--static-test]
                       target

**NOTE**: Type 1 AppImages need to be extracted or mounted manually before running the tests.

Regular Docker tests
====================

A regular test will try to run the target application inside the specified docker containers. A running X11 server
is required if the app has a GUI.

.. code-block:: shell

    appimage-tester --test ~/MyApp-1.8.4.AppImage   --docker-images 'appimagecrafters/tests-env:debian-stable'


Static Docker tests
===================

Static test will lockup the external dependencies of the given target and will check if all of then are present
in the system contained in the docker image. This does not execute the application.

.. code-block:: shell

    appimage-tester --static-test ~/MyApp-1.8.4.AppImage   --docker-images 'appimagecrafters/tests-env:debian-stable'

**NOTE**: Optional plugins can have runtime dependencies that may not be present in the test system but as they are
optional the app will run properly.

