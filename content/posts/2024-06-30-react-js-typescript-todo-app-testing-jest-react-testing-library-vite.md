---
title: React.js Typescript Todo App Testing with Jest and React Testing Library and Vite
description: ""
date: 2024-06-30T14:24:14.000Z
preview: ""
draft: false
tags: []
categories:
  - React.js
  - Testing
hero: /images/posts/2024-06-30-react-js-typescript-todo-app-testing-jest-react-testing-library-vite.png
type: posts
slug: react-js-typescript-todo-app-testing-jest-react-testing-library-vite
---

#### Introduction

In this tutorial, we shall setup [Jest](https://jestjs.io/) and [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/) to perform integration test. We will be testing React.Js components, so this will not be an end to test, but we will be testing the individual coponents involved in the Todo App. At each stage, we will render the components invovled using the library functions provided by React Testing Library. We shall pass in properties to each component and test their behavious and visualisation by simulating clicks and text typing into textboxes using the appropriate functions. Jest will be used as our test runner. These two libraries are independent but work very well together. So let's get started.

#### Initializing Vite Project

First, let's initialize a new Vite project with React and TypeScript templates.

```bash
npm create vite@latest jest-todo-app-test --template react-ts
cd jest-todo-app-test
npm install
```

#### Setting Up Jest
Next, we need to install Jest and its related dependencies:

```bash
npm install --save-dev jest @testing-library/react @testing-library/jest-dom @types/jest ts-jest babel-jest jest-environment-jsdom @babel/preset-env @babel/preset-react @babel/preset-typescript
```

* `jest`: Jest testing framework.
* `@testing-library/react`: React Testing Library for testing React components.
* `@testing-library/jest-dom`: Custom Jest matchers for testing DOM nodes.
* `@types/jest`: TypeScript definitions for Jest.
* `ts-jest`: TypeScript preprocessor for Jest.
* `babel-jest`: Babel transformer for Jest.

To run the test globally install it globally like:

```
npm install jest --global
```

Create a Jest configuration file (jest.config.js) in the root of your project:

```js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'jsdom',
  moduleNameMapper: {
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
  },
  setupFilesAfterEnv: ['./src/setupTests.ts'],
  transform: {
    '^.+\\.(js|jsx|ts|tsx)$': 'babel-jest',
  },
};
```

* `preset: 'ts-jest'`: Use ts-jest preset for TypeScript.
* `testEnvironment: 'jsdom'`: Use jsdom environment for simulating browser behavior in tests.
* `moduleNameMapper`: Map CSS imports to identity-obj-proxy to handle CSS modules in tests.
* `setupFilesAfterEnv`: Specify setup files to be run after the test framework is installed.
* `transform`: Use babel-jest to transform JavaScript and TypeScript files.

Create a Babel configuration file (babel.config.js) in the root of your project:

```js
module.exports = {
  presets: [
    '@babel/preset-env',
    '@babel/preset-react',
    '@babel/preset-typescript',
  ],
};
```

* `@babel/preset-env`: Transpile modern JavaScript to be compatible with older environments.
* `@babel/preset-react`: Transpile JSX to JavaScript.
* `@babel/preset-typescript`: Transpile/converts TypeScript to JavaScript.

Create a setupTests.ts file in the src directory:

```bash
import '@testing-library/jest-dom';
```

#### Writing the Todo App
Let's create a simple Todo app. First, create the necessary components and types.

`src/types.ts`

```ts
export interface Todo {
  id: number;
  text: string;
  completed: boolean;
}
```

This is a typescript interface to represent a single todo

Create a todo item in `src/components/TodoItem.tsx`
```tsx
import React, { useState } from "react";
import { Todo } from "../types";

interface TodoItemProps {
  todo: Todo;
  toggleTodo: (id: number) => void;
  deleteTodo: (id: number) => void;
  editTodo: (id: number, text: string) => void;
}

const TodoItem: React.FC<TodoItemProps> = ({
  todo,
  toggleTodo,
  deleteTodo,
  editTodo,
}) => {
  const [isEditing, setIsEditing] = useState(false);
  const [newText, setNewText] = useState(todo.text);

  const handleEdit = () => {
    setIsEditing(true);
  };

  const handleSave = () => {
    editTodo(todo.id, newText);
    setIsEditing(false);
  };

  return (
    <li>
      {isEditing ? (
        <input
          type="text"
          value={newText}
          onChange={(e) => setNewText(e.target.value)}
        />
      ) : (
        <span
          style={{ textDecoration: todo.completed ? "line-through" : "none" }}
          onClick={() => toggleTodo(todo.id)}
        >
          {todo.text}
        </span>
      )}
      {isEditing ? (
        <button onClick={handleSave}>Save</button>
      ) : (
        <button onClick={handleEdit}>Edit</button>
      )}
      <button onClick={() => deleteTodo(todo.id)}>Delete</button>
    </li>
  );
};

export default TodoItem;
```

Then create a todos list component in `src/components/TodoList.tsx`

```tsx
import React from "react";
import { Todo } from "../types";
import TodoItem from "./TodoItem";

interface TodoListProps {
  todos: Todo[];
  toggleTodo: (id: number) => void;
  deleteTodo: (id: number) => void;
  editTodo: (id: number, text: string) => void;
}

const TodoList: React.FC<TodoListProps> = ({
  todos,
  toggleTodo,
  deleteTodo,
  editTodo,
}) => {
  return (
    <ul>
      {todos.map((todo) => (
        <TodoItem
          key={todo.id}
          todo={todo}
          toggleTodo={toggleTodo}
          deleteTodo={deleteTodo}
          editTodo={editTodo}
        />
      ))}
    </ul>
  );
};

export default TodoList;
```

Create a component for testing `src/components/AddTodo.tsx`

```tsx
import React, { useState } from "react";

interface AddTodoProps {
  addTodo: (text: string) => void;
}

const AddTodo: React.FC<AddTodoProps> = ({ addTodo }) => {
  const [text, setText] = useState("");

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (!text.trim()) return;
    addTodo(text);
    setText("");
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <button type="submit">Add Todo</button>
    </form>
  );
};

export default AddTodo;
```

Now modify the main app file in `src/App.tsx`

```tsx
import React, { useState } from "react";
import { Todo } from "./types";
import TodoList from "./components/TodoList";
import AddTodo from "./components/AddTodo";

let nextId = 0;

const App: React.FC = () => {
  const [todos, setTodos] = useState<Todo[]>([]);

  const addTodo = (text: string) => {
    setTodos([...todos, { id: nextId++, text, completed: false }]);
  };

  const toggleTodo = (id: number) => {
    setTodos(
      todos.map((todo) =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  };

  const deleteTodo = (id: number) => {
    setTodos(todos.filter((todo) => todo.id !== id));
  };

  const editTodo = (id: number, text: string) => {
    setTodos(todos.map((todo) => (todo.id === id ? { ...todo, text } : todo)));
  };

  return (
    <div>
      <h1>Todo List</h1>
      <AddTodo addTodo={addTodo} />
      <TodoList
        todos={todos}
        toggleTodo={toggleTodo}
        deleteTodo={deleteTodo}
        editTodo={editTodo}
      />
    </div>
  );
};

export default App;
```

#### Writing Tests
Now let's write tests for our components and logic.

`src/components/TodoItem.test.tsx`

```tsx
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import TodoItem from './TodoItem';
import { Todo } from '../types';

test('renders a todo item and toggles it', () => {
  const todo: Todo = { id: 1, text: 'Learn Jest', completed: false };
  const toggleTodo = jest.fn();

  render(<TodoItem todo={todo} toggleTodo={toggleTodo} />);
  const todoItem = screen.getByText(/learn jest/i);
  expect(todoItem).toBeInTheDocument();
  expect(todoItem).toHaveStyle('text-decoration: none');

  fireEvent.click(todoItem);
  expect(toggleTodo).toHaveBeenCalledWith(todo.id);
});

```

* `render`: Render the component.
* `screen.getByText`: Select an element by its text content.
* `expect`: Assertion library provided by Jest.
* `fireEvent.click`: Simulate a click event.

`src/components/TodoList.test.tsx`

```tsx
import React from "react";
import { render, screen, fireEvent } from "@testing-library/react";
import TodoList from "./TodoList";
import { Todo } from "../types";

test("renders a list of todos and toggles them", () => {
  //Here, in the todoslist, we have two todos, we expect that the functions passed in to the component,
  // gets called with the correct id which corresponds to the id of the clicked todo.
  const todos: Todo[] = [
    { id: 1, text: "Learn Jest", completed: false },
    { id: 2, text: "Learn React Testing Library", completed: false },
  ];
  const toggleTodo = jest.fn();
  const deleteTodo = jest.fn();
  const editTodo = jest.fn();

  render(<TodoList todos={todos} toggleTodo={toggleTodo} deleteTodo={deleteTodo} editTodo={editTodo} />);
  const todoItems1 = screen.getByText(todos[0].text);
  const todoItems2 = screen.getByText(todos[1].text);
  const todoItems = screen.getAllByRole("listitem");
  expect(todoItems).toHaveLength(2);

  fireEvent.click(todoItems1);
  expect(toggleTodo).toHaveBeenCalledWith(1);

  fireEvent.click(todoItems2);
  expect(toggleTodo).toHaveBeenCalledWith(2);
});
```

`src/components/AddTodo.test.tsx`

```tsx
import React from "react";
import { render, screen, fireEvent } from "@testing-library/react";
import AddTodo from "./AddTodo";

test("adds a new todo item", () => {
  // Here in the AddTodo component,
  // we simulate etering text into the input field
  // Then we simulate a click on the add todo button
  // After that, we will assert/check if the addTodo function we
  // passed a a prop to the component is called with the
  // argument text containing the text which was entered into the text input
  const addTodo = jest.fn();

  render(<AddTodo addTodo={addTodo} />);
  const input = screen.getByRole("textbox");
  const button = screen.getByRole("button", { name: /add todo/i });

  fireEvent.change(input, { target: { value: "Learn Jest" } });
  fireEvent.click(button);

  expect(addTodo).toHaveBeenCalledWith("Learn Jest");
  expect(input).toHaveValue("");
});
```

`src/App.test.tsx`

```tsx
import React from "react";
import { render, screen, fireEvent } from "@testing-library/react";
import App from "./App";

// Testing the App Component
// This tests simulates entering text in the add text box
// then trigger a click on the add todo button
// We then expect a new todo to appear with the text we used
//to type into the textbox
test("adds and toggles todo items", () => {
  render(<App />);
  const input = screen.getByRole("textbox");
  const button = screen.getByRole("button", { name: /add todo/i });

  fireEvent.change(input, { target: { value: "Learn Jest" } });
  fireEvent.click(button);

  const todoItem = screen.getByText(/learn jest/i);
  expect(todoItem).toBeInTheDocument();
  expect(todoItem).toHaveStyle("text-decoration: none");

  fireEvent.click(todoItem);
  expect(todoItem).toHaveStyle("text-decoration: line-through");
});

// Here, we create a todo in the list using the simulation as done above,
// But instead of just check ing for its existence, we will trigger a
// click on it's delete button
// Afterwards, we will assert that the todo no longer exists in the document
test('deletes a todo item', () => {
  render(<App />);
  const input = screen.getByRole('textbox');
  const button = screen.getByRole('button', { name: /add todo/i });

  fireEvent.change(input, { target: { value: 'Learn Jest' } });
  fireEvent.click(button);

  const deleteButton = screen.getByText(/delete/i);
  fireEvent.click(deleteButton);

  expect(screen.queryByText(/learn jest/i)).not.toBeInTheDocument();
});

// Here, we also simulate creating a todo
// then we wll change the text using the change function
// Then we will click on the edit button, which will move the todo
// into edit mode
// Upon changing the text, we will trigger the save button which should
// update the todos text in the list
// We will then confirm/assert that the newly entered text is in the document
test('edits a todo item', () => {
  render(<App />);
  const input = screen.getByRole('textbox');
  const button = screen.getByRole('button', { name: /add todo/i });

  fireEvent.change(input, { target: { value: 'Learn Jest' } });
  fireEvent.click(button);

  const editButton = screen.getByText(/edit/i);
  fireEvent.click(editButton);

  const editInput = screen.getByDisplayValue(/learn jest/i);
  fireEvent.change(editInput, { target: { value: 'Learn React Testing' } });
  const saveButton = screen.getByText(/save/i);
  fireEvent.click(saveButton);

  expect(screen.getByText(/learn react testing/i)).toBeInTheDocument();
});
```

* `screen.getByRole`: Select an element by its role, for example button or checkbox.
* `screen.queryByText`: Select an element by its text content, returning null if not found.

#### Running Tests

To run the tests, add a script to your package.json:

```bash
"scripts": {
  "test": "jest"
}
```

Now you can run your tests using:

```bash
npm test
```

This runs all tests

To run an individual test use the globally installed one on the commandline like:

```
jest src/components/TodoItem.test.tsx
```

But replace the file above with our choice of test file

#### Possible Error

If you get errors about `babel.config.js` and or `jest.config.js`, rename both files and change their file endings to `babel.config.cjs` and `jest.config.cjs` respectfully.

### Conclusion
This tutorial covers setting up Jest for a React.js application using TypeScript and Vite, along with writing tests for various components and the main application logic. This setup ensures that your application is well-tested and reliable.