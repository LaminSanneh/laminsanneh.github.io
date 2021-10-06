---
title: Custom Events
weight: 80
menu:
  notes:
    name: Custom Events
    identifier: notes-vuejs-advanced-custom-events
    parent: notes-vuejs-advanced
    weight: 30
---

{{< note title="Vuejs Custom Events">}}
Use props to pass data into child components, custom events to pass data to parent elements.
Set listener on component, within its parent:
```html
<button-counter v-on:incrementBy="incWithVal">...</button-counter>
```

Inside parent component:
```js
methods: {
  incWithVal: function (toAdd) { ... }
}
```

Inside button-counter template:
```js
this.$emit('incrementyBy', 5)
```

{{< /note >}}