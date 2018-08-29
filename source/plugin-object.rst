The Plugin Object and Entrypoints
=================================

QIIME 2 plugins are separate software packages that define domain-specific functionality that can be accessed via the QIIME 2 framework. Creating a QIIME 2 plugin allows you to provide microbiome analysis functionality to QIIME 2 users. A plugin can be a standalone software project, or you can make a few small additions to your existing software project to make it a QIIME 2 plugin. Creating a single QIIME 2 plugin will make your functionality accessible through any QIIME 2 interface, including the QIIME 2 Studio (GUI), q2cli, and the Artifact API. This document will describe how to `register` a plugin, allowing this plugin to interact with the QIIME 2 framework.

Overview
--------
There are several high-level requirements for creating a QIIME 2 plugin:

1. Plugins must define one or more Python 3 functions accessible through QIIME 2. 
2. Plugins must be Python 3 packages that can be installed with ``setuptools``.
3. Plugins must instantiate a ``qiime2.plugin.Plugin`` object and define some information including the name of the plugin and its URL. In the plugin package’s ``setup.py`` file, this instance will be defined as an `entry point`.
4. Plugins must then register their functions as QIIME 2 :doc:`Actions <actions/index>`, which are accessible to users through any of the QIIME 2 interfaces.
5. Optionally, plugins should be distributed through `Anaconda`_. This will simplify installation for QIIME 2 users (since Anaconda is the supported mechanism for installing QIIME 2).

These steps are covered in detail below.

Writing a simple QIIME 2 plugin should be a straightforward process. For example, the `q2-emperor`_ plugin, which connects `Emperor`_ to QIIME 2, is written in a little over 100 lines of code (excluding unit tests and assets). It is a standalone plugin that defines how and which functionality in Emperor should be accessible through QIIME 2. Plugins will vary in their complexity. For example, a plugin that defines a lot of new functionality would likely be quite a bit bigger. `q2-diversity`_ is a good example of this. Unlike ``q2-emperor``, there is some specific functionality (and associated unit tests) defined in this project, and it depends on several other Python 3 compatible libraries.

Before starting to write a plugin, you should install QIIME 2 and some plugins to familiarize yourself with the system and to provide a means for testing your plugin.

Instantiating a plugin
----------------------

The next step is to instantiate a QIIME 2 ``Plugin`` object. This might look like the following:

.. code-block:: python

   from qiime2.plugin import Plugin
   import q2_diversity

   plugin = Plugin(
       name='diversity',
       version=q2_diversity.__version__,
       website='https://github.com/qiime2/q2-diversity',
       package='q2_diversity',
       description=('This QIIME 2 plugin supports metrics for calculating '
                    'and exploring community alpha and beta diversity through '
                    'statistics and visualizations in the context of sample '
                    'metadata.'),
       short_description='Plugin for exploring community diversity.',
   )


This will provide QIIME with essential information about your ``Plugin``.

The ``name`` parameter is the name that users will use to access your plugin from within different QIIME 2 interfaces. It should be a "command-line-friendly" name, so should not contain spaces or punctuation. (Avoiding uppercase characters and using dashes (``-``) instead of underscores (``_``) are preferable in the plugin ``name``, but not required).

``version`` should be the version number of your package (the same that is used in its ``setup.py``).

``website`` should be the page end users refer to for more information about your package.

``package`` should be the Python package name for your plugin.

``description`` should give a brief description of this plugin's functionality. This will be displayed when that plugin's help documentation is accessed via the QIIME 2 framework.

``short_description`` should give a very brief description of this plugin's functionality. This will be displayed when the QIIME 2 help documentation is accessed.

While not shown in the previous example, plugin developers can optionally provide the following parameters to ``qiime2.plugin.Plugin``:

* ``citations``: A list of bibtex-formatted citations. These are provided in a separate ``citations.bib`` file, loaded via the ``Citations`` API, and accessed by using their bibtex indices as keys. Citations can be listed during plugin or action registration, or both, but will usually only be listed for individual actions unless a single reference is appropriate for all actions in that plugin. ``q2-diversity`` has no such plugin-wide citation listed here, but a ``citations`` example is given for :doc:`method registration <actions/methods>`.

* ``user_support_text``: free text describing how users should get help with the plugin (e.g. issue tracker, StackOverflow tag, mailing list, etc.). If not provided, users are referred to the ``website`` for support. We encourage plugin developers to support their plugins on the QIIME 2 Forum, so you can include that URL as the ``user_support_text`` for your plugin. If you do that, you should get in the habit of monitoring the QIIME 2 Forum for technical support questions.

The ``Plugin`` object can live anywhere in your project, but by convention it will be in a file called ``plugin_setup.py``. For an example, see ``q2_diversity/plugin_setup.py``.


Defining your plugin object as an entry point
---------------------------------------------

Finally, you need to tell QIIME where to find your instantiated ``Plugin`` object. This is done by defining it as an ``entry_point`` in your project's ``setup.py`` file. In ``q2-diversity``, this is done as follows:

.. code-block:: python

   setup(
       ...
       entry_points={
           'qiime2.plugins': ['q2-diversity=q2_diversity.plugin_setup:plugin']
       }
   )

The relevant key in the ``entry_points`` dictionary will be ``'qiime2.plugins'``, and the value will be a single-element list containing a string formatted as ``<distribution-name>=<import-path>:<instance-name>``.
* ``<distribution-name>`` is the name of the Python package distribution (matching the value passed for ``name`` in this call to ``setup``)
* ``<import-path>`` is the import path for the ``Plugin`` instance you created above
* ``<instance-name>`` is the name for the ``Plugin`` instance you created above.

Registering actions
-------------------

Now that you have registered your plugin in QIIME 2, you need to register the plugin's ``Actions`` so that they can be recognized and used by the interface. Writing and registering different types of ``Actions`` is covered in the :doc:`Actions documentation <actions/index>`.

Using metadata in a plugin
--------------------------

See the :doc:`metadata documentation <metadata>` for information about :term:`primitive types <primitive type>` and registering metadata in actions.


.. _Anaconda: https://anaconda.org/
.. _q2-emperor: https://github.com/qiime2/q2-emperor
.. _Emperor: https://github.com/biocore/emperor
.. _q2-diversity: https://github.com/qiime2/q2-diversity
