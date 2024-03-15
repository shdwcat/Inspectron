# INSPECTRON 1.0.0
A library for easily creating GameMaker debug views

## What is Inspectron?
Inspectron is a library designed to help you easily create debug views for objects in your game, and lets you easily customize what values get displayed using a simple [fluent API][fluent_api]. Inspectron calculates the best place to put the debug view, ensuring that it's sized appropriately and remains entirely on screen, and automatically closes the view if the target object is destroyed.

Here's an example of Inspectron working out-of-the-box in the Windy Woods template project:
![image](https://github.com/shdwcat/Inspectron/assets/15136382/cac60a04-29e6-4ff7-a7a3-bca99a0ebf98)

Inspectron can automatically generate a debug view for built-in variables on an instance, as you can see above, but it also allows you to easily customize what values get displayed for objects you create.

Here's an example of the custom inspection for my GameMaker UI library, [YUI](https://github.com/shdwcat/YUI):
![image](https://github.com/shdwcat/Inspectron/assets/15136382/d0f68545-37af-4a2c-915d-58fce4386d14)

## Quick Start
First, import the .yymps package from the [Releases page](https://github.com/shdwcat/Inspectron/releases) into your project. Once that's done, you have a few options for how to enable Inspectron in your project, but the fastest way to see what Inspectron can do is to enable the default mouse gesture, which is as easy as setting one config macro.

To start, open the InspectronConfig script asset in the project root and set `INSPECTRON_GESTURE_ENABLED` to true:
```gml
// whether to automatically call InspectronSetGesture with the gesture settings below
#macro INSPECTRON_GESTURE_ENABLED true
```

Once this is set, Inspectron will configure the default mouse gesture, Shift+Middle-Click, to open the Inspectron debug view for any instance under the mouse cursor. If `INSPECTRON_AUTO_INSPECT_ENABLED` is true (which it is by default), Inspectron will automatically display many of the common built-in variables for those instances (and allow you to edit some of them!). You can see what this looks like in that first Windy Woods screenshot above.

You can customize which button and which modifier key to use in the config macros below that line:
```gml
// mouse button and optional modifier key to trigger Inspectron if enabled above
#macro INSPECTRON_GESTURE_MOUSE_BUTTON mb_middle
#macro INSPECTRON_GESTURE_MODIFIER vk_shift // set to undefined for no modifier
```

## Customize Which Objects Get Inspected

The automatic gesture method will show the Inspectron view for *all* instances under the mouse cursor, but if your game is complex you may want to customize which objects actually get included in the view, especially if you want to differentiate between instances that draw in the game world vs instances that draw to the GUI layer.

To do that, you'll need an object event to trigger Inspectron, where you can customize what Inspectron does. For example, you can add a Global Middle Released event to a controller object in your game. Once you have that, you'll want to call `InspectronGo()` with some custom arguments. For example:
```gml
// in Global Middle Released event
InspectronGo(obj_character_base, obj_gui_base, "Characters and GUI");
```
* The first parameter tells Inspectron which kind of instances to look for in the game world, using room coordinates.
* The second (optional) parameter tells Inspectron which kind of instances to look for in the GUI layer.
  * You can pass `undefined` if you don't want Inspectron to check for GUI instances.
* The third (optional) parameter is simply a name for the debug view. You can set up different gestures for inspecting different kinds of objects, so the name can help you remember which one is being shown (and will be listed in the Views menu at the top of the GameMaker debug overlay).
  * If you don't pass a name, `"Inspectron"` will be used by default.
* There is a fourth (optional) parameter, which can be a function that returns a name to display for a given instance. For example, if your characters have a `name` variable, you can pass `function (instance) { return instance.name; }`, and that `name` will be displayed in the Inspectron debug view, and in the dropdown to choose which instance to inspect if multiple are overlapping (see screenshots above).
  * By default, Inspectron will show the numeric ID of the instance and the name of the GM object from the instance's `object_index` variable (as seen in the screenshots above).

## Customize the Inspectron View for Different Objects

The built-in variables that Inspectron can display automatically are convenient, but you'll likely want to make your custom variables on objects inspectable as well. Inspectron makes this very easy, by providing a [fluent API][fluent_api] to tell Inspectron what variables to display and how to display them. Inspectron will then take care of making the underlying `dbg_` function calls for you to display the Debug view.

You should define the Inspectron fields for an object at the **very end** of the Create event of your object. For example:
```gml
// in obj_player Create Event

...
// All the other code in your create event
...

Inspectron()
  .Section("player")
  .Watch(nameof(player_health), "Health")
  .Checkbox(nameof(invulnerable))
  .BuiltIn()
```

The above code will tell Inspectron to show:
* A [section][dbg_section] named "player".
* The current value of the `player_value` variable on the obj_player instance.
  * The second parameter tells Inspectron to use `"Health"` as the label, instead of re-using the variable name (`"player_health"`).
* A [checkbox][dbg_checkbox] to allow you to set the `invulnerable` variable on the obj_player instance to `true` or `false`.
* The standard set of built-in variables that Inspectron can display by default.
  * You can go look at the function to see how it's set up! It's just using the same functions you can use elsewhere.

The fluent API means you can keep calling Inspectron field functions in sequence until you've defined everything you want to display.

NOTE: the first parameter must be the *name* of the variable on the instance that you want to display, as a string. It's best practice to use the `nameof()` function provided by GML, as if you later rename that variable where it's defined, the `nameof()` function will report an error.

### Handling Object Parents (and Inherited Constructors)
Actually you don't really have to do anything special here. If you define an Inspectron on a parent object, and then call `Inspectron.Etc()` on a child object, Inspectron will automatically include the parent inspectron in the child object's definition, at the end. This works no matter how many object parent/child relationships you set up.

You may want to include a `.Section` or `.Header` in child objects to make clear about which variables belong to which object. The same is true for constructor functions that inherit another constructor.

### Supported Functions
#### Basic
* `.BuiltIn()` - Adds the default list of inspections for built-in instance variables, e.g. x, y, sprite_index, etc.
  * These are not included by default when defining a custom Inspectron on an object/struct, you have to call this function yourself if you want them!
* `.Section(label)` - Creates a [dbg_section][dbg_section] with the provided `label`
* `.Header(header)` - Creates a [dbg_text][dbg_text] with `header` formatted as `[ header ]` (useful for separating groups of variables inside a section)
* `.Label_(label)` - Creates a [dbg_text][dbg_text] with `label` as the text
* `.Button(label, on_click)` - Creates a [dbg_button][dbg_button] with `on_click` as the button's function
* `.Watch(field_name, [label])` - Creates a [dbg_watch][dbg_watch] for the provided field name, with an optional custom `label`
* `.Color(field_name, [label])` - Creates a [dbg_color][dbg_color] for the provided field name, with an optional custom `label`
* `.Checkbox(field_name, [label])` - Creates a [dbg_checkbox][dbg_checkbox] for the provided field name, with an optional custom `label`
* `.TextInput(field_name, [label])` - Creates a [dbg_text_input][dbg_text_input] for the provided field name, with an optional custom `label`
* `.Slider(field_name, min, max, [label])` - Creates a [dbg_slider][dbg_slider] for the provided field name with the specified `min` and `max` values, and an optional custom `label`
* `.Sprite(field_name, [label])` - Creates a [dbg_sprite][dbg_sprite] for the provided field name, with an optional custom `label`
  * NOTE: Not recommended for large sprites as dbg_sprite will render the sprite at full size. You may want to use `.SpritePicker()` instead (see below)

#### Pickers (dropdowns)
* `.Picker(field_name, choices, [label])` - Creates a dropdown wth the provided `choices` Array as options. The name in the dropdown will be whatever `string(choice)` returns.
* `.LabeledPicker(field_name, choices, labels, [label])` - Creates a dropdown wth the provided `choices` Array as options, where the labels will be the corresponding string from the `labels` Array.
* `.FontPicker(field_name, [label])` - Creates a dropdown listing the current font for the named field, and allowing the user to pick from the list of all fonts
* `.SpritePicker(field_name, [label])` - Creates a dropdown listing the current sprite for the named field, and allowing the user to pick from the list of all sprites
* `.FilteredSpritePicker(field_name, filter_func, [label])` - Creates a dropdown listing the current sprite and allowing the user to pick from a filtered list of sprites
  * `filter_func` is of the form `(sprite_asset, sprite_name) -> Bool`, where returning `true` will include that sprite in the list.
* `.AssetPicker(field_name, asset_type, asset_name_func, [label])` - Creates a dropdown listing the current asset for the named field, and lists all of the assets of the provided `asset_type` using the `asset_name_func` to determine the name to display
  * NOTE: This is provided so that you can create Asset Pickers for things besides Font and Sprite, but over time Inspectron will get more functions for each Asset type as I get around to it
 
#### Advanced
* `.Include(field_name, [label])` - `field_name` should be the name of variable pointing to another GM instance or struct, with its own Inspectron defined on it. A label will be rendered, followed by all of the fields defined by the Inspectron on the referenced instance/struct (which can `.Include()` more instances/structs!), indented accordingly.
  * This can be very useful if, for example, your character instance has a `weapon` variable which is an instance or struct with its own variables. Define an Inspectron on that instance/struct and it can be displayed in the character's Inspectron!
* `.AtTop()` - You can call this after defining any other field (including `.Header()`/`.Button()`/etc) to ensure that the field will be displayed at the *top* of the Inspectron debug view, even if the object or struct is later extended by another inspectron (e.g. in the child object).
  * This can be useful with `.Button()`, for example, to expose some common functionality at the top of the inspector.

#### Others (experimental)
Some other functions may be defined in the Inspectron script file but should be considered experimental (and hence unsupported) if not listed above.

[fluent_api]: https://en.wikipedia.org/wiki/Fluent_interface
[dbg_section]: https://manual.gamemaker.io/monthly/en/GameMaker_Language/GML_Reference/Debugging/dbg_section.htm
[dbg_text]: https://manual.gamemaker.io/monthly/en/GameMaker_Language/GML_Reference/Debugging/dbg_text.htm
[dbg_button]: https://manual.gamemaker.io/monthly/en/GameMaker_Language/GML_Reference/Debugging/dbg_button.htm
[dbg_watch]: https://manual.gamemaker.io/monthly/en/GameMaker_Language/GML_Reference/Debugging/dbg_watch.htm
[dbg_color]: https://manual.gamemaker.io/monthly/en/GameMaker_Language/GML_Reference/Debugging/dbg_color.htm
[dbg_checkbox]: https://manual.gamemaker.io/monthly/en/GameMaker_Language/GML_Reference/Debugging/dbg_checkbox.htm
[dbg_text_input]: https://manual.gamemaker.io/monthly/en/GameMaker_Language/GML_Reference/Debugging/dbg_text_input.htm
[dbg_slider]: https://manual.gamemaker.io/monthly/en/GameMaker_Language/GML_Reference/Debugging/dbg_slider.htm
[dbg_sprite]: https://manual.gamemaker.io/monthly/en/GameMaker_Language/GML_Reference/Debugging/dbg_sprite.htm
