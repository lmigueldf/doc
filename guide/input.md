# Input Handling

## Examples

* [Draggable Views](../example/events-input-drag/)
* [Gesture Input](../example/events-input-gesture/)

## Basic Input

The most basic input handling of all consists of capturing click events. Here's now:

~~~
view.on('InputSelect', function() {
	console.log("We just got a click!");
});
~~~

`'InputSelect'` is an `InputEvent`. Read more about the `InputEvent` class [here](/api/event.html#class-event.input.inputevent). In addition to `'InputSelect'`, there are several other input events which can be subscribed to, including: `'InputStart'`, `'InputMove'`, `'InputOver'`, and `'InputOut'`. See [full input event documentation](/api/ui-view.html#input-events) to learn more about these and other events.

## Drag Input

### Drag Events

Drag input can be detected using the `'DragStart'`, `'Drag'`, and `'DragStop'` events. Invoke the `'startDrag'` function on `'InputStart'` to access these events, as in [this example](/api/ui-view.html#startdrag-options).

### Simple Dragging

If you simply want to make some view draggable, use `drag.makeDraggable`.

~~~
import ui.View;
import event.input.drag as drag;

var view = new ui.View({
	superview: someSuperView,
	width: 100,
	height: 100
});

drag.makeDraggable(someView);
~~~

You can pass in an options object as the second parameter of `drag.makeDraggable`. With the default options, this draggable view cannot be dragged beyond the bounds of its superview. With `unbound: true` and `cover: true`, the draggable view can never be dragged off of its superview. This can be very useful with a clipping superview that essentially serves as a window into the draggable view. Please see our [full makeDraggable documentation](/api/event.html#drag-events-event.input.drag) to learn more.

## Advanced Input

If you're interested in more events, such as `'Swipe'`, `'Pinch'`, `'Rotate'`, `'DragSingle'`, `'FingerDown'`, and `'FingerUp'`, check out the `GestureView`. Many games will have a single input layer sitting on top of everything else. The `GestureView` is a great choice for this because it manages complex input under the hood and emits simple events which are easily handled in game code. As a `View` subclass, it still gives you access to the more straightforward events, such as `'InputStart'` and `'InputSelect'`, which can easily be mapped to the underlying UI elments. Check out the full API documentation [here](/api/ui-gestureview.html).

## Blocking Input

As we mentioned [above](/guide/input.html#advanced-input), a common optimization is to move all your input handling into a single layer (see discussion [here](/guide/game-walkthrough.html#where-to-go-from-here)). This is easily accomplished with two properties: `canHandleEvents` and `blockEvents`.


### canHandleEvents


### blockEvents