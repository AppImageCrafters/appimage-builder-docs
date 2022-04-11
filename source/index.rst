.. _index:

""""""""""""""""""""""""""""""""""""""""
appimage-builder |version| documentation
""""""""""""""""""""""""""""""""""""""""

``appimage-builder`` is a novel tool for creating AppImages. It uses the
system package manager to resolve the application dependencies and
creates a complete bundle. It can be used to pack almost any kind of
applications including those made using: ``C/C++``, ``Python``, and ``Java``.

Featuring:
 - Real GNU/Linux packaging (no more distro packaging)
 - Simple recipes
 - Simple workflow
 - Backward and forward compatibility
 - One binary, many target systems.

For information about the AppImage packaging format visit: https://appimage.org/

.. _community:

============
Getting help
============

Having trouble? We'd like to help!

* Try the :doc:`FAQ <faq>` -- it's got answers to some common questions.
* Ask or search questions in `StackOverflow using the AppImage tag`_.
* Ask or search questions in the `AppImage subreddit`_.
* Ask a question in the `#appimage IRC channel`_.
* Report bugs with appimage-builder in our `issue tracker`_.

.. _AppImage subreddit: https://www.reddit.com/r/appimage/
.. _StackOverflow using the AppImage tag: https://stackoverflow.com/tags/AppImage
.. _#appimage IRC channel: irc://libera.chat/appimage
.. _issue tracker: https://github.com/AppImageCrafters/appimage-builder/issues

===========
First steps
===========

.. toctree::
   :caption: First steps
   :hidden:

   intro/overview
   intro/install
   intro/tutorial
   faq

:doc:`intro/overview`
    Understand what `appimage-builder` is and how it can help you.

:doc:`intro/install`
    Get `appimage-builder` installed on your computer.

:doc:`intro/tutorial`
    Write your first `appimage-builder` recipe.

.. toctree::
    :caption: Examples

    examples/bash
    examples/gnome
    examples/kde
    examples/multimedia
    examples/pyqt
    examples/gimp_path_mapping
    examples/flutter
    examples/gambas3

.. toctree::
    :caption: Advanced topics
    :hidden:

    advanced/updates
    advanced/signing
    advanced/testing
    advanced/troubleshooting
    advanced/full_bundle




.. toctree::
    :caption: Hosted Services
    :hidden:

    hosted-services/gitlab-ci
    hosted-services/github-actions


.. toctree::
    :caption: Reference
    :hidden:

    reference/version_1
