---
title: Expressions
weight: 10
menu:
  notes:
    name: Expressions
    identifier: notes-vuejs-basic-expressions
    parent: notes-vuejs-basic
    weight: 10
---

{{< note title="Vuejs Expressions">}}
```html
<div id="app">
  <p>I have a {{ product }}</p>
  <p>{{ product + 's' }}</p>
  <p>{{ isWorking ? 'YES' : 'NO' }}</p>
  <p>{{ product.getSalePrice() }}</p>
</div>
```
{{< /note >}}
