---
title: List Rendering
weight: 30
menu:
  notes:
    name: List Rendering
    identifier: notes-vuejs-basic-list-rendering
    parent: notes-vuejs-basic
    weight: 30
---

{{< note title="Vuejs List Rendering">}}
Basic array loop
```html
<li v-for="item in items" :key="item.id">
  {{ item }}
</li>
```

Loop with access to index
```html
<li v-for="(item, index) in items">...</li>
```

Baisc object loop
```html
<li v-for="value in object">...</li>
<li v-for="(value, index) in object">...</li>
<li v-for="(value, name, index) in object">...</li>
```

Loop using component instead of html native element e.g. div, li
```html
<cart-product v-for="item in products" :product="item" :key="item.id">
```

{{< /note >}}
