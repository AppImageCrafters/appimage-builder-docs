.. _intro-install:

"""""""""""""""""""""""""""""""""""
appimage-builder installation guide
"""""""""""""""""""""""""""""""""""

The project is built using Python 3 and uses various command-line applications to
fulfill its goal. Depending on the host system and the recipe the packages providing
such applications may vary.

**NOTICE**: The only package manager fully supported currently is **APT**. `appimage-builder` will
not work on systems without it.

Install on Debian/Ubuntu
------------------------
Installing dependencies

.. code-block:: shell

    sudo apt install -y python3-pip python3-setuptools patchelf desktop-file-utils libgdk-pixbuf2.0-dev fakeroot

    # Install appimagetool AppImage
    sudo wget https://github.com/AppImage/AppImageKit/releases/download/continuous/appimagetool-x86_64.AppImage -O /usr/local/bin/appimagetool
    sudo chmod +x /usr/local/bin/appimagetool

Installing latest tagged release:

.. code-block:: shell

    sudo pip3 install appimage-builder

Installing development version:

.. code-block:: shell

    sudo pip3 install git+https://github.com/AppImageCrafters/appimage-builder.git


Install in a Docker container
-----------------------------

There is an issue in the AppImage runtime format that prevents it proper execution inside docker containers.
Therefore we must use the following workaround to make `appimagetool` work properly:

.. code-block:: shell

    # Install appimagetool AppImage
    sudo wget https://github.com/AppImage/AppImageKit/releases/download/continuous/appimagetool-x86_64.AppImage -O /opt/appimagetool
    sudo chmod +x /opt/appimagetool
    cd /opt/; sudo /opt/appimagetool --appimage-extract
    sudo mv /opt/squashfs-root /opt/appimagetool.AppDir
    sudo ln -s /opt/appimagetool.AppDir/AppRun /usr/local/bin/appimagetool
