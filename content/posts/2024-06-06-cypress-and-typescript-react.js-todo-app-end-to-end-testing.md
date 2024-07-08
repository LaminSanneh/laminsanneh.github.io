---
title: Cypress and TypeScript React.js Todo App End-to-End Testing
description: ""
date: 2024-06-06T09:42:58.000Z
preview: ""
draft: false
tags: []
categories:
  - React.js
type: posts
slug: cypress-typescript-react-js-todo-app-testing
hero: /images/posts/2024-06-06-cypress-and-typescript-react.js-todo-app-end-to-end-testing.png
---

### Introduction
In this tutorial, we will build a React.js Todo app using TypeScript, initialize the project using Vite, and set up Cypress for end-to-end (E2E) testing. We will cover all the steps, including project setup, writing the Todo app, and creating Cypress tests.

### Prerequisites
Before we start, ensure you have the following installed on your machine:

* Node.js and npm (or yarn)
* A code editor (VS Code is recommended)

### Initialize the Project with Vite
Let's create a new React project using Vite.

```bash
# Create a new Vite project
npm create vite@latest react-todo-app -- --template react-ts

# Navigate to the project directory
cd react-todo-app

# Install dependencies
npm install
```

### Let's Create the Todo App

We will now create a simple Todo app with the following features:

1. Add a new todo item.
2. Mark a todo item as complete.
3. Delete a todo item.

First, let's create our components.

#### Create the TodoItem Component
Create a components directory inside `src/components` and add a TodoItem.tsx file.

```jsx
import React from "react";

interface TodoItemProps {
  todo: {
    id: number;
    text: string;
    completed: boolean;
  };
  toggleComplete: (id: number) => void;
  deleteTodo: (id: number) => void;
}

const TodoItem: React.FC<TodoItemProps> = ({
  todo,
  toggleComplete,
  deleteTodo,
}) => {
  return (
    <div>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => toggleComplete(todo.id)}
      />
      <span
        style={{ textDecoration: todo.completed ? "line-through" : "none" }}
      >
        {todo.text}
      </span>
      <button onClick={() => deleteTodo(todo.id)}>Delete</button>
    </div>
  );
};

export default TodoItem;
```

#### Create the TodoList Component
Next, create a TodoList.tsx file inside `src/components`.

```jsx
import React from "react";
import TodoItem from "./TodoItem";

interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

interface TodoListProps {
  todos: Todo[];
  toggleComplete: (id: number) => void;
  deleteTodo: (id: number) => void;
}

const TodoList: React.FC<TodoListProps> = ({
  todos,
  toggleComplete,
  deleteTodo,
}) => {
  return (
    <ul>
      {todos.map((todo, index) => (
        <li key={index}>
          <TodoItem
            key={todo.id}
            todo={todo}
            toggleComplete={toggleComplete}
            deleteTodo={deleteTodo}
          />
        </li>
      ))}
    </ul>
  );
};

export default TodoList;
```

#### Update the App Component

Finally, let's update our main App component.

```jsx
import React, { useState } from "react";
import TodoList from "./components/TodoList";

interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

const App: React.FC = () => {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [newTodo, setNewTodo] = useState("");

  const addTodo = () => {
    if (newTodo.trim() === "") return;
    const newTodoItem = {
      id: Date.now(),
      text: newTodo,
      completed: false,
    };
    setTodos([...todos, newTodoItem]);
    setNewTodo("");
  };

  const toggleComplete = (id: number) => {
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
      <h1>Todo App</h1>
      <input
        type="text"
        value={newTodo}
        onChange={(e) => setNewTodo(e.target.value)}
        placeholder="Add a new todo"
      />
      <button onClick={addTodo}>Add Todo</button>
      <TodoList
        todos={todos}
        toggleComplete={toggleComplete}
        deleteTodo={deleteTodo}
      />
    </div>
  );
};

export default App;
```

Update the main.tsx to render the App component.

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root") as HTMLElement).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

### Set Up Cypress for E2E Testing
#### Install Cypress

Install Cypress as a development dependency.

```bash
npm install cypress --save-dev
```

#### Configure Cypress
Open Cypress for the first time to generate the necessary configuration files.

```
npx cypress open
```

