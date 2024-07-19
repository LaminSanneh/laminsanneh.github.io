---
title: Playwright and React.js Typescript Todo App Automated End to End Testing with Vite
description: ""
date: 2024-07-09T20:22:16.664Z
preview: ""
draft: false
tags: []
categories:
  - React.js
  - Testing
hero: /images/posts/2024-07-09-playwright-react-js-typescript-todo-app-automated-testing-vite.png
type: posts
slug: playwright-react-js-typescript-todo-app-automated-testing-vite
---

### Introduction
In this tutorial, we'll create a simple Todo app using React.js and TypeScript, initialized with Vite, and then we'll write tests for it using [Playwright](https://playwright.dev/). We will use playwright to do an end to end test which will simulate the user browser interaction with our todo app like adding new todos, deleting todos, upadtinhg todos e.t.c. By the end of this guide, you'll have a solid understanding of how to use Playwright to test your React applications.

### Prerequisites

Before starting, ensure you have the following installed on your machine:

* Node.js (>= 14.x)
* npm or yarn
* Playwright

Excited? So let's get to it.

### Initialize the React App with Vite
First, let's create our React application using Vite.

#### Create the project:

```bash
npm create vite@latest todo-app-playwright-test --template react-ts
cd todo-app-playwright-test
```
<br/>

#### Install dependencies
```bash
npm install
```

<br/>

#### Start the development server:
```bash
npm run dev
```

Your new Vite project should now be running on `http://localhost:5173` if there are no other vite apps running on your system. Otherwise, observe the url in your console.

<br/>

### Build the Todo App

Next, let's build a simple Todo app. We'll create components for displaying the list of todos and adding new todos.

<br/>

#### Create a Todo interface:

Create a file named `types.ts` in the `src` directory:

```ts
export interface Todo {
  id: number;
  text: string;
  completed: boolean;
}
```

<br/>

#### Create the TodoItem component:

`src/components/TodoItem.tsx`

```tsx
import React from "react";
import { Todo } from "../types";

interface Props {
  todo: Todo;
  toggleTodo: (id: number) => void;
  deleteTodo: (id: number) => void;
}

const TodoItem: React.FC<Props> = ({ todo, toggleTodo, deleteTodo }) => {
  return (
    <li>
      <label>
        <input
          type="checkbox"
          checked={todo.completed}
          onChange={() => toggleTodo(todo.id)}
        />
        {todo.text}
      </label>
      <button onClick={() => deleteTodo(todo.id)}>Delete</button>
    </li>
  );
};

export default TodoItem;

```

<br/>

#### Create the TodoList component:

`src/components/TodoList.tsx`

```tsx
import React from "react";
import { Todo } from "../types";
import TodoItem from "./TodoItem";

interface Props {
  todos: Todo[];
  toggleTodo: (id: number) => void;
  deleteTodo: (id: number) => void;
}

const TodoList: React.FC<Props> = ({ todos, toggleTodo, deleteTodo }) => {
  return (
    <ul>
      {todos.map((todo) => (
        <TodoItem
          key={todo.id}
          todo={todo}
          toggleTodo={toggleTodo}
          deleteTodo={deleteTodo}
        />
      ))}
    </ul>
  );
};

export default TodoList;
```

<br/>

#### Create the AddTodo component:

`src/components/AddTodo.tsx`

```tsx
import React, { useState } from "react";

interface Props {
  addTodo: (text: string) => void;
}

const AddTodo: React.FC<Props> = ({ addTodo }) => {
  const [text, setText] = useState("");

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
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

<br/>

#### Update the main App component:

`src/App.tsx`

```tsx
import React, { useState } from "react";
import { Todo } from "./types";
import TodoList from "./components/TodoList";
import AddTodo from "./components/AddTodo";

const App: React.FC = () => {
  const [todos, setTodos] = useState<Todo[]>([]);

  const addTodo = (text: string) => {
    const newTodo: Todo = {
      id: todos.length + 1,
      text,
      completed: false,
    };
    setTodos([...todos, newTodo]);
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

  return (
    <div>
      <h1>Todo List</h1>
      <AddTodo addTodo={addTodo} />
      <TodoList todos={todos} toggleTodo={toggleTodo} deleteTodo={deleteTodo} />
    </div>
  );
};

export default App;
```

<br/>

#### Add Filter Todos Functionality

`src/components/Filter.tsx`

```tsx
import React from "react";

interface Props {
  filter: string;
  setFilter: (filter: string) => void;
}

const Filter: React.FC<Props> = ({ filter, setFilter }) => {
  return (
    <div>
      <button onClick={() => setFilter("all")} disabled={filter === "all"}>
        All
      </button>
      <button
        onClick={() => setFilter("completed")}
        disabled={filter === "completed"}
      >
        Completed
      </button>
      <button
        onClick={() => setFilter("active")}
        disabled={filter === "active"}
      >
        Active
      </button>
    </div>
  );
};

export default Filter;
```

<br/>

#### Update the App component to handle filtering todos:

`src/App.tsx`

```tsx
import React, { useState } from "react";
import { Todo } from "./types";
import TodoList from "./components/TodoList";
import AddTodo from "./components/AddTodo";
import Filter from "./components/Filter";

const App: React.FC = () => {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [filter, setFilter] = useState<string>("all");

  const addTodo = (text: string) => {
    const newTodo: Todo = {
      id: todos.length + 1,
      text,
      completed: false,
    };
    setTodos([...todos, newTodo]);
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

  const filteredTodos = todos.filter((todo) => {
    if (filter === "completed") {
      return todo.completed;
    }
    if (filter === "active") {
      return !todo.completed;
    }
    return true;
  });

  return (
    <div>
      <h1>Todo List</h1>
      <AddTodo addTodo={addTodo} />
      <Filter filter={filter} setFilter={setFilter} />
      <TodoList
        todos={filteredTodos}
        toggleTodo={toggleTodo}
        deleteTodo={deleteTodo}
      />
    </div>
  );
};

export default App;
```

### Setup Playwright

#### Install Playwright Testing Framework:

```bash
npm install --save-dev @playwright/test
```

#### Add Playwright scripts to package.json:
```json
"scripts": {
  "test:e2e": "playwright test",
  "test:e2e:headed": "playwright test --headed",
  "test:e2e:debug": "playwright test --debug"
}
```

This adds a script to run Playwright tests using the command `npm run test:e2e`.

#### Initialize Playwright:
```bash
npx playwright install
```

This will download some browser related libraries for chrome, firefox e.t.c.

#### Create Playwright configuration file:

Create a file named `playwright.config.ts` in the root directory:

```ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./tests",
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: "html",
  use: {
    baseURL: "http://localhost:5173",
    trace: "on-first-retry",
  },
  projects: [
    {
      name: "chromium",
      use: { ...devices["Desktop Chrome"] },
    },
    {
      name: "firefox",
      use: { ...devices["Desktop Firefox"] },
    },
  ],
});
```

### Write Playwright Tests

Let's write tests to ensure our Todo app works as expected.

#### Create a test file:

Create a directory named `tests` in the root of your project and add a file named todo.spec.ts:

`tests/todo.spec.ts`

```ts
import { test, expect } from '@playwright/test';

test.describe('Todo App', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test('should display the correct title', async ({ page }) => {
    // Here, the test tests that the title header 'Todo list' exists in the document
    await expect(page.locator('h1')).toHaveText('Todo List');
  });

  test('should add a new todo', async ({ page }) => {
    // In this test, we will fill the new todo input and submit the for,
    // tjis creates a new todo in the list
    //The test will check and confirm that  a new todo with the text entered above has
    //be added to the list of todos
    await page.fill('input[type="text"]', 'New Todo');
    await page.click('button[type="submit"]');
    await expect(page.locator('li')).toContainText('New Todo');
  });

  test('should toggle a todo', async ({ page }) => {
    // This test shows that whenever we click on a todo items checkbox
    // the state of the todo will change its state from active to completed
    // We will confirm the completed state by checking if the checkbox is checked
    // A second test is done by clicking on the checked checkbox again to move its
    // state from completed to active, and the next test will confirm that now
    // the checkbox is no longer checked
    await page.fill('input[type="text"]', 'Toggle Todo');
    await page.click('button[type="submit"]');
    const todoItem = page.locator('li').filter({ hasText: 'Toggle Todo' });
    const checkbox = todoItem.locator('input[type="checkbox"]');
    await checkbox.check();
    await expect(checkbox).toBeChecked();
    await checkbox.uncheck();
    await expect(checkbox).not.toBeChecked();
  });

  test('should delete a todo', async ({ page }) => {
    // This test tests that when we click the text of 'Delete Todo' next to a
    // todo item that todo will no longer be shown in the list by default
    // because it is deleted
    await page.fill('input[type="text"]', 'Delete Todo');
    await page.click('button[type="submit"]');
    const todoItem = page.locator('li').filter({ hasText: 'Delete Todo' });
    await todoItem.locator('button').click();
    await expect(todoItem).toHaveCount(0);
  });

  test('should filter todos', async ({ page }) => {
    // This tests that when we list todos, an active to and another one in progress
    // When we click the 'Completed Todo' button
    // Only the todo item which is completed is shown
    // When we click the button with 'Active Todo'
    // Only incomplete todos are shown
    // Finally, we wil click the All button and both completed and active todos will be shown
    // All actions adescribed show that the filter is working
    await page.fill('input[type="text"]', 'Completed Todo');
    await page.click('button[type="submit"]');
    await page.locator('li').filter({ hasText: 'Completed Todo' }).locator('input[type="checkbox"]').check();


    await page.fill('input[type="text"]', 'Active Todo');
    await page.click('button[type="submit"]');

    await page.click('button:has-text("Completed")');
    await expect(page.locator('li')).toHaveCount(1);
    await expect(page.locator('li')).toContainText('Completed Todo');

    await page.click('button:has-text("Active")');
    await expect(page.locator('li')).toHaveCount(1);
    await expect(page.locator('li')).toContainText('Active Todo');

    await page.click('button:has-text("All")');
    await expect(page.locator('li')).toHaveCount(2);
  });
});
```

* import { test, expect } from '@playwright/test';: Imports the test and expect functions from the Playwright testing library.
* test.describe('Todo App', () => { ... });: Defines a test suite named 'Todo App'.
* test.beforeEach(async ({ page }) => { await page.goto('/'); });: // This makes sure befire each test is run, we visit the home page `/`

#### Run the tests:

```bash
npm run test:e2e
```

This will run all the tests.

### Conclusion
We have successfully built a Todo app with additional features like deleting and filtering todos and tested these features using Playwright. This demonstrates the power and flexibility of Playwright in automating end-to-end testing for web applications.

Feel free to explore more features and write additional tests to further enhance your Todo app. Playwright's extensive API and robust support for multiple browsers make it an excellent tool for ensuring the quality and reliability of your web applications.
