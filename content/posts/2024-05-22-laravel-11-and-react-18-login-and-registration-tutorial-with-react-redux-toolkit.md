---
title: Laravel 11 and React 18 Login and Registration tutorial with React Redux Toolkit
description: ""
date: 2024-05-28T09:17:32.053Z
preview: ""
draft: false
tags: []
categories:
  - Laravel
  - React.js
type: posts
slug: laravel-11-react-18-login-registration-tutorial-react-redux-toolkit
hero: /images/posts/2024-05-22-laravel-11-and-react-18-login-and-registration-tutorial-with-react-redux-toolkit.png
---

## Introduction

In this tutorial, we will be setting up an indepth tutorial guide of react.js and laravel 11 system for user authentication and registration. We will be using typescript on the react.js frontned, along with react redux toolkit, but do not worry as youc an follow along even if you only know javascript. React redux toolkit helps with state management. It might be overkill for a simple login registration app like this, but makes life very easy once the app codebase grows. Users will be able to register using an email as their username and password. After successful registration, they shall be able to login. We wil demonstrate the login mechanism by having a userpforile area/route in the frontend of the application which will be locked and only accessible to logged in. The frontend and backend will be on different domains, so the after logging in, we will provide the user with a token, which will be stored in [localstorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage) of the browser. This will be send along in the header each request which requires authentication to get data back from the server. We will also need to setup [cors](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) so that our frontend domain can send requests to the backend domanin app.

## Project Structure

