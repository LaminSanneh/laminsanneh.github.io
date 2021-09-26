---
title: Binding
weight: 20
menu:
  notes:
    name: Bindings
    identifier: notes-vuejs-basic-binding
    parent: notes-vuejs-basic
    weight: 20
---

{{< note title="Binding">}}
Fullhand
```html
<a v-bind:href="url">...</a>
```

Shorthand
```html
<a :href="url">...</a>
```

{{< /note >}}

{{< note title="Binding2">}}
True or false will add or remove attribute:
```html
<button :disabled="isButtonDisabled”>...
```

If isActive is truthy, the class ‘active’ will appear:
```html
<div :class="{ active: isActive }">...
```

Style color set to value of activeColor:
```html
<div :style="{ color: activeColor }">
```

{{< /note >}}
