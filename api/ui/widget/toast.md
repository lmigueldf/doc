# Class: ui.widget.Toast

Inherits from
:    1. [ui.ImageView](./ui-images.html#class-ui.imageview)
     2. [ui.View](./ui-view.html)
     3. [event.Emitter](./event.html#class-event.emitter)

Widget for showing toast notifications.

## Examples

* [Toast](../example/ui-toast/)

## Methods

### new Toast ([options])

Parameters
:    1. `options {object}`
	     * `position {string} = 'bottom'` ---Where should this toast pop up?
		 * `images {object} = {}` ---What do your toasters look like? Only define what you need.
		 * `debug {boolean} = false` ---If true, your toaster gets a red background.

You can make a Toast with any of the following values for `position`: 'top', 'topright', 'bottomright', 'bottom', 'bottomleft', 'topleft'.

~~~
import ui.widget.Toast as Toast;

var toast = new Toast({
	superview: mainView,
	position: 'topright',
	images: {
		topright: 'resources/images/toaster.png'
	}
});
~~~