Firstly, please clone the starter repositories here, one for the [laravel backend](https://github.com/LaminSanneh/tutorial-repo-implementations/tree/main/base-projects/starter-kit-laravel) and one for the [react frontend](https://github.com/LaminSanneh/tutorial-repo-implementations/tree/main/base-projects/starter-kit-react). Put each of them in a separate folder. We already have all tthe required libraries added on the frontend inside `package.json` as listed below.

```
- react-dom": "^18.2.0",
- "@reduxjs/toolkit": "^2.2.1",
- "axios": "^1.6.7",
- "react-redux": "^9.1.0",
- "react-router-dom": "^6.22.3",
- "redux": "^5.0.1"
```

I used [vite](https://vitejs.dev/) to create the frontend project and [composer](https://getcomposer.org/) to create the backend project in two seperate folders. An alternative would have been to host the react project directly inside the laravel project folder, hence having one domain for both frontend and backend, but we won;t be doibg that in this tutorial. Possibly, we may do it in another blog post.

## Frontend

In the terminal, move inside the frontend folder using `cd your-frontend-folder` and run `npm install`. After successfully installing the libraries, run `npm run dev` to start the frontend server. Verify that the server is running at ` http://localhost:5173/`.

### Create components for LoginComponent.tsx and RegisterComponent.tsx and UserProfileComponent.tsx

Create a component in `src/components` as below:

`src/components/LoginComponent.tsx`

```tsx
import React from "react";
import "./LoginComponent.css";

const LoginComponent = () => {
  return (
    <div>
      <h2>Login</h2>
      <form className="login-form">
        <label htmlFor="username">
          <input type="text" placeholder="username" id="username" />
        </label>
        <label htmlFor="password">
          <input type="password" placeholder="password" id="password" />
        </label>
        <button>Login</button>
      </form>
    </div>
  );
};

export default LoginComponent;
```

with css file `src/components/LoginComponent.css`:

```css
.login-form label {
    display: block;
    margin-bottom: 10px;
}

.login-form label input {
    padding: 10px;
}
```

Create a component in `src/components/RegisterComponent.tsx`

```tsx
import React from "react";
import "./RegisterComponent.css";

const RegisterComponent = () => {
  return (
    <div>
      <h2>Register</h2>
      <form className="register-form">
        <label htmlFor="username">
          <input type="text" placeholder="username" id="username" />
        </label>
        <label htmlFor="password">
          <input type="password" placeholder="password" id="password" />
        </label>
        <button>Register</button>
      </form>
    </div>
  );
};

export default RegisterComponent;
```

with css file `src/components/RegisterComponent.css`:

```css
.register-form label {
    display: block;
    margin-bottom: 10px;
}
.register-form input {
    padding: 10px;
}
```

Create a component in `src/components/UserProfileComponent.tsx`

```tsx
import React from "react";
import "./UserProfileComponent.css";

const UserProfileComponent = () => {
  return (
    <div>
      <h2>Profile</h2>
      <form className="update-profile-form">
        <label htmlFor="name">
          <input type="text" placeholder="name" id="name" />
        </label>
        <label htmlFor="phone">
          <input type="text" placeholder="phone" id="phone" />
        </label>
        <label htmlFor="address">
            <textarea id="address" placeholder="address" ></textarea>
        </label>
        <button>Update Profie</button>
      </form>
    </div>
  );
};

export default UserProfileComponent;
```

with css file `src/components/UserProfileComponent.css`:

```css
.update-profile-form label {
    display: block;
    margin-bottom: 10px;
}
.update-profile-form input, .update-profile-form textarea {
    padding: 10px;
}
```

### Add routes and links for Login, Register and UdateProfile

Modify `App.tsx` like below to add the three links for the different components.

```tsx
import "./App.css";
import { Link, Route, BrowserRouter, Routes } from "react-router-dom";
import LoginComponent from "./components/LoginComponent";
import RegisterComponent from "./components/RegisterComponent";
import UserProfileComponent from "./components/UserProfileComponent";

function App() {
  return (
    <BrowserRouter>
      <Link to={"login"}>Login</Link><> </>
      <Link to={"register"}>Register</Link><> </>
      <Link to={"profile"}>Profile</Link>
      <Routes>
        <Route path="/" element={<LoginComponent />} />
        <Route path="/login" element={<LoginComponent />} />
        <Route path="/register" element={<RegisterComponent />} />
        <Route path="/profile" element={<UserProfileComponent />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

### Create reducers for handling login and registration

Create a file in `src/store/slices/authReducer.ts`

```tsx
import { createAsyncThunk, createSlice } from "@reduxjs/toolkit";
import { makeLoginUserRequest, makeRegisterUserRequest } from "../../services/authService";

export interface LoginUserData {
  email: string;
  password: string;
}

export interface RegisterUserData {
  name: string;
  email: string;
  password: string;
}

interface AuthState {
  loading: boolean;
  error: string | null;
  token: string | null;
  isAuthenticated: boolean;
}

const initialState: AuthState = {
  loading: false,
  error: null,
  token: null,
  isAuthenticated: false,
};

export const loginUser = createAsyncThunk<string, LoginUserData>(
  "auth/loginUser",
  async (credentials) => {
    return await makeLoginUserRequest(credentials);
  }
);

export const registerUser = createAsyncThunk<void, RegisterUserData>(
  "auth/registerUser",
  async (credentials) => {
    return await makeRegisterUserRequest(credentials);
  }
);

const authSlice = createSlice({
  name: "auth",
  initialState,
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(loginUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(loginUser.fulfilled, (state, action) => {
        state.loading = false;
        state.token = action.payload;
      })
      .addCase(loginUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || "Failed to login user";
      })
      .addCase(registerUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(registerUser.fulfilled, (state) => {
        state.loading = false;
      })
      .addCase(registerUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || "Failed to register user";
      })
      .addDefaultCase(() => {});
  },
});

export default authSlice.reducer;

```

Then create the corresponding service file, `src/services/authService.ts` which will actually make the API HTTP calls to our laravel backend:

```ts
import axios from "axios";
import { LoginUserData, RegisterUserData } from "../store/slices/authReducer";
import authHeader from "./authHeader";

export const USER_TOKEN_KEY = "USER_TOKEN_KEY";

export const makeLoginUserRequest = async (credentials: LoginUserData) => {
  try {
    const token = (await axios.post(`http://localhost:8000/api/login`, credentials))
    .data

    authHeader.initializeToken(token);

    localStorage.setItem(USER_TOKEN_KEY, token);

    return token;
  } catch (error) {
    throw new Error("Failed to login user");
  }
};

export const makeRegisterUserRequest = async (credentials: RegisterUserData) => {
  try {
    return (await axios.post(`http://localhost:8000/api/register`, credentials))
      .data;
  } catch (error) {
    throw new Error("Failed to register user");
  }
};
```

### Create reducers for handling the fetching of userdata and updating profile

Create a file in `src/store/slices/userReducer.ts`

```tsx
import { createAsyncThunk, createSlice } from "@reduxjs/toolkit";
import { makeGetUserRequest, makeUpdatetUserRequest } from "../../services/userService";

export interface UpdateUserData {
  name: string;
  phone: string;
  address: string;
}

interface User {
  id: number;
  name: string;
  phone: string;
  address: string;
}

interface UserState {
  user: User | null;
  loading: boolean;
  error: string | null;
}

const initialState: UserState = {
  user: null,
  loading: false,
  error: null,
};

export const getUser = createAsyncThunk<User, void>(
  "auth/getUser",
  async () => {
    return await makeGetUserRequest();
  }
);

export const updateUser = createAsyncThunk<User, UpdateUserData>(
  "auth/updateUser",
  async (userData) => {
    return await makeUpdatetUserRequest(userData);
  }
);

const authSlice = createSlice({
  name: "auth",
  initialState,
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(getUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(getUser.fulfilled, (state, action) => {
        state.loading = false;
        state.user = action.payload;
      })
      .addCase(getUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || "Failed to get user data";
      })
      .addCase(updateUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(updateUser.fulfilled, (state, action) => {
        state.loading = false;
        state.user = action.payload;
      })
      .addCase(updateUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || "Failed to update user";
      });
  },
});

