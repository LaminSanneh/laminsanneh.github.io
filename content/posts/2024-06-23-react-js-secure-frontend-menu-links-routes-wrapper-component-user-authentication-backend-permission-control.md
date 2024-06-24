---
title: React.js How to use User Roles to Secure Frontend Menu Links and Routes Using a Wrapper Component for User Authentication and Backend Permission Control
description: ""
date: 2024-06-23T12:35:17.035Z
preview: null
draft: false
tags: []
categories:
    - React.js
type: posts
slug: react-js-user-roles-secure-frontend-menu-links-routes-wrapper-component-user-authentication-backend-permission-control
hero: /images/posts/2024-06-23-react-js-secure-frontend-menu-links-routes-wrapper-component-user-authentication-backend-permission-control.png
---

In modern web applications, managing user authentication and permissions is crucial. One effective way to handle this in React.js is by using a wrapper component. This article will guide you through creating a wrapper component that controls access to routes and links based on user roles and permissions. We will leverage Redux Toolkit to manage user state and permissions.

### Prerequisites

Before we start, make sure you have the following setup:

* A React project created using Create React App.
* Redux Toolkit installed and configured in your project.

### Setting Up Redux Toolkit

First, let's set up Redux Toolkit to manage user state. Assume we have a slice called userSlice that stores user data, including their roles and permissions.

`src/store/slices/userSlice.js`
```tsx
import { createSlice } from '@reduxjs/toolkit';

export interface Role {
  id: number;
  name: string;
}

const initialState = {
  isAuthenticated: false,
  user: {
    roles: Role[]
  },
};

const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    login: (state, action) => {
      state.isAuthenticated = true;
      state.user = action.payload;
    },
    logout: (state) => {
      state.isAuthenticated = false;
      state.user = { roles: [] };
    },
  },
});

export const { login, logout } = userSlice.actions;
export default userSlice.reducer;

```

### Creating the Authentication Wrapper Component

Let's create a wrapper component that will control access to certain routes based on the user's roles and permissions.
This will allow the user to see the page if the user has atleast one of the list of required roles. For example if we supplied the ids for `adminRole` and `editorRole` as in [adminRoleId, editorRoleId] and user has role of [adminRoleId], since they have atleast one of the required roles, then they can view the page.

`src/components/ProtectedRoute.js`
```tsx
import React from "react";
import { Navigate } from "react-router-dom";
import { useSelector } from 'react-redux';

type ProtectedRouteProps = {
  children: React.JSX.Element;
  requiredRoles: number[];
};

const ProtectedRoute: React.FC<ProtectedRouteProps> = ({
  children,
  requiredRoles,
}) => {
  const userRoles = useSelector((state) => state.user.user.roles);
  const isAuthenticated = useSelector((state) => {
    return state.user.isAuthenticated;
  });

    // Alternative syntax to the above
    //   const { isAuthenticated, user } = useSelector((state) => {
    //     return state.user;
    //   });
    // const userRoles = user.roles;

  if (!isAuthenticated) {
    return <Navigate to="/login" />;
  }

  if (requiredRoles.length === 0 && userRoles) {
    return children;
  }

  const userRoleIds = userRoles?.map((userRole) => userRole.id);
  const hasRequiredRole =
    userRoleIds &&
    requiredRoles.some((role) => {
      return userRoleIds.includes(role);
    });

  if (!hasRequiredRole) {
    return <div>You do not have permission to view this page.</div>;
  }

  return children;
};

export default ProtectedRoute;

```

### Using the ProtectedRoute Component
Wrap your route elements with the `ProtectedRoute` component to control access.

`src/App.ts`
```tsx
import React from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import { useSelector } from 'react-redux';
import ProtectedRoute from './components/ProtectedRoute';
import HomePage from './pages/HomePage';
import AdminPage from './pages/AdminPage';
import UserDashboard from './pages/UserDashboard';
import LoginPage from './pages/LoginPage';

const App = () => {
  const { isAuthenticated } = useSelector((state) => state.user);

// These come from backend and represent the role database id's. You could query them through through an api or render them using server side rendering or
// just hardcode them like here
  const AVAILABLE_ROLES = {
    ADMIN_ROLE_ID: 1,
    EDITOR_ROLE_ID: 2,
    USER_ROLE_ID: 3,
  };

  return (
    <Router>
      <Routes>
        <Route path="/login" element={<LoginPage />} />
        <Route path="/" element={<HomePage />} />
        <Route
          path="/admin"
          element={
            <ProtectedRoute requiredRoles={[AVAILABLE_ROLES.ADMIN_ROLE_ID]}>
              <AdminPage />
            </ProtectedRoute>
          }
        />
        <Route
          path="/admin"
          element={
            <ProtectedRoute requiredRoles={[
                AVAILABLE_ROLES.ADMIN_ROLE_ID,
                AVAILABLE_ROLES.EDITOR_ROLE_ID,
                AVAILABLE_ROLES.USER_ROLE_ID
            ]}>
              <UserDashboard />
            </ProtectedRoute>
          }
        />
      </Routes>
    </Router>
  );
};

export default App;
```

