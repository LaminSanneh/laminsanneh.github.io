---
title: Setting Up a React.js Webpack and Typescript App From Scratch using Todo App
description: ""
date: 2024-07-22T11:30:00.000Z
preview: ""
draft: false
tags: []
categories:
  - React.js
hero: /images/posts/2024-07-19-setting-up-a-react.js-webpack-and-typescript-app-from-scratch-using-todo-app.png
type: posts
---

### Introduction
In this tutorial, we'll set up a React.js Todo application using TypeScript and Webpack from scratch. This will include configuring Webpack to handle TypeScript and React code, setting up a development server, and building our project for production.

### Prerequisites

* Node.js and npm installed on your machine.
* Basic understanding of React.js, TypeScript, and Webpack.

### Project Setup
First, let's create a new directory for our project and initialize a new Node.js project.

```bash
mkdir react-ts-todo
cd react-ts-todo
npm init -y
```

### Installing Dependencies
We'll need to install React, React DOM, TypeScript, and Webpack along with some other necessary packages.

```
npm install react react-dom
npm install --save-dev typescript ts-loader webpack webpack-cli webpack-dev-server html-webpack-plugin @types/react @types/react-dom
```

### Configuring TypeScript
Create a `tsconfig.json` file in the root of your project to configure TypeScript.

```json
{
  "compilerOptions": {
    "target": "ES5",
    "module": "commonjs",
    "lib": ["dom", "es2015"],
    "jsx": "react",
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

### Configuring Webpack
Create a `webpack.config.js` file in the root of your project. This file will define how Webpack processes our files.

```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  entry: "./src/index.tsx",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist"),
  },
  resolve: {
    extensions: [".ts", ".tsx", ".js"],
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: "ts-loader",
        exclude: /node_modules/,
      },
    ],
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: "./src/index.html",
    }),
  ],
  devServer: {
    static: {
      directory: path.join(__dirname, "dist"),
    },
    compress: true,
    port: 9001,
  },
  mode: "development",
};
```

### Creating the Source Files
Let's create the necessary source files and directories.

```bash
mkdir src
touch src/index.tsx src/App.tsx src/index.html
```

### Writing the React Application

`src/index.html`

Create an HTML template to serve our React application.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>React TypeScript Todo App</title>
</head>
<body>
  <div id="root"></div>
</body>
</html>
```

`src/index.tsx`

This is the entry point of our React application.

```tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')!).render(
    <React.StrictMode>
      <App />
    </React.StrictMode>,
  )
```

`src/App.tsx`
Create a simple Todo app in this file.

```tsx
import React, { useState } from 'react';

interface Todo {
  text: string;
  completed: boolean;
}

const App: React.FC = () => {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [newTodo, setNewTodo] = useState<string>('');

  const addTodo = () => {
    if (newTodo.trim() !== '') {
      setTodos([...todos, { text: newTodo, completed: false }]);
      setNewTodo('');
    }
  };

  const toggleTodo = (index: number) => {
    const newTodos = todos.slice();
    newTodos[index].completed = !newTodos[index].completed;
    setTodos(newTodos);
  };

  return (
    <div>
      <h1>Todo App</h1>
      <input
        type="text"
        value={newTodo}
        onChange={(e) => setNewTodo(e.target.value)}
      />
      <button onClick={addTodo}>Add Todo</button>
      <ul>
        {todos.map((todo, index) => (
          <li
            key={index}
            style={{
              textDecoration: todo.completed ? 'line-through' : 'none'
            }}
          >
            <span onClick={() => toggleTodo(index)}>{todo.text}</span>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default App;
```

### Running the Development Server
Now, we can run our development server using Webpack Dev Server.

Add a script in `package.json` to start the server:

```json
"scripts": {
  "start": "webpack serve --open"
}
```

Run the development server:

```bash
npm start
```

This should open your browser and display your React TypeScript Todo app running on http://localhost:9001.


### Building for Production

Install the following package whch clears the production build folder before each build.

```bash
npm install --save-dev clean-webpack-plugin
```

To build our project for production, we need to add a production configuration to our Webpack setup. Modify `webpack.config.js` to the following:


