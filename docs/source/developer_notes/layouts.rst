Layouts
=======

PlotArea
--------

This is the main base class within layouts. A ``Subplot`` and ``Dock`` are areas within a ``Figure``.
``Subplot`` and ``Dock`` inherit from ``PlotArea``.

``PlotArea`` has the following key properties that allow it to be a "plot area" that can be used to view graphical objects:

* scene - instance of ``pygfx.Scene``
* canvas - instance of ``WgpuCanvas``
* renderer - instance of ``pygfx.WgpuRenderer``
* viewport - instance of ``pygfx.Viewport``
* camera - instance of ``pygfx.PerspectiveCamera``, we always just use ``PerspectiveCamera`` and just set ``camera.fov = 0`` for orthographic projections
* controller - instance of ``pygfx.Controller``

If these concepts are unfamiliar to you we recommend learning about how rendering engines work, the pygfx guide
is a great place to start! https://docs.pygfx.org/stable/guide.html

Here is also a short video that goes through the basic concepts: https://www.youtube.com/watch?v=cvcAjgMUPUA

Abstract method that must be implemented in subclasses:

* get_rect - must return [x, y, width, height] that defines the viewport rect for this ``PlotArea``

Properties specifically used by subplots in a Figure:

* parent - A parent if relevant, used by individual ``Subplots`` in ``Figure``, and by ``Dock`` which are "docked" subplots at the edges of a subplot.
* position - if a subplot within a ``Figure``, it is the position of this subplot within the ``Figure``

Other important properties:

* graphics - a tuple of weakref proxies to all ``Graphics`` within this ``PlotArea``, users are only given weakref proxies to ``Graphic`` objects, all ``Graphic`` objects are stored in a private global dict.
* selectors - a tuple of weakref proxies to all selectors within this ``PlotArea``
* legend - a tuple of weakref proxies to all legend graphics within this ``PlotArea``
* name - plot areas are allowed to have names that the user can use for their convenience

Important methods:

* add_graphic - add a ``Graphic`` to the ``PlotArea``, append to the end of the ``PlotArea._graphics`` list
* insert_graphic - insert a ``Graphic`` to the ``PlotArea``, insert to a specific position of the ``PlotArea._graphics`` list
* remove_graphic - remove a graphic from the ``Scene``, **does not delete it**
* delete_graphic - delete a graphic from the ``PlotArea``, performs garbage collection
* clear - deletes all graphics from the ``PlotArea``
* center_graphic - center camera w.r.t. a ``Graphic``
* center_scene - center camera w.r.t. entire ``Scene``
* auto_scale - Auto-scale the camera w.r.t to the ``Scene``

In addition, ``PlotArea`` supports ``__getitem__``, so you can do: ``plot_area["graphic_name"]`` to retrieve a ``Graphic`` by
name.

You can also check if a ``PlotArea`` has certain graphics, ex: ``"some_image_name" in plot_area``, or ``graphic_instance in plot_area``

Subplot
-------

This class inherits from ``PlotArea`` and ``GraphicMethodsMixin``.

``GraphicMethodsMixin`` is a simple class that just has all the ``add_<graphic>`` methods. It is autogenerated by a utility script like this:

.. code-block:: bash

    python scripts/generate_add_methods.py

Each ``add_<graphic>`` method basically creates an instance of ``Graphic``, adds it to the ``Subplot``, and returns the ``Graphic``.

``Subplot`` has one property that is not in ``PlotArea``:

* docks: a ``dict`` of ``PlotAreas`` which are located at the "top", "right", "left", and "bottom" edges of a ``Subplot``.

By default their size is ``0``. They are useful for putting things like histogram LUT tools.

The key method in ``Subplot`` is an implementation of ``get_rect`` that returns the viewport rect for this subplot.

Figure
------

Now that we have understood ``PlotArea`` and ``Subplot`` we need a way for the user to create them!

A ``Figure`` contains a grid of subplot and has methods such as ``show()`` to output the figure.
``Figure.__init__`` basically does a lot of parsing of user arguments to determine how to create
the subplots. All subplots within a ``Figure`` share the same canvas and use different viewports to create the subplots.