export default authSlice.reducer;

```

Then create the corresponding service file, `src/services/userService.ts`:

```ts
import axios from "axios";
import { UpdateUserData } from "../store/slices/userReducer";
import authHeader from "./authHeader";

export const makeGetUserRequest = async () => {
  try {
    const headers = {headers: authHeader.getAuthHeader()};
    return (await axios.get(`http://localhost:8000/api/getUserData`, headers)).data;
  } catch (error) {
    throw new Error("Failed to get user data");
  }
};

export const makeUpdatetUserRequest = async (userData: UpdateUserData) => {
  try {
    const headers = {headers: authHeader.getAuthHeader()};
    return (
      await axios.post(
        `http://localhost:8000/api/updateUserProfile`,
        userData,
        headers
      )
    ).data;
  } catch (error) {
    throw new Error("Failed to register user");
  }
};
```

Finally, create a file in `src/services/authHeader.ts`, where we shall have the logic for storing and retreving user token. The token gets stored after user login and gets retrieved and sent along with requests which need authetication.

```ts
import { USER_TOKEN_KEY } from "./authService";

const authHeader = {
  accessToken: '',

  initializeToken: (accessToken: string) => {
    authHeader.accessToken = accessToken;
  },

  getAuthHeader: () => {
    if (!authHeader.accessToken) {
      const token = localStorage.getItem(USER_TOKEN_KEY);

      if (token) {
        authHeader.initializeToken(token);
      }
    }

    if (authHeader.accessToken) {
      return {Authorization: 'Bearer ' + authHeader.accessToken};
    } else {
      return {};
    }
  },
};

export default authHeader;

```

Let's setup the redux react toolkit reducers which will combine and setup the slices above. Create a file in `src/store/index.ts`

```ts
import { combineReducers, configureStore } from "@reduxjs/toolkit";
import authReducer from "./slices/authReducer";
import userReducer from "./slices/userReducer";
import { useDispatch, useSelector, useStore } from "react-redux";

const rootReducer = combineReducers({
  auth: authReducer,
  user: userReducer,
});

