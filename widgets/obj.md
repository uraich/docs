```eval_rst
.. include:: /header.rst 
:github_url: |github_link_base|/widgets/obj.md
```
# Base object (lv.obj)

## Overview

The 'Base Object' implements the basic properties of widgets on a screen, such as:
- coordinates
- parent object
- children
- main style
- attributes like *Click enable*, *Drag enable*, etc.

In object-oriented thinking, it is the base class from which all other objects in LVGL are inherited. This, among another things, helps reduce code duplication.

The functions and functionalities of Base object can be used with other widgets too. For example:\
`scr=lv.obj()`\
`lv.scr_load(scr)`
`slider=lv.slider(lv.scr_act(),None)` (lv.scr_act() is the currently active screen)\
`slider.set_width(100)`

The Base object can be directly used as a simple widgets. It nothing else then a rectangle.

### Coordinates

#### Size
The object size can be modified on individual axes with `obj.set_width(new_width)` and `obj.set_height(new_height)`, or both axes can be modified at the same time with `obj.set_size(new_width, new_height)`. `obj` is previously defined e.g. with `obj = lv.obj(lv.scr_act(),None)`

Styles can add [Margin](/overview/style/#properties) to the objects. Margin tells that "I want this space around me". 
To set width or height reduced by the margin `lv.obj.set_width_margin(new_width)` or `lv.obj.set_height_margin(new_height)`. 
In more exact way: `new_width = left_margin + object_width + right_margin`.

To get the width or height which includes the margins use `lv.obj_get_width/height_margin(obj)`.


Styles can add [Padding](/overview/style/#properties) to the object as well. Padding means "I don't want my children too close to my sides, so keep this space".
To set width or height reduced by the padding `obj.set_width_fit(new_width)` or `obj.set_height_fit(new_height)`.  
In a more exact way: `new_width = left_pad + object_width + right_pad`
To get the width or height which is REDUCED by padding use `obj.get_width/height_fit()`. It can be considered the "useful size of the object".

Margin and padding gets important when [Layout](/widget/cont#layout) or [Auto-fit](/wisgets/cont#fit) is used by other widgets.

#### Position
You can set the x and y coordinates relative to the parent with `obj.set_x(new_x)` and `obj.set_y(new_y)`, or both at the same time with `obj.set_pos(new_x, new_y)`.

#### Alignment
You can align the object to another with `obj.align(obj_ref, lv.ALIGN_..., x_ofs, y_ofs)`.
- `obj` is the object to align.
- `obj_ref` is a reference object. `obj` will be aligned to it. If `obj_ref = NULL`, then the parent of `obj` will be used.
- The third argument is the *type* of alignment. These are the possible options:
![](/misc/align.png "Alignment types in LVGL")

  The alignment types build like `lv.ALIGN.OUT_TOP_MID`.
- The last two arguments allow you to shift the object by a specified number of pixels after aligning it.

For example, to align a text below an image: `text.align(image, lv.ALIGN.OUT_BOTTOM_MID, 0, 10)`.
Or to align a text in the middle of its parent: `text.align(None, lv.ALIGN.CENTER, 0, 0)`.


`obj.align_origo` works similarly to `obj.align` but it aligns the center of the object.

For example, `btn.align_origo(image, lv.ALIGN.OUT_BOTTOM_MID, 0, 0)` will align the center of the button the bottom of the image (btn was again defined with `btn=lv.btn(lv.scr_act(),None)` before).

The parameters of the alignment will be saved in the object if `LV.USE_OBJ_REALIGN` is enabled in *lv_conf.h*. You can then realign the objects simply by calling `obj.realign()`. 
It's equivalent to calling `obj.align` again with the same parameters.

If the alignment happened with `obj.align_origo`, then it will be used when the object is realigned.

The `obj.align_x/y` and `obj.align_origo_x/y` function can be used t align only on one axis.

If `obj.set_auto_realign(True)` is used the object will be realigned automatically, if its size changes in `obj.set_width/height/size()` functions. 
It's very useful when size animations are applied to the object and the original position needs to be kept.


**Note that the coordinates of screens can't be changed. Attempting to use these functions on screens will result in undefined behavior.**

### Parents and children
You can set a new parent for an object with `obj.set_parent(new_parent)`. To get the current parent, use `obj.get_parent()`.

To get the children of an object, use `obj.get_child(child_prev)` (from last to first) or `obj.get_child_back(child_prev)` (from first to last).
To get the first child, pass `None` as the second parameter and use the return value to iterate through the children. The function will return `None` if there are no more children. For example:

```c
child = obj.get_child(parent, NULL);
while(child) {
    /*Do something with "child" */
    child = obj.get_child(parent, child);
}
```

`obj.count_children()` tells the number of children on an object. `obj.count_children_recursive()` also tells the number of children but counts children of children recursively.

### Screens
When you have created a screen like `screen = lv.obj(None, None)`, you can load it with `lv.scr_load(screen)`. The `lv.scr_act()` function gives you a pointer to the current screen.

If you have more display then it's important to know that these functions operate on the lastly created or the explicitly selected (with `lv_disp_set_default`) display.

To get an object's screen use the `obj.get_screen()` function.

### Layers
There are two  automatically generated layers:
- top layer
- system layer

They are independent of the screens and they will be shown on every screen. The *top layer* is above every object on the screen and the *system layer* is above the *top layer* too.
You can add any pop-up windows to the *top layer* freely. But, the *system layer* is restricted to system-level things (e.g. mouse cursor will be placed here in `lv.indev.set_cursor()`).

The `lv.layer_top()` and `lv.layer_sys()` functions gives a pointer to the top or system layer.

You can bring an object to the foreground or send it to the background with `obj.move_foreground()` and `obj.move_background()`.

Read the [Layer overview](/overview/layer) section to learn more about layers.

### Events

To set an event callback for an object, use `obj.set_event_cb(event_cb)`,

To manually send an event to an object, use `lv.event_send(obj, lv.EVENT_..., data)`

Read the [Event overview](/overview/event) to learn more about the events.


## Parts

The widgets can have multiple parts. For example a [Button](/widgets/btn) has only a main part but a [Slider](/widgets/slider) is built from a background, an indicator and a knob. 

The name of the parts is constructed like `lv. + <type> .PART. <NAME>`. For example `lv.btn_PART.MAIN` or `lv.slider.PART.KNOB`. The parts are usually used when styles are add to the objects. 
Using parts different styles can be assigned to the different parts of the objects. 

To learn more about the parts read the related section of the [Style overview](/overview/style#parts).


### States
The object can be in a combinations of the following states:
- **lv.STATE.DEFAULT**  Normal, released
- **lv.STATE.CHECKED** Toggled or checked
- **lv.STATE.FOCUSED** Focused via keypad or encoder or clicked via touchpad/mouse 
- **lv.STATE.EDITED**  Edit by an encoder
- **lv.STATE.HOVERED** Hovered by mouse (not supported now)
- **lv.STATE.PRESSED** Pressed
- **lv.STATE.DISABLED** Disabled or inactive

The states are usually automatically changed by the library as the user presses, releases, focuses etc an object. 
However, the states can be changed manually too. To completely overwrite the current state use `lv_obj_set_state(obj, part, LV_STATE...)`. 
To set or clear given state (but leave to other states untouched) use `lv_obj_add/clear_state(obj, part, LV_STATE_...)`
In both cases ORed state values can be used as well. E.g. `obj.set_state(part, lv.STATE.PRESSED | lv.PRESSED_CHECKED)`.

To learn more about the states read the related section of the [Style overview](/overview/style#states).

### Style
Be sure to read the [Style overview](/overview/style) first.

To add a style to an object use `obj.add_style(part, new_style)` function. The Base object use all the rectangle-like style properties. 

To remove all styles from an object use `obj.reset_style_list(part)`

If you modify a style, which is already used by objects, in order to refresh the affected objects you can use either `obj.refresh_style()` on each object using it or 
to notify all objects with a given style use `obj.report_style_mod(style)`. If the parameter of `obj.report_style_mod` is `None`, all objects will be notified.

### Attributes
There are some attributes which can be enabled/disabled by `obj.set_...(True/False)`:

- **hidden** -  Hide the object. It will not be drawn and will be considered by input devices as if it doesn't exist., Its children will be hidden too.
- **click** -  Allows you to click the object via input devices. If disabled, then click events are passed to the object behind this one. (E.g. [Labels](/widgets/label) are not clickable by default)
- **top** -  If enabled then when this object or any of its children is clicked then this object comes to the foreground.
- **drag** - Enable dragging (moving by an input device)
- **drag_dir** - Enable dragging only in specific directions. Can be `LV_DRAG_DIR_HOR/VER/ALL`.
- **drag_throw** - Enable "throwing" with dragging as if the object would have momentum
- **drag_parent** - If enabled then the object's parent will be moved during dragging. It will look like as if the parent is dragged. Checked recursively, so can propagate to grandparents too.
- **parent_event** - Propagate the events to the parents too. Checked recursively, so can propagate to grandparents too.
- **opa_scale_enable** - Enable opacity scaling. See the [#opa-scale](Opa scale) section.

### Protect
There are some specific actions which happen automatically in the library.
To prevent one or more that kind of actions, you can protect the object against them. The following protections exists:
- **lv.PROTECT.NONE** No protection
- **lv.PROTECT.POS**  Prevent automatic positioning (e.g.  Layout in [Containers](/widgets/cont))
- **lv.PROTECT.FOLLOW** Prevent the object be followed (make a "line break") in automatic ordering (e.g. Layout in [Containers](/widgets/cont))
- **lv.PROTECT.PARENT** Prevent automatic parent change. (e.g. [Page](/widgets/page) moves the children created on the background to the scrollable)
- **lv.PROTECT.PRESS_LOST** Prevent losing press when the press is slid out of the objects. (E.g. a [Button](/widgets/btn) can be released out of it if it was being pressed)
- **lv.PROTECT.CLICK_FOCUS** Prevent automatically focusing the object if it's in a *Group* and click focus is enabled.
- **lv.PROTECT.CHILD_CHG** Disable the child change signal. Used internally by the library

The `obj.add/clear_protect(lv.PROTECT_...)` sets/clears the protection. You can use *'OR'ed* values of protection types too.

### Groups

Once, an object is added to *group* with `lv.group_add_obj(group, obj)` the object's current group can be get with `obj.get_group()`.

`obj.is_focused(obj)` tells if the object is currently focused on its group or not. If the object is not added to a group, `False` will be returned.

Read the [Input devices overview](/overview/indev) to learn more about the *Groups*.

### Extended click area
By default, the objects can be clicked only on their coordinates, however, this area can be extended with `lv_obj_set_ext_click_area(obj, left, right, top, bottom)`.
`left/right/top/bottom` describes how far the clickable area should extend past the default in each direction.

This feature needs to enabled in *lv_conf.h* with `LV_USE_EXT_CLICK_AREA`. The possible values are:
- **lv.EXT_CLICK_AREA_FULL** store all 4 coordinates as `lv_coord_t`
- **lv.EXT_CLICK_AREA_TINY** store only horizontal and vertical coordinates (use the greater value of left/right and top/bottom) as `uint8_t`
- **lv.EXT_CLICK_AREA_OFF** Disable this feature

## Events
Only the [Generic events](../overview/event.html#generic-events) are sent by the object type.

Learn more about [Events](/overview/event).

## Keys
No *Keys* are processed by the object type.

Learn more about [Keys](/overview/indev).


## Example

```eval_rst

.. include:: /lv_examples/src/lv_ex_widgets/lv_ex_obj/index.rst

```

