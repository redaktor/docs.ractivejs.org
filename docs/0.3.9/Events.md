[[Home]] > [[API Reference]] > [[Events]]

Like many libraries, Ractive implements the [publish/subscribe](http://addyosmani.com/blog/understanding-the-publishsubscribe-pattern-for-greater-javascript-scalability/) mechanism to allow you to trigger behaviour at certain points.

There are three types of events - standard events, proxy events, and custom events. They all use the same interface - [[ractive.on()]]:

```js
ractive = new Ractive({
  el: myContainer,
  template: myTemplate
});

ractive.on( 'teardown', function () {
  alert( 'Bye!' );
});
```

In all cases, event handlers will be called with `ractive` as `this`.

## Standard events

Standard events are those emitted by all Ractive instances:

### set

Fired whenever [[ractive.set()]] is called. The handler's arguments will be `(keypath, value)` or `(map)` depending on the arguments to [[ractive.set()]].

### update

Fired whenever [[ractive.update()]] is called, with `(keypath)` as the sole argument (unless no keypath was specified, in which case there are no arguments).

### teardown

Fired whenever [[ractive.teardown()]] is called. No arguments.


## Proxy events

Proxy events are a convenient way of handling user interaction. Instead of doing `someElement.addEventListener( eventName, handler )` or `$( someSelector ).on( eventName, handler )`, you can write the name of a *proxy event* right into your template:

```html
<button on-click='activate'>Activate!</button>
<button on-click='deactivate'>Deactivate!</button>
```

```js
ractive = new Ractive({
  el: myContainer,
  template: myTemplate
});

ractive.on({
  activate: function ( event ) {
    // do something
  },
  deactivate: function ( event ) {
    // do something
  }
});
```

This way we don't have to bother traversing the DOM or storing references to elements and handlers. We also don't need to worry about removing event handlers when we detach chunks of the DOM - Ractive takes care of it.

Proxy event handlers receive an `event` argument. This isn't the same as the `event` argument you'd receive in a traditional handler (i.e. bound with `node.addEventListener` or `$(node).on()` etc), but an object with five properties:

* `node` - the relevant element
* `original` - the underlying DOM event (useful for `event.original.preventDefault()` etc)
* `keypath` - the keypath of the current context
* `context` - the value of `ractive.get(event.keypath)`
* `index` - a map of index references

*TODO - explain what the hell the last three of those things meant... In the meantime folks, do a `console.log( event )` inside your handlers and all will be revealed!*

Because of the way proxy events work, they more or less eliminate the need for event delegation.

Any standard DOM event that an element supports can be used (e.g. `on-mouseover='highlight'`, `on-touchstart='dragstart'`, `on-blur='submit'`, `on-error='loadFallbackImage'`, and so on), as can non-standard events defined in [[ractive-events]] such as `on-tap`.

*TODO - chained proxy events, additional arguments in custom event definitions*

## Custom events

You can easily create events of your own with [[ractive.fire()]]. This is most useful in the context of subclasses created with [[Ractive.extend()]].

In this example, we respond to the user clicking on an item by firing a `selected` event with the item data as the first (and only) argument:

```html
<div class='catalogue'>
  {{#items:i}}
    <div on-tap='select' data-index='{{i}}'>
      {{>content}}
    </div>
  {{/items}}
</div>
```

```js
var Catalogue = Ractive.extend({
  template: catalogueTemplate,
  partials: { content: contentPartial }, // pretend we made one
  init: function ( options ) {
    var self = this, items = options.items;

    this.on( 'select', function ( event ) {
      var index = event.node.getAttribute( 'data-index' );
      self.fire( 'selected', items[ index ] );
    });
  }
});
```

