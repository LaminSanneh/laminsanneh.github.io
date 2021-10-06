---
title: Mongoose Connection
weight: 100
menu:
  notes:
    name: Mongoose Connection
    identifier: notes-mongoose-js-basic-connection
    parent: notes-mongoose-js-basic
    weight: 10
---

<!-- Variable -->
{{< note title="Connect to Mongoose Database" >}}

```js
const mongoose = require('mongoose');

const db = mongoose.connection;

mongoose.connect("mongodb://localhost:27017/appname");

db.on('open', () => {
    console.log('Connected to mongoDB');
});

db.on('error', (error) => {
    console.log(error);
});
```

{{< /note >}}
