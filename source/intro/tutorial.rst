.. _intro-tutorial:

""""""""
Tutorial
""""""""

In this page is explained how to build an AppImage for a simple Qt/Qml application. The tutorial is meant to be
performed in a Ubuntu (18.04 or newer) system where **appimage-builder** have been installed. Check the :ref:`intro-install`
for instructions. The application code can be found `here`_.

.. _here: https://www.opencode.net/azubieta/qt-appimage-template

=====================
Compiling the sources
=====================

The first step in our process is to build the application binaries. We will set the install prefix to '/usr' as
appimage-builder expects to find the application resources (desktop entry and the icon) in their standard paths.

.. _DESTDIR: https://www.gnu.org/prep/standards/html_node/DESTDIR.html

.. code-block:: shell

    # install build dependencies
    sudo apt-get install zlib1g-dev git cmake qtdeclarative5-dev qml-module-qtquick-layouts qml-module-qtquick-layouts qml-module-qtquick-controls2

    # get the application source code
    git clone https://www.opencode.net/azubieta/qt-appimage-template.git

    # step into the application source code dir
    cd qt-appimage-template

    # configure the build in 'release' mode using '/usr' as prefix
    cmake . -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr

    # compile the application
    make


====================
Preparing the AppDir
====================

We will use the `DESTDIR`_ make variable to install the binaries using a root different than '/'.
Notice that while many build toolchains such as cmake support this variable it's not standard.

.. code-block:: shell

    # install the application to 'AppDir'
    make install DESTDIR=AppDir

After installing the application into the AppDir we need to verify that it works as expected. Therefore we will
execute it and manually check that the application windows shows and all the components are visible and functional.

If the application fails to run or doesn't shows a window when executed you will need to investigate and solve
the issue before continuing. Notice that applications must be relocatable in order to be put inside and AppImage.

.. code-block:: shell

    # execute the application
    AppDir/usr/bin/appimage-demo-qt5


=====================
Generating the recipe
=====================


``appimage-builder`` is capable of inspecting the runtime dependencies of a given application to create a recipe for
packaging it as AppImage. This can be done using the ``--generate`` argument as follows:

.. code-block:: shell

    # run recipe generation assistant
    $ appimage-builder --generate

The tool will prompt a questionnaire to gather the minimal required information to execute the application. If the a
desktop entry is found in the AppDir it will be used to fill the fields but you will be able to edit all the values.
Make sure of specifying the executable path properly otherwise the execution will fail.

.. code-block:: shell

    > Basic Information :
    > ? ID [Eg: com.example.app] : appimage-demo-qt5
    > ? Application Name : AppImage Demo Qt5
    > ? Icon : appimage-demo-qt5
    > ? Version : latest
    > ? Executable path relative to AppDir [usr/bin/app] : usr/bin/appimage-demo-qt5
    > ? Arguments [Default: $@] : $@
    > ? Update Information [Default: guess] : guess
    > ? Architecture :  amd64

Once the questionnaire is completed the application will be executed. At this point ,ake sure here to test all your
applications features so all the external resources it may use are accessed and detected by the tool. Once your are
done testing close the application normally.

The tool will filter the accessed files, map them to deb packages and refine list to only include those packages that
are not dependencies of others already listed in order to reduce the list size. Finally the recipe will be wrote
in a file named ``AppImageBuilder.yml`` located in the current working directory.

=====================
Creating the AppImage
=====================

Once you have a recipe in place you can call ``appimage-builder`` to create the final AppImage. The tool will perform
the following steps:

Script step
===========

Recipes can include an optional section name ``script``. This can be used to perform the installation of our application
binaries to the AppDir. This is not created by the generator but you can edit the ``AppImageBuilder.yml`` file and
add the following code before calling `appimage-builder`.

This step can be skip using the `--skip-script` argument.

.. code-block:: yaml

    script: |
        # remove any existent binaries
        rm AppDir | true

        # compile and install binaries into AppDir
        cmake . -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr
        make install DESTDIR=AppDir


Build step
==========

This is where the major part of the job is done. The tool will proceed to gather all the dependencies and to configure
the final bundle. Here are some of the actions it will perform:

- setup an independent apt configuration to resolve dependencies and download the packages
- download the packages
- extract the packages into the AppDir
- copy any other file that wasn't found in a package (the ones listed in ``files > include`` )
- remove excluded files (``files > exclude``)

- configure the runtime environment which includes:
    - configure Qt and other frameworks/modules/libraries present in the bundle
    - setup the library and binary paths (LD_LIBRARY_PATH and PATH environment variables)
    - setup the ld-linux interpreter and deploying the `custom AppRun`_ to ensure backward compatibility

.. _custom AppRun: https://github.com/appimagecrafters/AppRun


.. code-block:: shell

    # create the AppImage
    appimage-builder --recipe AppImageBuilder.yml

.. note::
    This step can be skip by passing the argument ``--skip-build``.

Test step
=========

The only way of ensuring that our application will run a given GNU/Linux distribution is by testing it. The tool can
make use of docker to run the AppDir that was created in the build step in different systems according to the
specifications on the recipe test section.

.. code-block:: yaml

    # test section example
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

The application will be executed in each one of the systems listed above. You will have to manually verify that
everything works as expected and close the application so the tests can continue.

.. note::
    The tool will use a set of docker images that can be found here: `docker test environments`_

.. note::
    Downloading the docker images may take a while the first time and the application may seem idle. Please be patient
    or manually download the images using ``docker pull <image>``

    .. _docker test environments: https://hub.docker.com/r/appimagecrafters/tests-env

.. warning::
    Docker must me installed in the system and the user must be able to use it without root permissions. Use the
    following snippet to allow it.

    .. code-block:: shell

        # install docker
        sudo apt-get install docker.io

        # give non root permissions
        sudo groupadd docker
        sudo usermod -aG docker $USER

        # restart the your system


.. note::
    This step can be skip by passing the argument ``--skip-test``. You would like to use this argument when creating
    scripts for packaging your software using Gitlab-Ci, GitHub Actions or other build service.


AppImage step
=============

The tool will make use of ``appimagetool`` to generate the final ApppImage file. The resulting file should be located
in the current working directory.

Congratulations, you should have a working AppImage at this point!

.. note::
    This step can be skip by passing the argument ``--skip-appimage``.

===================
Refining the recipe
===================

While the ``--generate`` argument can be used to create an initial working recipe you will like to inspect and refine
its contents. By example is common to find theme packages being included when those are something quite distribution
specific. You can try removing those packages and run ``appimage-builder`` again (without the ``--generate`` argument)
to validate that the resulting bundle is still functional. Repeat the process until you're happy with the result.

Some ``libc`` related files may also be found in the ``file > include`` section. Those can be safely excluded most of the
times but remember to test.


===========
What's next
===========

The next steps for you is to learn how to do :ref:`advanced-updates` and :ref:`advanced-signing`. You may also want
to check the recipe specification :ref:`recipe_version_1` for advanced tuning.

Thanks for your interest!