When asked whether to install the dependencies
```bash
Need to install the following packages:
cypress@13.13.0
Ok to proceed? (y)
```

Answer with `y` to signify `yes`.

A new ui from our browser should open with two options, `E2E Testing` and `Component Testing`.
CLick on the first option to setup the required files.

This command will create a cypress directory with several files inside it. You should also have in your project root directory a file called `cypress.config.ts`.

Update the `cypress.config.ts` with the content so that it picks up our test files.

```ts
import { defineConfig } from 'cypress'

export default defineConfig({
  e2e: {
    specPattern: 'cypress/e2e/**/*.{cy,spec}.{js,jsx,ts,tsx}',
    baseUrl: 'http://localhost:5173'
  }
})
```

The `specPattern` is specifying where to look for file and to look fo typescript and javascrpt file with the the `.cy` pattern
The baseUrl is setting the base url of our todo app server.

Click contnue in the ui.

#### Write Cypress Tests
Let's write some tests for our Todo app. Create a new file `todo_spec.cy.ts` in the cypress/e2e directory.

```ts
describe("Todo App", () => {
  beforeEach(() => {
    cy.visit("/");
  });

// Entering text into a todo box and clicking in add todo button should make the text entered
// to appear in teh list of todos
  it("should add a new todo", () => {
    cy.get('input[placeholder="Add a new todo"]').type("Learn Cypress");
    cy.contains("Add Todo").click();
    cy.contains("Learn Cypress").should("exist");
  });

// Click on the checkbox next to a todo and it should have a strike through css property
  it("should mark a todo as complete", () => {
    cy.get('input[placeholder="Add a new todo"]').type("Learn Cypress");
    cy.contains("Add Todo").click();
    cy.contains("Learn Cypress")
      .parent()
      .find('input[type="checkbox"]')
      .check();
    cy.contains("Learn Cypress")
      .should("have.css", "text-decoration")
      .and("include", "line-through");
  });

    // Click on a delete button next to a todo and the todo shuld not exist
  it("should delete a todo", () => {
    cy.get('input[placeholder="Add a new todo"]').type("Learn Cypress");
    cy.contains("Add Todo").click();
    cy.contains("Learn Cypress").parent().contains("Delete").click();
    cy.contains("Learn Cypress").should("not.exist");
  });

// Try adding an empty todo and it should not be added to thelist
  it("should not add empty todo", () => {
    cy.get('input[placeholder="Add a new todo"]').type("{enter}");
    cy.get("ul").should("not.contain", "li");
  });

// Add a todo, click on the checkbox next it to make sure that it is marked as complete
// with a css strike through and in the same test, uncheck it to make sure the strike
//through dissappears
  it("should toggle todo completion", () => {
    cy.get('input[placeholder="Add a new todo"]').type("Learn Cypress");
    cy.contains("Add Todo").click();
    cy.contains("Learn Cypress")
      .parent()
      .find('input[type="checkbox"]')
      .check();
    cy.contains("Learn Cypress")
      .should("have.css", "text-decoration")
      .and("include", "line-through");
    cy.contains("Learn Cypress")
      .parent()
      .find('input[type="checkbox"]')
      .uncheck();
    cy.contains("Learn Cypress")
      .should("have.css", "text-decoration")
      .and("not.include", "line-through");
  });

  it("should clear input after adding todo", () => {
    cy.get('input[placeholder="Add a new todo"]').type("Learn Cypress");
    cy.contains("Add Todo").click();
    cy.get('input[placeholder="Add a new todo"]').should("have.value", "");
  });
});

```

In the ui, after continue, select your preferred browser and select run test.

In the new window, you shall see the newly created test `todo_spec.cy.ts`.

#### Run Cypress Headlessly
You can also run Cypress tests in headless mode for automated testing pipelines.

```bash
npx cypress run --headless
```

This command runs all Cypress tests in headless mode, which is suitable for CI/CD pipelines.

#### Conclusion
In this extended tutorial, we added more Cypress tests to cover edge cases and user interactions in our React.js Todo app. Cypress provides a powerful framework for writing E2E tests that ensure the functionality and reliability of your application. By following these steps, you can effectively test and validate your React.js applications built with TypeScript and initialized using Vite.
