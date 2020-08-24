.. _advanced-troubleshooting:

"""""""""""""""
Troubleshooting
"""""""""""""""

Resulting AppImage can be defective for several reasons here we will explore then and explain the possible solutions.

==================
Bundle information
==================

The first thing to check when Appimage is created is the ``.bundle.yml`` file. It's located in the AppDir root.
This file contain a resume of the packages included in the bundle and the libraries it expect to be present in
the system. You can inspect then too look for missing packages or undesired external dependencies.

**NOTE**: This file is only generated for AppImages built using ``appimage-builder`` >= v0.5.3.


appimage-builder
==================
``appimage-builder`` may be used with two different log levels in its arguments. 
The default ``--log`` level is ``INFO``,
and the more informative ``--log`` level is ``DEBUG``.
Users may specify log arguements in the ``appimage-builder``
command using the ``--log LOGLEVEL`` argument where 
"LOGLEVEL" is either "INFO" or "DEBUG".
e.g. ``appimage-builder --log DEBUG --generate``

.. code-block:: text

    usage: appimage-builder [-h] [--recipe RECIPE] [--log LOGLEVEL < INFO | DEBUG> ] 
                        [--skip-script] [--skip-build] [--skip-tests]
                        [--skip-appimage] [--generate]

    AppImage crafting tool
    optional arguments:
     -h, --help       show this help message and exit
     --recipe RECIPE  recipe file path (default: $PWD/AppImageBuilder.yml)
     --log LOGLEVEL   logging level (default: INFO, debug: DEBUG, e.g. appimage-builder --log DEBUG --generate)
     --skip-script    Skip script execution
     --skip-build     Skip AppDir building
     --skip-tests     Skip AppDir testing
     --skip-appimage  Skip AppImage generation
     --generate       Try to generate recipe from an AppDir


appimage-inspector
==================

``appimage-inspector`` is a tool for inspecting AppImages and AppDirs. It's shipped along with appimage-builder
since ``v0.5.3``. This tool allow us to query information about a given target bundle.

.. code-block:: text

    usage: appimage-inspector [-h] [--log LOGLEVEL] [--print-needed]
                              [--print-runtime-needed]
                              [--print-dependants DO_PRINT_DEPENDANTS]
                              target

    AppImage/AppDir analysis tool

    positional arguments:
      target                AppImage or AppDir to be inspected

    optional arguments:
      -h, --help            show this help message and exit
      --log LOGLEVEL        logging level (default: INFO)
      --print-needed        Print bundle needed libraries
      --print-runtime-needed
                            Print bundle needed libraries for the current system
      --print-dependants DO_PRINT_DEPENDANTS
                            Print bundle libraries that depends on


======
Issues
======

Non portable application
========================

Many applications are coded to find their resources in fixed locations. Every time an AppImage runs it's mounted
in a different location, something like ``/tmp/.mountXXXXXX`` where the exes are replaced by random alpha-numeric
characters. This means that the app resource path will change every time it's ran.

Therefore app developers should make their apps configurable at runtime. This can be done by using environment
variables or a configuration file next to the main binary.


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