```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = (env, argv) => {
  const isProduction = argv.mode === 'production';

  return {
    entry: './src/index.tsx',
    output: {
      filename: 'bundle.[contenthash].js',
      path: path.resolve(__dirname, 'dist')
    },
    resolve: {
      extensions: ['.ts', '.tsx', '.js']
    },
    module: {
      rules: [
        {
          test: /\.tsx?$/,
          use: 'ts-loader',
          exclude: /node_modules/
        }
      ]
    },
    plugins: [
      new CleanWebpackPlugin(),
      new HtmlWebpackPlugin({
        template: './src/index.html'
      })
    ],
    devServer: {
      static: {
        directory: path.join(__dirname, "dist"),
      },
      compress: true,
      port: 9001,
    },
    optimization: {
      splitChunks: {
        chunks: 'all',
      },
    },
    mode: isProduction ? 'production' : 'development'
  };
};
```

To build for production, add the following to `package.json` scripts

```json
"build": "webpack --mode production"
```


and run:

```bash
npm run build
```

This will generate the production build in the `dist` folder. If you look at the generated files in that
directory, you will see minified versions of the files in the src folder.

### Adding CSS Support
To add CSS support to our project, we can modify our Webpack configuration to handle CSS files using `style-loader` and `css-loader`. These two loaders makes sure the dev server loads the css inside our html file during development.

We also need the plugin `MiniCssExtractPlugin`. This plugin extracts CSS into separate files, which allows for better caching and performance. This plugin is needed for production code. We shall configure our webpack config file to use one set or the other based on whether we are in development mode or production.

#### Installing Required Packages

Install the necessary loaders:

```bash
npm install --save-dev style-loader css-loader mini-css-extract-plugin
```

#### Modifying `webpack.config.js`

Update `webpack.config.js` to handle CSS files:

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = (env, argv) => {
  const isProduction = argv.mode === 'production';

  return {
    entry: './src/index.tsx',
    output: {
      filename: 'bundle.[contenthash].js',
      path: path.resolve(__dirname, 'dist')
    },
    resolve: {
      extensions: ['.ts', '.tsx', '.js']
    },
    module: {
      rules: [
        {
          test: /\.tsx?$/,
          use: 'ts-loader',
          exclude: /node_modules/
        },
        {
          test: /\.css$/i,
          use: [
            isProduction ? MiniCssExtractPlugin.loader : 'style-loader',
            'css-loader'
          ]
        }
      ]
    },
    plugins: [
      new CleanWebpackPlugin(),
      new HtmlWebpackPlugin({
        template: './src/index.html'
      }),
      new MiniCssExtractPlugin({
        filename: '[name].[contenthash].css'
      })
    ],
    devServer: {
      contentBase: path.join(__dirname, 'dist'),
      compress: true,
      port: 9000
    },
    optimization: {
      splitChunks: {
        chunks: 'all',
      },
    },
    mode: isProduction ? 'production' : 'development'
  };
};
```

If you notice the naming pattern used for the output css file for production `[name].[contenthash].css`.
The output file will be using the original filename combined with a dynamic hash for cache-busting and the file type suffix.

We could use a similar pattern for the main javascript outputs. So in the output part:

```js
output: {
      filename: 'bundle.[contenthash].js',
      path: path.resolve(__dirname, 'dist')
    },
```

we could use:

```js
output: {
      filename: '[name].[contenthash].js',
      path: path.resolve(__dirname, 'dist')
    },
```

but that is up to you.

#### Using CSS in Your Components
Now you can create CSS files and import them into your components. Create for instance, a styles.css file inside your src directory:

```css
body {
  font-family: Arial, sans-serif;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  margin-bottom: 0.5rem;
}

input[type="text"] {
  padding: 0.5rem;
  font-size: 1rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  padding: 0.5rem 1rem;
  font-size: 1rem;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}
```

Import `styles.css` in your `App.tsx` at the top:

```tsx
import React, { useState } from 'react';
import './styles.css';
```

### Conclusion
In this tutorial, we set up a React.js Todo application with TypeScript and Webpack from scratch. We configured Webpack to handle TypeScript and React code, set up a development server, and built our project for production. This setup can serve as a solid foundation for more complex React applications using TypeScript and Webpack.

I wrote about webpack for another blog a while back, which is a little more indepth, but may need some updating. However, there are still some insights to gain from it over here https://jscrambler.com/blog/easy-custom-webpack-setup-for-react-js-applications
