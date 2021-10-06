---
title: Binding
weight: 20
menu:
  notes:
    name: Binding
    identifier: notes-vuejs-basic-binding
    parent: notes-vuejs-basic
    weight: 20
---

{{< note title="Vuejs Binding">}}
Fullhand attribute binding
```html
<a v-bind:href="url">...</a>
```

Shorthand
```html
<a :href="url">...</a>
```

Two way data binding
```html
<input v-model="firstName">
```
{{< /note >}}

{{< note title="Vuejs Binding2">}}
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

{{< note title="Vuejs Binding3">}}
Toggles the display: none CSS property:
```html
<p v-show="showProductDetails">
  ...
</p>
```

Passing arguments to a computed binding:
```html
<template>
  <ProductComponent :cost="product_type('product_2')"></ProductComponent>
</template>

<script>
  import ProductComponent from @/components/ProductComponent
  
  export default {
    components: { ProductComponent },
    data() {
      return {
        products: {
          product_1: '100',
          product_2: '200',
          product_3: '300'
        }
      }
    },
    computed: {
      product_type() {
        // Argument passed to arrow function, NOT computed function declaration.
        return (product_id) => { // Arrow function to allow 'this' instance to be accessible.
          return this.products[product_id]  // Square bracket notation for 'any' type variable
        }
      }
    }
  }
</script>
```
{{< /note >}}