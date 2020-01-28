.. _doc_size_and_anchors:

Using Control nodes
===================

If a game was always going to be run on the same device and at the same
resolution, positioning GUI elements would be a simple matter of setting the
position and size of each one of them. Unfortunately, that is rarely the case,
as computer monitors, tablets, portable consoles and mobile phones have
different resolutions and aspect ratios.

To build flexible interfaces, Godot provides you with a set of :ref:`Control
<class_Control>` nodes. Those Control nodes can adapt automatically in position
and size, and thus allow you create interfaces that are independent from your
screen resolution. Within a tree, a Control node always position itself
according to its parent's position (exactly like Node2Ds), but also according
to its parent's size. A Control node which has no Control parent uses the
viewport as a reference instead.

Building interfaces can be done through two different workflows: using the
anchors and margins system, or using containers nodes. As it uses a hierarchy
of nodes to position the nodes, the container workflow is likely the most
straightfoward approach to position Control nodes. However, the anchors and
margin workflow may require less nodes and provide a little bit more
flexibility in some cases. Note that both systems can be mixed together if
needed.

Positioning Control nodes using anchors and margins
---------------------------------------------------

Godot uses the anchors and margins system to position and resize a
:ref:`Control <class_Control>` node, **provided the node is not a child of a node
extending the :ref:`Container <class_Container>` class.**

To position itself according to its parent, a Control nodes use two types of
properties: the anchors and the margins. Those properties exist for each of the
4 sides of a Control node, i.e. left, right, top and bottom.

On the one hand, anchors are set using a proportion of the parent's size. They
allow setting the position of each side of a Control node **relatively** to its
parent's corresponding side. On the other hand, the margins are **absolute**
offset values in pixels. Given a Control's side (left for example), the child
Control's left side's position is calculated by adding the left anchor's value,
multiplied by the parent's width, to the left margin (absolute) value~:

```
Control side's position = Anchor * (parent's width or height) + Margin
```
.. note:: You can visualize the anchors position and the margins values by
          activating the ``View > Show`` helpers entry in the 2D editor.

In general, we position a child node inside it's parent boundaries,
consequently, the anchors values are usually set between 0 and 1 (nonetheless,
they can be set outside this range if needed). Margins have wider value range
though, since they correspond to a number of pixels.

To illustrate Control positioning works, let us consider the default anchors
values, i.e. 0 for each side. For horizontal values (left and right), this
means the anchors are positioned at the left of the parent Control, while the
vertical ones (top and bottom) are positioned at the top. In this default
situation, all margins values represent a distance in pixels relative to the
top-left corner of the parent Control or (in case there is no parent Control)
the viewport~:

# Image

When horizontal (left,right) and/or vertical (top,bottom) anchors are
changed to 1, the margin values become relative to the bottom-right
corner of the parent Control or viewport~:

# Image

In this example the left and right anchors are set to 0.5, while the top and
bottom ones are set to 0. In this situation margin values are relative to the
parent's top center~:

# Image

However, the anchors and margins system becomes more interesting when the
anchors values on a same axis (horizontal or vertical) are not the same. For
example, setting the left anchor to 0 and the right anchors to 1, means that
the left margin of a Control is relative to the left side of its parent, while
the right margin is relative to its right side. This means that the child
Control will grow horizontally as its parents does~:

# Image

All anchors values presented above are available as default presets in the
Layout menu. By default, those menu entries also move an resize the node to the
corresponding preset, but you can also choose to keep the Control in place by
using the "Anchors only" sub-menu.

Usually in the editor, moving a node modify its margins, however, you might
want to change the anchors instead. This is the case for example when you want
to keep the parent/child aspect ratio while moving the node. This is possible
by enabling the "Anchor mode" in the editor~:

.. note:: The "Keep size" preset positions the anchors at the boundaries of the
          Control node, and sets its margins to 0. This allows keeping the
          parent-child aspect ratio. When you use this preset, the "Anchor mode"
          of the editor is automatically actived. For all other presets, the
          "Anchor mode" is automatically disabled.

The Layout menu provides presets for the most common behaviors for positioning
Control nodes, however custom values for anchors and margins may be used to
obtain less common behaviors. For example, a Control may be set to scale up
according to its parent, but only up to the third of its parents size, by
setting a right anchor to `0.33`.

The flexibility the anchors and margins workflow provides allows almost any
possible layout with very few nodes. It is particularly interesting when you
have to position nodes in specific positions, and when there are no strong
relationship (alignment, same size...) between the Control nodes to position.
A typical use case could be placing the elements of an HUD, as the GUI elements
would be scattered over the screen~:

# Image

For GUI containing more elements that need to be precisely aligned, or elements
that need to be added dynamically, the anchors and margins workflow would
requires a lot of set up or scripting to obtain the desired results. In such
situation, the Container-based is a better fit.

Positioning Control nodes using Containers
------------------------------------------

Another way to position Control nodes is to use :ref:`Container
<class_Container>` nodes. As a Container automatically position its children,
it avoids the need to set the anchors and margins values on each of those
children. The Container class exposes a function (automatically called by the
engine), which position its children. The built-in classes extending the
:ref:`Container <class_Container>` class provide a wide range of positioning
behavior, however, if the built-in classes are not enough for you needs, you
can also implement your own Container. (#TODO: Add reference) Note that a
Container overrides it's children anchors and margins properties, so you should
not modify them (this might lead to unwanted behaviors).

Within a Container, the positionning of a Control node might depend on
properties exposed by the Container itself, on properties exposed by the child,
or both. In this regard, the Control class provides a set of properties oftenly
used by built-in Containers: the size flags, the grow directions and the
minimum size. (#TODO: Add reference)

To illustrate how a Container works, we can use the very common
:ref:`HBoxContainer <class_HBoxContainer>`. This node aligns its children
horizontally. As we can see with this container, adding a several childs to
this HBoxContainer aligns the children nodes automatically:

# Image

As we can see, the HBoxContainer provides a custom constant called
"separation", that allows increasing the space between each child node:

# Image

To obtain a per-child specific behavior, each child :ref:`Control
<class_Control>` exposes a set of size flags. In :ref:`HBoxContainer
<class_HBoxContainer>,`, the vertical `fill` flag makes the child node take the
available vertical space in the container (it is on by default):

# Image

When the `fill` flag is disabled, the `shrink center` and `shrink end` align
the node vertically, respectively at the center and at the bottom of the
container:

# Image

The horizontal `expand` flag makes a node take as much horizontal space as
there is left in the container. If several nodes have this expand flag, the
remaining space is shared between those nodes:

# Image

Finally, note that the child control nodes can never be shrunk more than their
computed minimum size, which usually depends on the Control's content. For a
:ref:`Button <class_Button>`, this minimum size is computed according to its
text and stylebox, while for most containers, the computed minimum size depends
on the minimum size of all of its children. The way this minimum size is
computed is usually straighforward, as it is meant to avoid unexpected glitches
and overlapping elements in the GUI.

If the computed minimum size does not fit you needs, you can still override it
using the control's `min_size` property. However, this size can only be
increased, and not reduced. If you need smaller GUI elements, you should likely
create a stylebox with smaller margins, so that the GUI can be made more
compact. Be careful with small GUI elements though, not everyone has a perfect
eyesight.

The example we went through with the :ref:`HBoxContainer <class_HBoxContainer>`
class is quite representative about how Containers handle their children
positioning: they mostly use a combination of their properties, their custom
constants and their children size flags. The impact of those individual
properties depends on the container used though, please see the reference
#TODO: add reference # for more information.

######## 
