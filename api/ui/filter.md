# Class: ui.filter.Filter

## Methods

### new Filter ([options])

Parameters
:    1. `options {object}`
         * `priority {number} = 0`
         * `r {number} = 0` ---Red component.
         * `g {number} = 0` ---Green component.
         * `b {number} = 0` ---Blue component.
         * `a {number} = 0` ---Alpha component.

A generic class that specialized filters inherit from.

~~~
import ui.filter.Filter as Filter;

var filter = new Filter();
~~~

Filters can be applied to views using [`View.addFilter`](/api/ui-view.html#addfilter-filter) and removed using [`View.removeFilter`](/api/ui-view.html#removefilter).

### get ()

Returns
:    1. `{object}`

Returns the filter options.

### getType ()

Returns
:    1. `{string}`

Returns the filter type.

### update ([options])

Parameters
:    1. `options {object}`

Update the filter using the values supplied in the given options.

### getColorString ()

Returns
:    1. `{string}`

Returns the color of a filter in the format: `"rgba(R, G, B, A)"`.


# Class: ui.filter.LinearAddFilter

Inherits from
:    1. [ui.filter.Filter](#class-ui.filter.filter)

Linear add (lighten) filter.

## Methods

### new LinearAddFilter ([options])

Parameters
:    1. `options {object}`

~~~
import ui.filter.LinearAddFilter as LinearAddFilter;
~~~


# Class: ui.filter.MultiplyFilter

Inherits from
:    1. [ui.filter.Filter](#class-ui.filter.filter)

Filter a view by multiplying pixels by the given rgb values
with the given strength. This filter has it's best use in
the case of having a grayscaled image and "colorizing"
it. (Such as using a single grayscaled image for a marble,
and coloring it with a multiply filter so as to reduce
assets).

Multiply filter.

### new MultiplyFilter ([options])

Parameters
:    1. `options {object}`

~~~
import ui.filter.MultiplyFilter as MultiplyFilter;
~~~