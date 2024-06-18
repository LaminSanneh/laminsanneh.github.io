---
title: React Redux Toolkit How to Set Up, Understand, and Use the library in Your React.js App
description: ""
date: 2024-06-18T11:09:36.526Z
preview: ""
draft: false
tags: []
categories:
    - React.js
type: posts
slug: react-redux-toolkit-set-understand-library-react-js-app
---

Integrating Redux into your React application can enhance your state management, making your app more robust and scalable. Redux Toolkit simplifies this process by providing a set of tools and best practices. This guide will walk you through setting up Redux Toolkit in a React application, understanding its key components, and effectively using it to manage your app's state.

### Step 1: Setting Up Redux Toolkit

```bash
npm install @reduxjs/toolkit react-redux
```

#### Configuring the Store
In your project, create a file named store.ts for setting up the Redux store and reducers.

`store.ts`
```ts
import { combineReducers } from "redux";
import postSlice from "./slices/postSlice";
import userSlice from "./slices/userSlice";
import { configureStore } from "@reduxjs/toolkit";
import { useDispatch, useSelector, useStore } from "react-redux";

// Combine your slice reducers
const rootReducer = combineReducers({
  post: postSlice,
  user: userSlice,
});

// Configure the store
export const store = configureStore({
  reducer: rootReducer,
});

// Define types for RootState, AppDispatch, and AppStore
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
export type AppStore = typeof store;

// Custom hooks to use throughout your app
export const useAppDispatch = useDispatch.withTypes<AppDispatch>();
export const useAppSelector = useSelector.withTypes<RootState>();
export const useAppStore = useStore.withTypes<AppStore>();

```

In this setup, combineReducers is used to combine postSlice and userSlice reducers into a single root reducer. The configureStore function from Redux Toolkit is then used to create the store.

### Step 2: Creating Slices
Slices in Redux Toolkit combine the reducer logic and actions into a single file. Let's create a postSlice to manage the state of posts in your application.

`slices/postSlice.ts`

```ts
import { createAsyncThunk, createSlice } from "@reduxjs/toolkit";
import axios from "axios";
import { Post } from "../types"; // Assuming you have a Post type defined

// Async thunk for fetching posts
export const fetchPosts = createAsyncThunk<Post[], void>(
  "post/fetchPosts",
  async () => {
    try {
      const response = await axios.get("http://localhost:8000/api/posts");
      return response.data;
    } catch (error) {
      throw new Error("Failed to get posts data");
    }
  }
);

// Initial state
const initialState = {
  posts: [],
  loading: false,
  error: null,
};

// Create the post slice
const postSlice = createSlice({
  name: "post",
  initialState,
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchPosts.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchPosts.fulfilled, (state, action) => {
        state.loading = false;
        state.posts = action.payload;
      })
      .addCase(fetchPosts.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || "Failed to load posts";
      });
  },
});

export default postSlice.reducer;

```

Here, createAsyncThunk is used to handle asynchronous actions. This function helps to manage different states like pending, fulfilled, and rejected. The slice manages the state changes in response to these actions.

### Step 3: Integrating Redux Store with React
To use the Redux store in your React app, wrap your root component with the Provider component from react-redux.

`App.tsx`

```ts
import React from "react";
import { Provider } from "react-redux";
import { store } from "./store.ts";
import YourComponent from "./YourComponent"; // Import your main component

const App: React.FC = () => {
  return (
    <Provider store={store}>
      <YourComponent />
    </Provider>
  );
};

export default App;

```

Wrapping your application with the Provider component makes the Redux store available to all components in your app.


### Step 4: Using Custom Hooks for State and Dispatch
To interact with the Redux store, use the custom hooks useAppDispatch and useAppSelector that you defined in StorageEvent.ts.

Example Usage in a Component

```tsx
import React, { useEffect } from "react";
import { useAppDispatch, useAppSelector } from "./StorageEvent";
import { fetchPosts } from "./slices/postSlice";

const PostList: React.FC = () => {
  const dispatch = useAppDispatch();
  const { posts, loading, error } = useAppSelector((state) => state.post);

  useEffect(() => {
    dispatch(fetchPosts());
  }, [dispatch]);

  if (loading) {
    return <div>Loading...</div>;
  }

  if (error) {
    return <div>Error: {error}</div>;
  }

  return (
    <div>
      <h1>Posts</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
};

export default PostList;

```

In this component, useAppDispatch is used to dispatch the fetchPosts action, and useAppSelector is used to select the post state from the Redux store.

### Conclusion
By following these steps, you can set up Redux Toolkit in your React application to manage your state efficiently. Redux Toolkit streamlines the process of writing Redux logic, making your code cleaner and easier to maintain. Happy coding!
