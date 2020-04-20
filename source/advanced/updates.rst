.. _advanced-updates:

""""""""""""""""
AppImage Updates
""""""""""""""""


===========================
The AppImage Update process
===========================

An AppImage can be updated by just downloading the latest whole binary from the author or using delta updates. The
second method is much more efficient as it only downloads the parts that changed, therefore, it's faster. Also the
`appimageupdatetool`_ checks the file signature (if present) to ensure that the downloaded file is legit.


The `appimageupdatetool`_ uses the the `zsync method`_ to do the delta update. Therefore a ``.zsync`` file is required
for the updates to work. The file url is embed into the AppImage, this is known as the update information. There are
several ways of specifying this url:

.. _appimageupdatetool: https://github.com/AppImage/AppImageUpdate/releases
.. _zsync method: https://en.wikipedia.org/wiki/Rsync

Update information
==================

According to `the specification`_ an AppImage **MAY** have update information embedded for exactly one transport
mechanism. Currently three transport mechanisms are available, but only one can be used for each given AppImage.
Below we describe then in detail.

.. _the specification: https://github.com/AppImage/AppImageSpec/blob/master/draft.md#update-information

zsync
-----

The zsync transport requires a HTTP server that can handle HTTP range requests. Its update information is in the form:
``zsync|<zsync file URL>``, by example ``zsync|https://server.domain/path/Application-latest-x86_64.AppImage.zsync``



GitHub Releases
---------------

The GitHub Releases transport extends the zsync transport in that it uses version information from
`GitHub Releases`_. Its update information is in the form:

``gh-releases-zsync|<name space>|<project>|latest|<zsync file name>``, by example
``gh-releases-zsync|probono|AppImages|latest|Subsurface-*x86_64.AppImage.zsync``

.. _GitHub Releases: https://help.github.com/articles/about-releases/


bintray-zsync
-------------

The bintray-zsync transport extends the zsync transport in that it uses version information from
`Bintray`_. Its update information is in the form:
``bintray-zsync|<username>|<repository>|<package name>|<zsync file path>``, by example
``bintray-zsync|probono|AppImages|Subsurface|Subsurface-_latestVersion-x86_64.AppImage.zsync``

.. _Bintray: https://bintray.com/


===================================
Setting AppImage update information
===================================

Before setting the update information make sure that ``zsync`` is installed in the build system. Then just add the update
information line according to the selected method in ``AppImage >> update-information`` like this

.. code-block:: yaml

    AppImage:
      update-information: gh-releases-zsync|probono|AppImages|latest|Subsurface-*x86_64.AppImage.zsync
      arch: aarch64

Once the build finish there will be a ``.zsync`` file next to the AppImage one. You should publish both of then
according to the chosen update protocol.