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


`appimage-builder` is capable of inspecting the runtime dependencies of a given application to create a recipe for
packaging it as AppImage. This can be done using the `--generate` argument as follows:

.. code-block:: shell

    # run recipe generation assistant
    $ appimage-builder --generate

The tool will prompt a questionary to gather the minimal required information to execute the application. If the a
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

Once the questionary is completed the application will be executed. At this point ,ake sure here to test all your
applications features so all the external resources it may use are accessed and detected by the tool. Once your are
done testing close the application normally.

The tool will filter the acessed files, map them to deb packages and refine list to only include those packages that
are not dependencies of others already listed in order to reduce the list size. Finally the recipe will be wrote
in a file named `AppImageBuilder.yml`.


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

.. _ones here: https://hub.docker.com/r/appimagecrafters/tests-env

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

