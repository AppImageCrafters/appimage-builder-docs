.. _intro-tutorial:

""""""""
Tutorial
""""""""

In this page is explained how to build an AppImage for a simple Qt/Qml application. The tutorial is meant to be
performed in a Ubuntu (18.04 or newer) system where **appimage-builder** have been installed. Check the :ref:`intro-install`
for instructions. The application code can be found `here`_.

.. _here: https://www.opencode.net/azubieta/qt-appimage-template

=======================
Compile the Application
=======================

The first step in our process is to build the application binaries and deploy them into a folder named AppDir.
It's important to use '/usr' as install prefix as appimage-builder expects to find the application resources
such as the desktop entry and the icon in their standar paths relative to AppDir.

We will use the `DESTDIR`_ make variable to install the binaries using a root different than '/'.
Notice that while many build toolchains such as cmake support this variable it's not standard.

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

    # install the application to 'AppDir'
    make install DESTDIR=AppDir


=================
Recipe Generation
=================

Once the application binaries are deployed to an ``AppDir`` we proceed to run ``appimage-builder --generate``. This
will proceed to read the application executable from the desktop entry and launch it. The application will show up and
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

