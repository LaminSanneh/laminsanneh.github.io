---
title: Mongoose Schema
weight: 101
menu:
  notes:
    name: Mongoose Schema
    identifier: notes-mongoose-js-basic-schema
    parent: notes-mongoose-js-basic
    weight: 11
---

<!-- Variable -->
{{< note title="Mongoose Schema" >}}

Basic Schema
```js
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const userSchema = new Schema({
    name: { type: String, required: true, max: 100 },
    email: { type: String, required: false, max: 100, default: null },
});

module.exports = mongoose.model('user', userSchema);
```

{{< /note >}}
