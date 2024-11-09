---
title: Go Local Modules and Packages - Understand How to Create and Use Them
description: ""
date: 2024-11-09T14:11:14.712Z
preview: ""
draft: false
tags: []
categories: []
hero: ""
fmContentType: posts
---

### Introduction

When building applications in Go, modularizing your code into separate modules and packages is essential for maintaining clean and reusable code. In this tutorial, we’ll walk through creating two local Go modules: `restapi01/backend` and `restapi/maincaller`. The `backend` module will serve as a basic HTTP server that outputs a simple HTML view, while the `maincaller` module will call functions from `backend`.

We shall look into:
1. Initializing Go modules and packages.
2. Using `go mod edit` and `go mod tidy` to set up local imports without publishing packages.
3. Testing the interaction between these modules and calling one from the other.

Let's get started!

### Project Structure

We'll set up the following structure:

```
restapi/
├── backend/
│   ├── go.mod
│   ├── server.go
├── maincaller/
│   ├── go.mod
│   ├── main.go
```

Each module (`backend` and `maincaller`) will live in its own directory and act independently.

### Step 1: Initializing the Backend Module

The `backend` module will serve as an HTTP server. Let’s set up `restapi/backend` to listen for requests and return a simple HTML response.

1. **Create the `backend` directory**:
   ```bash
   mkdir -p restapi/backend
   cd restapi/backend
   ```

2. **Initialize a Go module**:
   ```bash
   go mod init restapi01/backend
   ```

   Your `restapi/backend/go.mod` file should look like this with your current go version:

   ```bash
   module restapi01/backend

   go 1.20
   ```

3. **Write the `server.go` file**:

   ```go
   package backend

   import (
       "fmt"
       "net/http"
   )

   func StartServer() {
       http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
           fmt.Fprintln(w, "<h1>Hello from Backend!</h1>")
       })
       fmt.Println("Starting server on :8080")
       http.ListenAndServe(":8080", nil)
   }
   ```

   Here, `StartServer` is an exported function that starts an HTTP server on port `8080`. When accessed, it will return a simple HTML message.

4. **Initialize and tidy dependencies**:
   ```bash
   go mod tidy
   ```

### Step 2: Initializing the Maincaller Module

Next, let's set up `restapi/maincaller`, which will call the `backend` module to start the server.

1. **Create the `maincaller` directory**:
   ```bash
   mkdir -p ../maincaller
   cd ../maincaller
   ```

2. **Initialize a Go module**:
   ```bash
   go mod init restapi/maincaller
   ```

   Your `restapi/backend/go.mod` file should look like this with your current go version:

   ```bash
   module restapi01/maincaller

   go 1.20
   ```

3. **Write the `main.go` file**:

   ```go
   package main

   import (
       "fmt"
       "restapi01/backend"
   )

   func main() {
       fmt.Println("Calling backend server...")
       backend.StartServer()
   }
   ```

   Here, the `maincaller` package imports the `restapi01/backend` package and calls its `StartServer` function.

4. **Edit go.mod to Reference `backend` Locally**:
   Since `restapi01/backend` is a local module, we need to tell Go to look for it locally instead of remotely.

   Run the following command to add a local replace directive:
   ```bash
   go mod edit -replace=restapi01/backend=../backend
   ```

   It should add a line like:

   ```bash
   replace restapi01/backend => ../backend
   ```

5. **Use `go mod tidy`**:
   ```bash
   go mod tidy
   ```

   This command will update your `go.mod` and `go.sum` files to ensure dependencies are managed correctly and the replace directive is respected. After running it, you `go.mod` file in the `maincaller` directory should have an extra line like:

   ```bash
   require restapi01/backend v0.0.0-00010101000000-000000000000
   ```

### Step 3: Run the Application

Now, let’s run `maincaller` to see everything in action.

1. Navigate to the `maincaller` directory:
   ```bash
   cd maincaller
   ```

2. Start the application:
   ```bash
   go run main.go
   ```

   You should see the output:
   ```
   Calling backend server...
   Starting server on :8080
   ```

3. **Test the Server**:
   Open your browser and navigate to `http://localhost:8080`. You should see:
   ```
   Hello from Backend!
   ```

### What is `go mod edit -replace`

The `go mod edit -replace` directive tells Go to look for `restapi01/backend` in the local `../backend` directory rather than remotely. This is useful for testing and development, allowing us to manage dependencies without publishing modules to a repository.

### Summary
Using local modules and packages helps you organize code efficiently, especially in larger projects. With the replace directive, you gain the flexibility to structure projects without worrying about dependency management issues. Happy coding!
