.. _intro-install:

""""""""""""""""""
Installation guide
""""""""""""""""""

The project is built using Python 3 and uses various command-line applications to
fulfill its goal. Depending on the host system and the recipe the packages providing
such applications may vary.

-----------------------
Installing dependencies
-----------------------

Debian/Ubuntu
-------------

.. code-block:: shell

    sudo apt install -y python3-pip python3-setuptools patchelf desktop-file-utils libgdk-pixbuf2.0-dev fakeroot strace

    # Install appimagetool AppImage
    sudo wget https://github.com/AppImage/AppImageKit/releases/download/continuous/appimagetool-x86_64.AppImage -O /usr/local/bin/appimagetool
    sudo chmod +x /usr/local/bin/appimagetool

Archlinux
---------

.. code-block:: shell

    sudo pacman -Sy python-pip python-setuptools binutils patchelf desktop-file-utils gdk-pixbuf2 wget fakeroot strace

    # Install appimagetool AppImage
    sudo wget https://github.com/AppImage/AppImageKit/releases/download/continuous/appimagetool-x86_64.AppImage -O /usr/local/bin/appimagetool
    sudo chmod +x /usr/local/bin/appimagetool


---------------------------
Installing appimage-builder
---------------------------

Installing latest tagged release:

.. code-block:: shell

    sudo pip3 install appimage-builder

Installing development version:

.. code-block:: shell

    sudo pip3 install git+https://github.com/AppImageCrafters/appimage-builder.git

------
Docker
------

There is a docker image with appimage-builder ready to be used at hub.docker.com.

Use the following command to get it:

`docker pull appimagecrafters/appimage-builder:latest`


Install appimagetool
--------------------


There is an issue in the AppImage runtime format that prevents it proper execution inside docker containers.
Therefore we must use the following workaround to make `appimagetool` work properly.

.. code-block:: shell

    # Install appimagetool AppImage
    sudo wget https://github.com/AppImage/AppImageKit/releases/download/continuous/appimagetool-x86_64.AppImage -O /opt/appimagetool

    # workaround AppImage issues with Docker
    cd /opt/; sudo chmod +x appimagetool; sed -i 's|AI\x02|\x00\x00\x00|' appimagetool; sudo ./appimagetool --appimage-extract
    sudo mv /opt/squashfs-root /opt/appimagetool.AppDir
    sudo ln -s /opt/appimagetool.AppDir/AppRun /usr/local/bin/appimagetool
