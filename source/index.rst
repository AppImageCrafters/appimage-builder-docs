.. _index:

""""""""""""""""""""""""""""""""""""""""
appimage-builder |version| documentation
""""""""""""""""""""""""""""""""""""""""

``appimage-builder`` is a novel tool for creating AppImages. It uses the
system package manager to resolve the application dependencies and
creates a complete bundle. It can be used to pack almost any kind of
applications including those made using: ``C/C++``, ``Python``, and ``Java``.

For more information about the AppImage packaging format visit: https://appimage.org/

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
.. _#appimage IRC channel: irc://irc.freenode.net/appimage
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
   intro/examples

:doc:`intro/overview`
    Understand what `appimage-builder` is and how it can help you.

:doc:`intro/install`
    Get `appimage-builder` installed on your computer.

:doc:`intro/tutorial`
    Write your first `appimage-builder` recipe.

:doc:`intro/examples`
    Learn more by playing with a pre-made `appimage-builder` project.

===============
Advanced topics
===============

.. toctree::
   :caption: Advanced topics
   :hidden:

   advanced/updates

   advanced/signing

   advanced/testing

   advanced/troubleshooting


:doc:`advanced/updates`
    Add delta updates support to your AppImage

:doc:`advanced/signing`
    Sign your binaries to protect your users

:doc:`advanced/testing`
    Test in different systems

:doc:`advanced/troubleshooting`
    Find and fix issues

===============
Hosted Services
===============

.. toctree::
   :caption: Hosted Services
   :hidden:

   hosted-services/gitlab-ci

:doc:`hosted-services/gitlab-ci`
    Learn how to produce AppImages on Gitlab instances

====================
Recipe specification
====================

.. toctree::
   :caption: Recipe specification
   :hidden:

   recipe/version_1

:doc:`recipe/version_1`
    Specification of the `appimage-builder` version 1 recipe format.
