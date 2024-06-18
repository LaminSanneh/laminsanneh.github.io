---
title: How to Use Laravel 11 Gates and Policies to Secure Your Application Data through Your Controllers and Routes
description: ""
date: 2024-06-18T19:44:58.929Z
preview: ""
draft: false
tags: []
categories: []
type: posts
slug: laravel-11-gates-policies-secure-application-data-controllers-routes
hero: /images/posts/2024-06-18-how-to-use-laravel-11-gates-and-policies-to-secure-your-application-data-through-your-controllers-and-routes.png
---

### Introduction
Securing your application data is a crucial aspect of building robust web applications. Laravel 11 provides a powerful mechanism to handle authorization through Gates and Policies. This post will guide you through the process of using Gates and Policies to secure your application data within a controller, using a `Post` model as an example.

### What are Gates and Policies?
Gates and Policies are two complementary features in Laravel that help in authorizing actions. Gates provide a simple closure-based approach to authorization, whereas Policies are dedicated classes that group authorization logic for a specific model or resource.

### Defining a Policy
In Laravel, a Policy is a class that contains authorization logic for a particular model. Let's define a `PostPolicy` to manage authorization for viewing, updating, and deleting posts.

`PostPolicy.php`
```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * Determine whether the user can view the model.
     */
    public function view(User $user, Post $post): bool
    {
        return $user->isAdmin() || $user->isEditor() || $post->author_id == $user->id;
    }

    /**
     * Determine whether the user can update the model.
     */
    public function update(User $user, Post $post): bool
    {
        return $user->isAdmin() || $user->isEditor() || $post->author_id === $user->id;
    }

    /**
     * Determine whether the user can delete the model.
     */
    public function delete(User $user, Post $post): bool
    {
        return $user->isAdmin() || $user->isEditor() || $post->author_id === $user->id;
    }
}

```

In this PostPolicy class, the methods `view`, `update`, and `delete` determine if a user can perform these actions based on their role (admin or editor) or if they are the author of the post.

### Applying Policies in a Controller
To use these policies in your controller, you will leverage the `Gate::authorize` method. This method checks the authorization and throws an exception if the user is not authorized.

`PostController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;
use InvalidArgumentException;

class PostsController extends Controller
{
    public function find(int $postId)
    {
        $post = Post::find($postId);

        Gate::authorize('view', $post);

        return response()->json($post);
    }

    public function update(Request $request, int $postId)
    {
        $data = $request->only(['title', 'body']);

        $post = Post::with('author')->find($postId);

        Gate::authorize('update', $post);

        $post->update($data);

        return response()->json($post);
    }

    public function delete(int $postId)
    {
        $post = Post::find($postId);

        Gate::authorize('delete', $post);

        $deleted = $post->delete();

        if (!$deleted) {
            throw new InvalidArgumentException('Could Not Delete Post');
        }

        return response()->json();
    }
}
```

In this `PostsController` class:
* The `find` method retrieves a post and authorizes the 'view' action.
* The `update` method updates a post after authorizing the 'update' action.
* The `delete` method deletes a post after authorizing the 'delete' action.

If the current user is not authorized to perform the action, `Gate::authorize` will throw an `AuthorizationException`, effectively preventing unauthorized access.

### Manually Handling Authorization Failures
While Gate::authorize is convenient, it throws an exception when the check fails, but there are scenarios where you might want to manually handle the failure of an authorization check. For example, you might want to return a custom error message or perform some other action.

#### Manual Authorization Check
Here's how you can manually handle authorization failures in a controller method:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;
use InvalidArgumentException;

class PostsController extends Controller
{
    public function create(Request $request)
    {
        if ($request->user()->cannot('create', Post::class)) {
            abort(403, 'You are not authorized to create a post');
        }

        // Proceed with post creation
        $data = $request->only(['title', 'body']);
        $post = Post::create($data);

        return response()->json($post);
    }

    // Other methods (find, update, delete)...
}
```

In this create method, we manually check if the user has permission to create a post using `$request->user()->cannot('create', Post::class)`. If the user is not authorized, we use abort(403, 'You are not authorized to create a post') to return a 403 Forbidden response with a custom message.

### Defining Gates
Gates can be defined in the `AuthServiceProvider` or anywhere in your application bootstrapping process. Here’s how to define gates using both closure and policy methods.

#### Closure-Based Gates
You can define a gate using a closure in the `AuthServiceProvider`:

```php
use Illuminate\Support\Facades\Gate;
use App\Models\Post;
use App\Models\User;

