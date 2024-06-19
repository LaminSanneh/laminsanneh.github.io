---
title: How to Make a Dynamic Multi Checkbox System in React.js Seamlessly
description: ""
date: 2024-06-19T15:11:57.350Z
preview: ""
draft: false
tags: []
categories: []
type: posts
slug: dynamic-multi-checkbox-system-react-js-seamlessly
hero: /images/posts/2024-06-19-how-to-make-a-dynamic-multi-checkbox-system-in-react.js-seamlessly.png
---

### Introduction
Creating a dynamic multi-checkbox system in React.js can be a powerful way to allow users to select multiple options, such as user roles, in an intuitive and interactive manner. In this blog post, we'll walk through a complete example of how to implement such a system, focusing on updating the state based on user interactions and fetching initial data from an external source. Let's dive into the code and explore each part of the implementation.

### Component Overview
The component we'll be discussing is `MyUserRolesComponent`, which allows users to select multiple roles via checkboxes. The component initializes with all checkboxes unchecked and updates its state as users interact with it. Additionally, it fetches existing role data for a user from a server and updates the checkboxes accordingly.

Here's the full component code for reference:

```tsx
import React, { useState, useEffect } from 'react';

function MyUserRolesComponent () {
  // By default all checkbox values are set to false
  const selectedRoleValues = {
    1: false,
    2: false,
    3: false
  };

  // component state for holding the selected roles in the dom
  const [selectedRoles, setSelectedRoles] = useState<{[index: number]: boolean}>(selectedRoleValues);

  // handler which updates the state selectedRoles whenever a user checks or unchecks a role
  const onSelectedRoleChange = (roleId: number) => {
    setSelectedRoles({...selectedRoles, ...{[roleId]: !selectedRoles[roleId]}});
  }

  useEffect(() => {
    let ignoreFetchedUserData = false;

    if (!route.id) {
      return;
    }

    dispatch(fetchUser(Number.parseInt(route.id, 10)))
      .unwrap()
      .then((response) => {
        if (ignoreFetchedUserData) {
          // this is the result of the server call using some reducer action which returns the user roles from some server
          // for now, we abstract the function call
          const roles: {[index: number]: boolean}[]  = response.roles;
          const selectedRolesFromServer = Object.assign({}, ...roles.map(role => ({ [role.id]: true })));

          // here we update the selectedRoles state by overriding it with some values from the server
          setSelectedRoles({...selectedRoles, ...selectedRolesFromServer});
        }
      })
      .catch(() => {
        console.log("Error fetching user");
      });

    return () => {
      console.log("Clearing Use effect");
      ignoreFetchedUserData = true;
    };
  }, [route, dispatch]);

  return (
    <div>
      {
          availableRoles.map((role) => {
            return (
                  <label key={role.id}>
                    <input
                      type="checkbox"
                      checked={selectedRoles[role.id]}
                      onChange={() => { onSelectedRoleChange(role.id); }}
                    />{" "}
                    {role.name}
                  </label>
            );
          })
        }
    </div>
  );
}

export default MyUserRolesComponent;
```

Lets now dive deeper and explain each part:

### State Initialization
We start by initializing the state for the checkboxes. Each checkbox corresponds to a user role, identified by a unique ID. Initially, all checkboxes are set to `false` (unchecked).

```tsx
const selectedRoleValues = {
  1: false,
  2: false,
  3: false
};
const [selectedRoles, setSelectedRoles] = useState<{[index: number]: boolean}>(selectedRoleValues);
```

### Handling Checkbox Changes
The `onSelectedRoleChange` function is a handler that toggles the checked state of a checkbox when it is clicked. It updates the state by flipping the boolean value of the corresponding role ID.

```tsx
const onSelectedRoleChange = (roleId: number) => {
  setSelectedRoles({...selectedRoles, ...{[roleId]: !selectedRoles[roleId]}});
}
```

### Fetching Initial Data
The `useEffect` hook is used to fetch the initial roles from the server when the component mounts. It updates the state with the roles fetched for the user, allowing the component to reflect the existing data.

```tsx
useEffect(() => {
  let ignoreFetchedUserData = false;

  if (!route.id) {
    return;
  }

  dispatch(fetchUser(Number.parseInt(route.id, 10)))
    .unwrap()
    .then((response) => {
      if (!ignoreFetchedUserData) {
        const roles: {[index: number]: boolean}[]  = response.roles;
        const selectedRolesFromServer = Object.assign({}, ...roles.map(role => ({ [role.id]: true })));
        setSelectedRoles({...selectedRoles, ...selectedRolesFromServer});
      }
    })
    .catch(() => {
      console.log("Error fetching user");
    });

  return () => {
    console.log("Clearing Use effect");
    ignoreFetchedUserData = true;
  };
}, [route, dispatch]);
```

### Rendering Checkboxes
Finally, the component renders a list of checkboxes based on the `availableRoles` array. Each checkbox is controlled by the state, and clicking a checkbox triggers the `onSelectedRoleChange` handler.

```tsx
return (
  <div>
    {
      availableRoles.map((role) => {
        return (
          <label key={role.id}>
            <input
              type="checkbox"
              checked={selectedRoles[role.id]}
              onChange={() => { onSelectedRoleChange(role.id); }}
            />{" "}
            {role.name}
          </label>
        );
      })
    }
  </div>
);
```

### Conclusion
By following the steps outlined above, you can create a dynamic multi-checkbox system in React.js that seamlessly handles user interactions and updates based on external data. This approach ensures that your component is both user-friendly and robust, capable of reflecting the current state of user roles accurately.

Feel free to extend this example by adding more features, such as saving the selected roles back to the server or adding additional input validation. Happy coding!