### Creating the Navigation Link Wrapper
Now, let's create a wrapper component for navigation links that will only show the link if the user has the required permissions.

`src/components/ProtectedLink.ts`
```tsx
import React from "react";
import { Link, To } from "react-router-dom";
import { useSelector } from 'react-redux';

type ProtectedLink = {
  children: React.JSX.Element | string;
  requiredRoles: number[];
  to: To;
  className?: string;
};

const ProtectedLink: React.FC<ProtectedLink> = ({ to, requiredRoles, children, ...props }) => {
  const userRoles = useSelector((state) => state.user.user?.roles);
  const isAuthenticated = useSelector((state) => {
    return state.user.isAuthenticated;
  });

  if (!isAuthenticated) {
    return null;
  }

  const { className } = props;

  const userRoleIds = userRoles?.map((userRole) => userRole.id);
  const hasRequiredRole =
    userRoleIds &&
    requiredRoles.some((role) => {
      return userRoleIds.includes(role);
    });

  if (!hasRequiredRole) {
    return null;
  }

  return <Link className={className} to={to}>{children}</Link>;
};

export default ProtectedLink;
```

### Using the ProtectedLink Component
Use the ProtectedLink component to conditionally render navigation links.

`src/components/Navigation.ts`
```tsx
import React from 'react';
import ProtectedLink from './components/ProtectedLink';

// These come from backend and represent the role database id's. You could query them through through an api or render them using server side rendering or
// just hardcode them like here
const AVAILABLE_ROLES = {
    ADMIN_ROLE_ID: 1,
    EDITOR_ROLE_ID: 2,
    USER_ROLE_ID: 3,
  };

const Navigation = () => {
  return (
    <nav>
      <ul>
        <li>
          <ProtectedLink to="/admin" requiredPermissions={[AVAILABLE_ROLES.ADMIN_ROLE_ID]}>Admin</ProtectedLink>
        </li>
      </ul>
    </nav>
  );
};

export default Navigation;
```

### Creating the Unprotected Navigation Link Wrapper
Now, let's create a wrapper component for navigation links that will hide the link if the user is loggedin. For example, we want to hide the login and registration links if the user is already logged in.

`src/components/UnProtectedLink.ts`
```tsx
import React from "react";
import { Link, To } from "react-router-dom";
import { useSelector } from 'react-redux';

type UnProtectedLinkProps = {
  children: React.JSX.Element | string;
  to: To;
  className?: string;
};

const UnProtectedLink: React.FC<UnProtectedLinkProps> = ({ to, children, ...props }) => {
  const isAuthenticated = useSelector((state) => {
    return state.user.isAuthenticated;
  });
  const user = useSelector((state) => {
    return state.user.user;
  });

  if (isAuthenticated && user) {
    return null;
  }

  const { className } = props;

  return <Link className={className} to={to}>{children}</Link>;
};

export default UnProtectedLink;
```

### Using the UnProtectedLink Component
Use the ProtectedLink component to conditionally render navigation links.

`src/components/Navigation.ts`
```tsx
import React from 'react';
import UnProtectedLink from './components/UnProtectedLink';

const Navigation = () => {
  return (
    <nav>
      <ul>
        <li>
          <UnProtectedLink to="/login">Login</UnProtectedLink>
          <UnProtectedLink to="/register">Register</UnProtectedLink>
        </li>
      </ul>
    </nav>
  );
};

export default Navigation;
```
When ever the user is logged in these links will be hidden. When the user logs out, then the will be shown

### Conclusion
In this article, we explored how to create wrapper components in React.js to control access to route elements and navigation links based on user authentication and roles. By leveraging Redux Toolkit and the `useSelector` hook, we can easily manage user data and implement permission control in a clean and efficient way.

Using these techniques, you can build more secure and user-friendly applications by ensuring that only authorized users can access certain parts of your application. Happy coding!