export const store = configureStore({
  reducer: rootReducer,
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
export type AppStore = typeof store;

export const useAppDispatch = useDispatch.withTypes<AppDispatch>();
export const useAppSelector = useSelector.withTypes<RootState>();
export const useAppStore = useStore.withTypes<AppStore>();
```

In the above, we are combining `userSlice` reducer and `authSlice` reducer into one which will be used to initialise the store. The line after `useAppDispatch`, `useAppSelector` and `useAppStore` are just some syntactic sugar setups so that typescript can easily recognize our entities. In short, `useAppDIspatch` is to be used in place of `useDispatch`, `useAppSelector` in place of `useSelector` and `useAppStore` in place of `useStore`. If we did not set that up, we would have to repeat a lot of code whenever we want to use those if we wanted to get code typescript hinting.

Lets look inside of `App.tsx`, modify it like below by adding the store to the main `App` component

Import our `store` and `Provider` at the top:

```tsx
import { Provider } from "react-redux";
import { store } from "./store";
```

Then wrap the remaining code inside the `Provider` like below:

```tsx
<Provider store={store}>
    <BrowserRouter>
    <Link to={"login"}>Login</Link><> </>
    <Link to={"register"}>Register</Link><> </>
    <Link to={"profile"}>Profile</Link>
    <Routes>
        <Route path="/" element={<LoginComponent />} />
        <Route path="/login" element={<LoginComponent />} />
        <Route path="/register" element={<RegisterComponent />} />
        <Route path="/profile" element={<UserProfileComponent />} />
    </Routes>
    </BrowserRouter>
</Provider>
```

## Backend

### Install backend composer packages, setup database environment, user model and setup cors

In the terminal run `cd backend-folder` and run `composer install`

Then install the `Sanctum` package using `php artisan install:api`. This adds a new package, `"laravel/sanctum": "^4.0",` in `composer.json` inside the `require` key.

Answer no when asked if you want to run the database migration. Inside the user model `app/Models/User.php`, add the trait `Laravel\Sanctum\HasApiTokens`:

This package is used to issue user tokens upon login and is used to protect routes which need authentication to access.

After installing sanctum, run `php artisan config:publish cors` to publish the `cors.php` config file, which will create a new file in `config/cors.php`.

Create a file inside the same backend folder `.env` and modify the follwing variables:

```env
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=name-of-database
DB_USERNAME=database-server-username
DB_PASSWORD=database-server-password
```

Make sure you have mysql installed, running. Also enure the database set in the variable `DB_DATABASE` has been created.

At the tip of the model import the class :

```php
use Laravel\Sanctum\HasApiTokens;
```

and use it as seen below:

```php
use HasFactory, HasApiTokens, Notifiable;
```

Modify the `$fillable` property and add these two columns so we can update those as well:

```php
'phone',
'address',
```

### Create Routes and Controllers for login and registration

Inside the file `routes/api.php` add the following routes at the end:

```php
Route::post('/login', [AuthController::class, 'login']);
Route::post('/register', [AuthController::class, 'register']);
```

and import the AuthController `use App\Http\Controllers\AuthController;`

Now create the AuthController in `app/Http/Controllers/AuthController.php` with the content:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use App\Models\User;
use Illuminate\Support\Facades\Hash;

class AuthController extends Controller
{
    public function login(Request $request)
    {
        $credentials = $request->only('email', 'password');

        if (Auth::attempt($credentials)) {
            $user = Auth::user();
            $token = $user->createToken('auth_token')->plainTextToken;

            return response()->json($token);
        }

        return response()->json(['message' => 'Unauthorized'], 401);
    }

    public function register(Request $request)
    {
        $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|string|email|max:255|unique:users',
            'password' => 'required|string|min:8',
        ]);

        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => Hash::make($request->password),
        ]);

        return response()->json(['message' => 'User registered successfully'], 201);
    }
}

```

### Create Routes and Controllers for updating and getting user data

Add these routes to `routes/api.php`:

```php
Route::middleware('auth:sanctum')->group(function () {
    Route::get('getUserData', [UserProfileController::class, 'user']);
    Route::post('updateUserProfile', [UserProfileController::class, 'updateProfile']);
});
```

This line `Route::middleware('auth:sanctum')->group(function () {` is meant to add a protection middleware group so that the routes are only accessibly by a logged in user who provides a valid token. This is made possible by the sanctum package we just installed.

Import the controller at the top `use App\Http\Controllers\UserProfileController;`:

Add a controller file in `app/Http/Controllers/UserProfileController.php` with the content:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Illuminate\Support\Facades\Auth;

class UserProfileController extends Controller
{
    public function updateProfile(Request $request)
    {
        $request->validate([
            'name' => 'required|string|max:255',
            'phone' => ['required', 'string'],
            'address' => ['required', 'string'],
        ]);

        $userData = [
            'name' => $request->input('name'),
            'phone' => $request->input('phone'),
            'address' => $request->input('address')
        ];

        $user = Auth::user();
        $updated = $user->update($userData);

        if (!$updated) {
            return response('Could not update profile', Response::HTTP_BAD_REQUEST);
        }

        return response($user->only('id', 'name', 'phone', 'address'), 200);
    }

    public function user()
    {
        if (Auth::check()) {
            $user = Auth::user();
            return response($user->only('id', 'name', 'phone', 'address'), 200);
        }

        return response()->json(['message' => 'Unauthorized'], 401);
    }
}

```

### Update user migration file to add extra fields

Inside ther migration file `0001_01_01_000000_create_users_table.php`

After the line `$table->string('email')->unique();`

add the following:

```php
$table->string('phone')->nullable();
$table->text('address');
```

In the backend terminal, run `php artisan migrate` to create the database tables and columns.

Finally, run php artisan serve to run the laravel php backend server Ensure you have atleast `php 8.2` installed on yur local system. To verify that the backend is running, visit `http://localhost:8000/` in your browser and yu shall see some beautiful UI with some links made by laravel authors.

### Update login, register and userprofile components

Lets move into the `RegisterComponent.tsx` and modify it as below:

```tsx
import React, { useState } from "react";
import "./RegisterComponent.css";
import { registerUser } from "../store/slices/authReducer";
import { useNavigate } from "react-router-dom";
import { useAppDispatch } from "../store";

const RegisterComponent = () => {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const dispatch = useAppDispatch();
  const naivgate = useNavigate();

  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();

    dispatch(registerUser({ name, email, password }))
      .unwrap()
      .then(() => {
        naivgate("/login");
      })
      .catch((response) => {
        alert(`Error occured ${response.message}`);
      });
  };

  const onPropertyChage = (
    callback: typeof setName | typeof setEmail | typeof setPassword
  ) => {
    return (event: React.ChangeEvent<HTMLInputElement>): void => {
      callback(event.target.value);
    };
  };

  return (
    <div>
      <h2>Register</h2>
      <form onSubmit={handleSubmit} className="register-form">
        <label htmlFor="name">
          <input onChange={onPropertyChage(setName)} type="text" placeholder="name" id="name" />
        </label>
        <label htmlFor="email">
          <input onChange={onPropertyChage(setEmail)} type="text" placeholder="email" id="email" />
        </label>
        <label htmlFor="password">
          <input onChange={onPropertyChage(setPassword)} type="password" placeholder="password" id="password" />
        </label>
        <button>Register</button>
      </form>
    </div>
  );
};

export default RegisterComponent;
```

What's changed is that we added handlers for when the form is sbmitted a nd when the user changes the input fields, we update the corresponding local component states. We also made a call to the reducer method registerUser so we make a backend call to the server.

Modify the `LoginComponent.tsx` like so:

```tsx
import React, { useState } from "react";
import "./LoginComponent.css";
import { useAppDispatch } from "../store";
import { loginUser } from "../store/slices/authReducer";
import { useNavigate } from "react-router-dom";

const LoginComponent = () => {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const dispatch = useAppDispatch();
  const naivgate = useNavigate();

  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    dispatch(loginUser({email: username, password})).unwrap().then(() => {
        naivgate("/profile");
    }).catch((response) => {
        alert(`Error occured ${response.message}`);
    });
  };

  const onPropertyChage = (callback: typeof setUsername | typeof setPassword) => {
    return (event: React.ChangeEvent<HTMLInputElement>): void => {
        callback(event.target.value);
    }
  }

  return (
    <div>
      <h2>Login</h2>
      <form onSubmit={handleSubmit} className="login-form">
        <label htmlFor="username">
          <input onChange={onPropertyChage(setUsername)} type="text" placeholder="username" id="username" />
        </label>
        <label htmlFor="password">
          <input onChange={onPropertyChage(setPassword)} type="password" placeholder="password" id="password" />
        </label>
        <button>Login</button>
      </form>
    </div>
  );
};

export default LoginComponent;
```

In here also, we are doing similar actions as we have done for the register component, but upon successfull login, we are redirected to the userprofile screen.

Let us now modify the `UserProfileComponent.tsx`:

```tsx
import React, { useEffect, useState } from "react";
import "./UserProfileComponent.css";
import { useAppDispatch } from "../store";
import { getUser, updateUser } from "../store/slices/userReducer";

const UserProfileComponent = () => {
  const dispatch = useAppDispatch();
    const [name, setName] = useState('');
    const [phone, setPhone] = useState('');
    const [address, setAddress] = useState('');

  useEffect(() => {
    dispatch(getUser())
      .unwrap()
      .then((response) => {
        setName(response.name)
        setPhone(response?.phone || '')
        setAddress(response?.address || '')
      })
      .catch((response) => {
        alert(`Error occured ${response.message}`);
      });
  }, [dispatch]);

  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    dispatch(updateUser({name, phone, address})).unwrap().then(() => {
        console.log('Updated successfuly');
    }).catch((response) => {
        alert(`Error occured ${response.message}`);
    });
  };

  const onPropertyChage = (callback: typeof setName | typeof setPhone | typeof setAddress) => {
    return (event: React.ChangeEvent<HTMLInputElement|HTMLTextAreaElement>): void => {
        callback(event.target.value);
    }
  }

  return (
    <div>
      <h2>Profile</h2>
      <form onSubmit={handleSubmit} className="update-profile-form">
        <label htmlFor="name">
          <input value={name} onChange={onPropertyChage(setName)} type="text" placeholder="name" id="name" />
        </label>
        <label htmlFor="phone">
          <input value={phone} onChange={onPropertyChage(setPhone)} type="text" placeholder="phone" id="phone" />
        </label>
        <label htmlFor="address">
          <textarea value={address} onChange={onPropertyChage(setAddress)} id="address" placeholder="address"></textarea>
        </label>
        <button>Update Profie</button>
      </form>
    </div>
  );
};

export default UserProfileComponent;
```

What happens here is that, we load the user data once, when the component is mounted, then we set the states for name, phone and address which are tied to the proper input fields. Once the data is returned from the backend, the field will become automatically pupulated. We can change the field values, and submit the data which will send them to teh backend to be updated for the user.

## Add feature to Logout users

### Backend logout Logic

In the backend file `routes/api.php`, add the line:

```php
Route::post('/logout', [AuthController::class, 'logout']);
```

in the protected route group, so that only loggedin users can access it to logout and destroy the user session and delete the token.

Add the method in te `AuthController`.

```php
public function logout(Request $request)
{
    $request->user()->currentAccessToken()->delete();
    return response()->json(['message' => 'Logged out successfully']);
}
```

### Frontend logout Logic

In the frontend, in `App.tsx` add this line to the list of routes

```tsx
<Route path="/logout" element={<LogoutComponent />} />
```

and add the following to the links section:

```tsx
<> </><Link to={"logout"}>Logout</Link>
```

and import the component:

```tsx
import LogoutFLogoutComponentorm from "./components/LogoutComponent";
```

Create a component for logout `src/components/LogoutComponent.tsx`

```tsx
import React from "react";
import { logoutUser } from "../store/slices/authReducer";
import { useAppDispatch } from "../store";
import { useNavigate } from "react-router-dom";

const LogoutComponent = () => {
  const dispatch = useAppDispatch();
  const navigate = useNavigate();

  const handleAcceptLogout = () => {
    dispatch(logoutUser())
      .unwrap().then(() => {
        navigate("/login");
      })
      .catch((response) => {
        alert(`Error occured ${response.message}`);
      });
  };

  const handleCancelLogout = () => {
    navigate("/profile");
  };

  return (
    <div>
      <h2>Logout</h2>
      <h2>Are you sure you want to logout?</h2>
      <button onClick={handleAcceptLogout}>Yes</button>
      <> </>
      <button onClick={handleCancelLogout}>No</button>
    </div>
  );
};

export default LogoutComponent;
```

The logou component show two buttons, one to confirm logout, which will send an api request. The second to cancel the logout and send us back to the profie screen.

Inside of `src/store/slices/authReducer.ts` add this:

```ts
export const logoutUser = createAsyncThunk<void, void>(
  "auth/logoutUser",
  async () => {
    return await makeLogoutUserRequest();
  }
);
```

and add along the followibg cases to extra reducers:

```ts
.addCase(logoutUser.pending, (state) => {
    state.loading = true;
    state.error = null;
})
.addCase(logoutUser.fulfilled, (state) => {
    state.loading = false;
    state.token = null;
})
.addCase(logoutUser.rejected, (state, action) => {
    state.loading = false;
    state.error = action.error.message || "Failed to logout user";
})
```

Next, inside of `src/services/authService.ts`, add:

```ts
export const makeLogoutUserRequest = async () => {
  try {
    const headers = {headers: authHeader.getAuthHeader()};
    await axios.post(`http://localhost:8000/api/logout`, null, headers);

    authHeader.clearToken();

    localStorage.removeItem(USER_TOKEN_KEY);
  } catch (error) {
    throw new Error("Failed to loglogoutin user: service");
  }
};
```

This sends a request to the laravel backend api. The backend deletes the user tokens. When that succeeds, the frontend clears the localstorage key which held the token, and the `authHeader` config file is cleared from holding a copy of that token. We are then redirected to the login page.
