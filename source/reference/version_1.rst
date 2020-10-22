.. _recipe_version_1:

""""""""""""""""""
Version: 1 (Draft)
""""""""""""""""""

In this section is described the recipe specification and all the components that affects its behaviour.

**THIS DOCUMENT IS UNDER CONSTRUCTION AND MAY CHANGE UNTIL THE 1.0 RELEASE OF APPIMAGE-BUILDER**

=====================
Environment variables
=====================

Environment variables can be placed anywhere in the configuration file; must have ``!ENV`` before them and
specified in this format to be parsed: ``${VAR_NAME}``.

.. code-block:: yaml

    AppDir:
      app_info:
        version: !ENV ${APP_VERSION}
        exec: !ENV 'lib/${GNU_ARCH_TRIPLET}/qt5/bin/qmlscene'
    AppImage:
      arch: !ENV '${TARGET_ARCH}'
      file_name: !ENV 'myapp-${APP_VERSION}_${TIMESTAMP}-${ARCH}.AppImage'


**NOTE**: To mix variables that must be parsed with other that not use the following
syntax: ``!ENV '${PARSED_VAR}-"$NON_PARSED_VAR"'``

.. _recipe_version_1_script:

======
script
======

The script section consists of a list of shell instructions. It should be used to deploy
your application binaries and resources or other resources that cannot be resolved
using the package manager.

Example of how to deploy a regular ``cmake`` application binaries.

.. code-block:: yaml

    script:
      - cmake .
      - make DESTDIR=Appdir install


In the cases where you don't use a build tool or it doesn't have an install feature,
you can run any type of command in this section. In the example below a QML file
is deployed to be used as part of a pure QML application.

.. code-block:: yaml

    script:
      - mkdir -p AppDir
      - cp -f main.qml AppDir/

.. _recipe_version_1_appdir:

======
AppDir
======

The `AppDir`_ section is the heart of the recipe. It will contain information about the
software being packed, its dependencies, the runtime configuration, and the tests.

The execution order is as follows:
- bundle dependencies
- configure runtime
- run tests

.. _recipe_version_1_section_scripts:

---------------
Section scripts
---------------

It's possible to insert scripts before and after the bundle and runtime steps. Those can be used to perform
additional tweaks to the AppDir before proceeding with the tests.

The allowed keys are:

- before_bundle
- after_bundle
- before_runtime
- after_runtime

This is an example of how to use the after bundle to patch a configuration file.

.. code-block:: yaml

    AppDir:
      after_bundle: |
        echo "source /etc/timidity/freepats.cfg" | tee AppDir/etc/timidity/timidity.cfg


----
path
----
Path to the AppDir.

.. code-block:: yaml

    AppDir:
      path: ./AppDir


.. _recipe_version_1_app_info:

--------
app_info
--------

- **id**: application id. Is a mandatory field and must match the application desktop entry name without the ``.desktop``
  extensions. It's recommended to used reverse domain notation like *org.goodcoders.app*.
- **name**: Application name, feel free here.
- **icon**: Application icon. It will be used as the bundle icon. The icon will be copied from
  ``$APPDIR/usr/share/icons`` or from your system folder ``/usr/share/icons``.
- **version**: application version.
- **exec**: path to the application binary. In the case of interpreted programming languages such as Java, Python or
  QML, it should point to the interpreter binary.
- **exec_args**: arguments to be passed when starting the application. You can make use of environment variables such
  as ``$APPDIR`` to refer to the bundle root and/or ``$@`` to pass arguments to the binary.

.. code-block:: yaml

  app_info:
    id: org.apppimagecrafters.hello_qml
    name: Hello QML
    icon: text-x-qml
    version: 1.0
    exec: usr/lib/qt5/bin/qmlscene
    exec_args: $@ ${APPDIR}/main.qml

.. _recipe_version_1_apt:

---
apt
---

 The apt section is used to list the packages on which the app depends and the sources
 to fetch them.

- **arch**: Binaries architecture. It must match the one used in the sources configuration.
- **sources**: apt sources to be used to retrieve the packages.

    * **sourceline**: apt configuration source line. It's recommended to include the Debian architecture on
      it to speed up builds.
    * **key_url**: apt key to validate the packages in the repository. An URL to the actual
      key is expected.
