.. _intro-install:

====================
Get appimage-builder
====================

""""""""
AppImage
""""""""

``appimage-builder`` is available as ready to use AppImage for amd64 systems. Just download it from the `releases`_ and
make it executable. In case you want it to be available in your cli as a command you can move it to ``/usr/local/bin`` or other location
in your ``PATH``.

.. _releases: https://github.com/AppImageCrafters/appimage-builder/releases

.. code-block:: shell

    wget -O appimage-builder-x86_64.AppImage https://github.com/AppImageCrafters/appimage-builder/releases/download/v1.1.0/appimage-builder-1.1.0-x86_64.AppImage
    chmod +x appimage-builder-x86_64.AppImage

    # install (optional)
    sudo mv appimage-builder-x86_64.AppImage /usr/local/bin/appimage-builder


""""""""""""
Docker Image
""""""""""""

There is a `docker image`_ with appimage-builder ready to be used at hub.docker.com.

.. _docker image: https://hub.docker.com/r/appimagecrafters/appimage-builder

.. code-block:: shell

    docker pull appimagecrafters/appimage-builder:latest


.. note::
    Testing AppImages is not supported on this format. Always use `--skip-test`.


"""""""""""""""""""
Manual Installation
"""""""""""""""""""

The project is built using Python 3 and uses various command-line applications to fulfill its goal.
Depending on the host system and the recipe the packages providing such applications may vary.

-----------------------
Installing dependencies
-----------------------

Debian/Ubuntu
-------------

.. code-block:: shell

    sudo apt install -y binutils coreutils desktop-file-utils fakeroot fuse libgdk-pixbuf2.0-dev patchelf python3-pip python3-setuptools squashfs-tools strace util-linux zsync

    # Install appimagetool AppImage (only for appimage-buidler < v1.0.3)
    sudo wget https://github.com/AppImage/AppImageKit/releases/download/continuous/appimagetool-x86_64.AppImage -O /usr/local/bin/appimagetool
    sudo chmod +x /usr/local/bin/appimagetool

Archlinux
---------

.. code-block:: shell

    sudo pacman -Sy binutils desktop-file-utils fakeroot gdk-pixbuf2 patchelf python-pip python-setuptools squashfs-tools strace wget zsync

    # Install appimagetool AppImage (only for appimage-buidler < v1.0.3)
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


Install appimagetool
--------------------

.. note:: 
    Only for appimage-buidler < v1.0.3
    
There is an issue in the AppImage runtime format that prevents it proper execution inside docker containers.
Therefore we must use the following workaround to make `appimagetool` work properly.

.. code-block:: shell

    # Install appimagetool AppImage 
    sudo wget https://github.com/AppImage/AppImageKit/releases/download/continuous/appimagetool-x86_64.AppImage -O /opt/appimagetool

    # workaround AppImage issues with Docker
    cd /opt/; sudo chmod +x appimagetool; sed -i 's|AI\x02|\x00\x00\x00|' appimagetool; sudo ./appimagetool --appimage-extract
    sudo mv /opt/squashfs-root /opt/appimagetool.AppDir
    sudo ln -s /opt/appimagetool.AppDir/AppRun /usr/local/bin/appimagetool
