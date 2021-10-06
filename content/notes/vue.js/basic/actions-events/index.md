---
title: Actions/Events
weight: 40
menu:
  notes:
    name: Actions/Events
    identifier: notes-vuejs-basic-actions-events
    parent: notes-vuejs-basic
    weight: 40
---

{{< note title="Vuejs Actions/Events">}}
```html
<button v-on:click="addToCart">...</button> // Fullhand
<button @click="addToCart">...</button> // Shorthand
```

Arguments can be passed:
```html
<button @click="addToCart(product)">...</button>
```

To prevent default behaviour (e.g. page reload):
```html
<form @submit.prevent="addProduct">...</form>
```

Only trigger once:
```html
<img @mouseover.once="showImage">
.stop // Stop all event propagation
.self // Only trigger if event.target is element itself
```

Keyboard entry example:
```html
<input @keyup.enter="submit">
```

Call onCopy when control-c is pressed:
```html
<input @keyup.ctrl.c="onCopy">
```

Key modifiers:
```js
.tab
.delete
.esc
.space
.up
.down
.left
.right
.ctrl
.alt
.shift
.meta
```

Mouse modifiers:
```js
.left
.right
.middle
```

{{< /note >}}