/**
 * Register any authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Gate::define('update', function (User $user, Post $post) {
        return $user->id === $post->author_id;
    });
}
```

This gate checks if the user’s ID matches the author ID of the post.

#### Policy-Based Gates
You can also define a gate to use a specific method from a policy:

```php
use Illuminate\Support\Facades\Gate;
use App\Policies\PostPolicy;

/**
 * Register any authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Gate::define('update', [PostPolicy::class, 'update']);
}
```

This gate uses the `update` method from the `PostPolicy` class to perform the authorization check.

We have attached the PostPolicy as a handler for the PostModel, but if we created a policy class using a convention of `[ModelName][Policy]`, this definition would not be necessary if the gate name `update` and policy method handler `update` are the same.

This is because using the convention to name your policy Laravel will discover the policy and link it to the model used in the name automatically, given that it is in the policies folder. However, if for example in this example above we wanted to not use the covention and use a policy class like `DraftPostPolicy` and our post model is called Post, then Laravel will not pick it up automatically, that is when the code snippet above is neccessary, so instead we'd do something like:

```php
use Illuminate\Support\Facades\Gate;
use App\Policies\DraftPostPolicy;

/**
 * Register any authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Gate::define('update', [DraftPostPolicy::class, 'update']);
}
```

### Using Gates at the Route Level
Laravel also allows you to apply gates directly in route definitions using middlewares. This approach is helpful for simple authorization checks that can be defined directly within your route declarations.

#### Middleware-Based Authorization in Routes
You can use the `can` middleware to check authorizations at the route level:

```php
use App\Models\Post;

Route::put('/post/{post}', function (Post $post) {
    // The current user may update the post...
})->middleware('can:update,post');

Route::post('/post', function () {
    // The current user may create posts...
})->middleware('can:create,App\Models\Post');
```

In these examples:

* The `PUT /post/{post}` route uses the `can:update,post` middleware to ensure the user can update the specified post.
* The `POST /post` route uses the `can:create,App\Models\Post` middleware to ensure the user can create a new post.


#### Route Method Authorization
You can also use the `can` method directly in your route definition:

```php
use App\Models\Post;

Route::post('/post', function () {
    // The current user may create posts...
})->can('create', Post::class);
```

This example checks if the current user can create a post before executing the route's closure.

### Using Gates at the Blade Template Level
Laravel's Blade templating engine makes it easy to include authorization checks directly in your views. You can use the `@can`, `@cannot`, `@elsecan`, and `@else` directives to conditionally display content based on the user's permissions.

#### Blade Example
Here’s how you can use Gates at the Blade template level:

```php
@can('update', $post)
    <!-- The current user can update the post... -->
@endcan

@can('create', App\Models\Post::class)
    <!-- The current user can create new posts... -->
@endcan

@cannot('delete', $post)
    <!-- The current user cannot delete this post... -->
@endcannot

@elsecan('create', App\Models\Post::class)
    <!-- The current user can create new posts... -->
@else
    <!-- The current user cannot update or create posts... -->
@endcan
```

In this example:

* `@can('update', $post)` checks if the current user can update the given post.
* `@can('create', App\Models\Post::class)` checks if the current user can create a new post.
* `@cannot('delete', $post)` checks if the current user cannot delete the given post.
* `@elsecan('create', App\Models\Post::class)` provides an alternative block if the user can create a post but not update or delete.
* `@else` is used as a fallback for users who do not meet any specified conditions.

### Registering Policies

To ensure Laravel knows about your policies, you need to register them in the AuthServiceProvider.

`AuthServiceProvider.php`

```php
<?php

namespace App\Providers;

use App\Models\Post;
use App\Policies\PostPolicy;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * The policy mappings for the application.
     *
     * @var array
     */
    protected $policies = [
        Post::class => PostPolicy::class,
    ];

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();
    }
}

```

Here, we map the Post model to the PostPolicy class.

We have attached the PostPolicy as a handler for the Post Model, but if we created a policy class using a convention of `[ModelName][Policy]`, this definition in the `AuthServiceProvider` would not be necessary as laravel will discover it automatically, given that it is in the policies folder. However, if for example in this example above we wanted to not use the covention and use a policy class like `DraftPostPolicy` and our post model is called Post, then Laravel will not pick it up automatically, that is when the code snippet above is neccessary, so instead we'd do something like:

```php
protected $policies = [
    Post::class => DraftPostPolicy::class,
];
```

### Conclusion
By using Laravel 11's Gates and Policies, you can implement robust security measures in your application. Policies allow you to encapsulate authorization logic in dedicated classes, keeping your controllers clean and focused. The `Gate::authorize` method ensures that only authorized users can perform specific actions, throwing exceptions for unauthorized attempts. Additionally, manually handling authorization failures provides flexibility to customize responses and actions based on your application's needs. Defining gates in the `AuthServiceProvider` allows you to centralize and manage your authorization logic effectively.

Applying authorization at the route level using middleware and the `can` method ensures a clean and declarative approach to securing your routes. This method enhances the readability and maintainability of your route definitions while providing robust security. Furthermore, using gates within Blade templates ensures that only authorized content is rendered, enhancing the user experience and security of your application.

This approach not only secures your application data but also makes your authorization logic reusable and easy to maintain. Happy coding!