- **include**: List of packages to be included in the bundle. Package dependencies will
  also be bundled.
- **exclude**: List of packages to *not* bundle. Use it to exclude packages
  that aren't required by the application.

.. code-block:: yaml

   apt:
    arch: i386
    sources:
      - sourceline: 'deb [arch=i386] http://mx.archive.ubuntu.com/ubuntu/ bionic main restricted universe multiverse'
        key_url: 'http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3b4fe6acc0b21f32'

    include:
      - qmlscene
      - qml-module-qtquick2
    exclude:
      - qtchooser


The tool generates a cache where the downloaded packages and other auxiliary files are
stored, it will be located in the current work dir with the name **appimage-builder-cache**.
It's safe to erase it and should not be included in your VCS tree.


-----
files
-----

The files section is used to manipulate (include/exclude) files directly.
`Globing expressions`_ can be used to match multiple files at
once.

.. _Globing expressions: https://docs.python.org/3.6/library/glob.html#module-glob

- **include**: List of absolute paths to files. The file will be copied under the same name
  inside the AppDir. i.e.: ``/usr/bin/xrandr`` will end at ``$APPDIR/usr/bin/xrandr``.
- **exclude**: List of relative globing shell expressions to the files that will
  not be included in the `AppDir`_. Expressions will be evaluated relative to the
  `AppDir`_. Use it to exclude unrequired files such as *man* pages or development
  resources.

.. code-block:: yaml

  files:
    exclude:
      - usr/share/man
      - usr/share/doc/*/README.*
      - usr/share/doc/*/changelog.*
      - usr/share/doc/*/NEWS.*
      - usr/share/doc/*/TODO.*

.. _recipe_version_1_test:

----
test
----

The `test` section is used to describe test cases for your final AppImage. The AppDir as it's can be already executed.
Therefore it can be placed inside a Docker container and executed. This section eases the process. Notice that you will
have to test that the application is properly bundled and isolated, therefore it's recommended to use minimal Docker
images (i.e.: with no desktop environment installed).

**IMPORTANT**: Docker is required to be installed and running to execute the tests.

Each test case has a name, which could be any alphanumeric string and the
following parameters:

- **image**: Docker image to be used.
- **command**: command to execute.
- **use_host_x**: whether to share or not the host X11 session with the container.
  *This feature may not be supported by some containers as it depends on X11*.
- **env**: list of environment variables to be passed to the Docker container.

.. code-block:: yaml

  test:
    fedora:
      image: fedora:26
      command: "./AppRun main.qml"
      use_host_x: True
    ubuntu:
      image: ubuntu:xenial
      command: "./AppRun main.qml"
      use_host_x: True

-------
runtime
-------

Advanced runtime configuration.

- **env**: Environment variables to be set at runtime.
- **path_mappings**
    Setup path mappings to workaround binaries containing fixed paths. The mapping is performed at runtime by
    intercepting every system call that contains a file path and patching it. Environment variables are supported
    as part of the file path.

    Paths are specified as follows: <source>:<target>

    Use the *$APPDIR* environment variable to specify paths relative to it.




.. _AppRun project repo: https://github.com/appimagecrafters/AppRun


 .. code-block:: yaml

  runtime:
    path_mappings:
      - /etc/gimp/2.0/:$APPDIR/etc/gimp/2.0/
    env:
      PATH: '${APPDIR}/usr/bin:${PATH}'

========
AppImage
========

The AppImage section refers to the final bundle creation. It's basically a wrapper over ``appimagetool``

- **arch**: AppImage runtime architecture. Usually, it should match the embed binaries architecture, but a different
  —compatible one— could be used. For example, i386 binaries can be used in an AMD64 architecture.
- **update-info**: AppImage update information. See `Making AppImages updateable`_.
- **sign-key**: The key to sign the AppImage. See `Signing AppImage`_.
- **file_name**: Use it to rename your final AppImage. By default it will be named as follows:
  ``${AppDir.app_info.name}-${AppDir.app_info.version}-${AppImage.arch}.AppImage``. Variables are not supported yet and
  are used only for illustrative purposes.

.. _Making AppImages updateable: https://docs.appimage.org/packaging-guide/optional/updates.html
.. _Signing AppImage: https://docs.appimage.org/packaging-guide/optional/signatures.html
