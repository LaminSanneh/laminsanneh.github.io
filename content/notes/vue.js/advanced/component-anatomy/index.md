---
title: Component Anatomy
weight: 70
menu:
  notes:
    name: Component Anatomy
    identifier: notes-vuejs-advanced-component-anatomy
    parent: notes-vuejs-advanced
    weight: 20
---

{{< note title="Vuejs Component Anatomy">}}
  
```html
<template>
  <span>{{ message }}</span>
</template>

<script>
  import ProductComponent from '@/components/ProductComponent'
  import ReviewComponent from '@/components/ReviewComponent'
  
  export default {
    components: { // Components that can be used in the template
      ProductComponent,
      ReviewComponent
    },
    props: { // The parameters the component accepts
      message: String,
      product: Object,
      email: {
        type: String,
        required: true,
        default: 'none',
        validator: function (value) {
          // Should return true if value is valid
        }
      }
    },
    data: function () { // Must be a function
      return {
        firstName: 'Vue',
        lastName: 'Mastery'
      }
    },
    computed: { // Return cached values until dependencies change
      fullName: function () {
        return `${this.firstName} ${this.lastName}`
      }
    },
    watch: { // Called when firstName changes value
      firstName: function (value, oldValue) { /* ... */ }
    },
    methods: { /* ... */ }
  }
</script>
```
{{< /note >}}