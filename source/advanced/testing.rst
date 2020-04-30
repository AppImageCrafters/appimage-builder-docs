.. _advanced-testing:

"""""""""""""""""""""""""""
Testing and Troubleshooting
"""""""""""""""""""""""""""

Here you will find information regarding how to test and troubleshoot your AppImage.

==========================
appimage-builder AppImages
==========================

AppImages built using ``appimage-builder`` include almost every library and resource required by the bundled application
to run. This allows to execute it in almost every GNU/Linux with ``glibc`` system. But there are some libraries that
cannot be embed for technical reasons. The most relevant is ``libGL`` for NVidia, it's a requirement that the client
side driver version to be equal to the kernel side. Therefore the graphic stack libraries and others related to then
are excluded.

This leads us with a bundle that at runtime it uses some libraries from the system and others from the bundle. If the
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

- Arch
- Fedora
- Debian
- Ubuntu
- Centos

Those distributions are between the most populars or are base for others so if your app work there it has a high
provability to work on derivatives.

The whole docker images list can be found: https://hub.docker.com/repository/docker/appimagecrafters/tests-env

For details on how to setup the tests cases check the :ref:`recipe_version_1_test` specification.

===============
Troubleshooting
===============


Missing libraries
=================

In some scenarios your application may crash on a certain system. This is usually happens because a required library
is not being embed. To identify the culprit run your application using ``LD_DEBUG=libs``. This will print to the
standard output the information related to the shared libraries loading an unloading.


The output will look like this:

.. code-block:: shell

      5491:     find library=libpthread.so.0 [0]; searching
      5491:      search cache=/etc/ld.so.cache
      5491:       trying file=/lib/x86_64-linux-gnu/libpthread.so.0
      5491:
      5491:
      5491:     calling init: /lib/x86_64-linux-gnu/libpthread.so.0

In this case ``libpthread.so.0`` is found and initialized. As we will have a missing library we have to look for
those output blocks where there is a ``find library`` with out a ``init:``. To do it in a test inside docker use
the following snippet:

.. code-block:: shell

      test:
        debian:
          image: appimagecrafters/tests-env:fedora-30
          command: "./AppRun"
          use_host_x: True
          env:
            - LD_DEBUG=libs

More information about the glibc loader debug information can be found on the tool `manual pages`_.
.. _manual pages: http://man7.org/linux/man-pages/man8/ld.so.8.html

To fix this issue just add to your bundle the package that provides this library.

Missing resources
=================

To detect which resource files (settings files, icons, database files or others) are being used by the application we
can use ``strace``. Specifically you can trace ``openat`` calls like this:

.. code-block:: shell

    $strace -e trace=openat ls

    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libselinux.so.1", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libpcre.so.3", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libdl.so.2", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libpthread.so.0", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/proc/filesystems", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
    openat(AT_FDCWD, ".", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
    appimage-appsdir   AppImageServices  builder       builder-tests-env  libappimage                  TheAppImageWay
    appimage-firstrun  apprun            builder-docs  cli-tool           plasma-appimage-integration

Fixing this kind or issues is a bit more complicated as the path to the resources are sometime fixed in the source code.
If it's possible you can try patching the binaries but the recommended solution is to modify the source code to resolve
the resource files from a relative location. For this purpose you can use a configuration file next to the main binary
or environment variables.
