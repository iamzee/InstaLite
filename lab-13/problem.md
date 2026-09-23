# Prisma Client + CRUD

We will be continuing the **InstaLite** app and this lab, we will be building the routes to perform CRUD (Create, Read, Update and Delete) operations on `User` and `Post` tables created in the previous lab.

## What you will build

We will be building the following routes using the MVC framework

| Endpoint            | Purpose        |
| ------------------- | -------------- |
| `POST /users`       | Create a user  |
| `GET /users`        | List all users |
| `GET /users/:id`    | Get one user   |
| `PATCH /users/:id`  | Update a user  |
| `DELETE /users/:id` | Delete a user  |
| `POST /posts`       | Create a post  |
| `GET /posts`        | List all posts |
| `GET /posts/:id`    | Get one post   |
| `PATCH /posts/:id`  | Update a post  |
| `DELETE /posts/:id` | Delete a post  |

## Step 01 - Creating an express server

Ok, we know we need to create the above routes and we know to create these routes, we need **Express**. So, without a delay, let's first create a simple Express server.

**Task:**

1. Create `src/index.js` file. This file is going to be the main file for our backend application.
2. Install `express`.
3. Import `express` module.
4. Create an `express` app.
5. Create a single route `GET /health` that responds with `200` status code and a JSON response `{ "status": "OK" }`.
6. Start the server on port 3000.
7. When the server start, it should print `"Server started on port 3000"` on the console.

**Test:**

How to test everything works fine????

Start the server. It should print the required statement on the console.
(I hope you know how to start a server 😶‍🌫️)

Then open Insomnia / Postman / Thunderclient, and send a request to `GET /health` and check whether you are getting the correct response with correct status code and correct content type of `application/json`

## Step 02 - Generate Prisma Client

Now once the express app is ready, we need to generate the prisma client, so that we can use prisma to query data from the database.

**Task:**

1. Generate Prisma Client

**Test:**

If the command successfully runs, you will get somewhat like the below output

```txt
◇ injected env (1) from .env
Loaded Prisma config from prisma7.config.js.

Prisma schema loaded from prisma/schema.prisma.

✔ Generated Prisma Client (v7.10.0) to ./node_modules/@prisma/client in 55ms

Start by importing your Prisma Client (See: https://pris.ly/d/importing-client)
```

## Step 03 - Setup prisma client

Now, we generated the prisma client in step 02, we need to set it up so, that we can use `prisma` to query data from the database.
Its a one time step

**Task:**

1. Setup prisma client inside `src/index.js`.

**Test:**

To test whether your prisma client is setup correctly and you can access the data successfully from the database, copy paste the below command below your prisma setup insde `src/index.js` and check whether you are getting `"✅ Database connection successful"` inside the console.

```js:src/index.js
async function testConnection() {
  try {
    await prisma.$queryRaw`SELECT 1`;

    console.log("✅ Database connection successful");
  } catch (error) {
    console.error("❌ Database connection failed");
    console.error(error);
  } finally {
    await prisma.$disconnect();
  }
}

testConnection();
```

You can delete this after you test your setup, hehe.

## Step 04 - POST /users

Now, our `express` and `prisma`, both are setup. Let's start CRUD operations on the `User` table first.

We will start with creating users in the `User` table.

**Task:**

In `src/server.js`, add `POST /users`:

| Aspect                         | Requirement                                                           |
| ------------------------------ | --------------------------------------------------------------------- |
| Accepted body fields           | `username`, `email`, `fullname` (required); `bio` (optional)          |
| Any other body field           | Ignored (a client must not be able to set `id`, `createdAt`, etc.)    |
| A required field is missing    | `400` with `{ "error": "username, email and fullname are required" }` |
| Success                        | `201` with the created user                                           |
| Username or email already used | `409` with `{ "error": "username or email already taken" }`           |

**Test:**

```sh
# 1
curl -i -X POST localhost:3000/users -H "Content-Type: application/json" \
  -d '{"username":"riya_travels","email":"riya@example.com","fullname":"Riya Sharma"}'
# 2
curl -i -X POST localhost:3000/users -H "Content-Type: application/json" \
  -d '{"username":"riya_travels","email":"other@example.com","fullname":"Copy Cat"}'
# 3
curl -i -X POST localhost:3000/users -H "Content-Type: application/json" \
  -d '{"username":"onlyname"}'
# 4
curl -i -X POST localhost:3000/users -H "Content-Type: application/json" \
  -d '{"username":"sneaky","email":"sneaky@example.com","fullname":"Sneaky","id":999}'
```

**Expected Results:**

| #   | Status | Body                                                                                                                             |
| --- | ------ | -------------------------------------------------------------------------------------------------------------------------------- |
| 1   | `201`  | User object with an `id`, `username` `"riya_travels"`, `bio` `null`, `isPrivate` `false`, and `createdAt`/`updatedAt` timestamps |
| 2   | `409`  | `{ "error": "username or email already taken" }`                                                                                 |
| 3   | `400`  | `{ "error": "username, email and fullname are required" }`                                                                       |
| 4   | `201`  | User created with an auto-generated `id` - **not** `999`                                                                         |

## Step 05 - GET /users

**Task:**

Add `GET /users` route, returning every user as a JSON array with status code of 200.

**Test:**

```sh
curl -i localhost:3000/users
```

## Step 06 - GET /users/:id

**Task:**

Add `GET /users/:id`, returning one user.

| Case                 | Status | Body                            |
| -------------------- | ------ | ------------------------------- |
| User exists          | `200`  | The user object                 |
| No user with that id | `404`  | `{ "error": "User not found" }` |

**Tests:**

Test both cases - `200` as well as `404`

```shell
curl -i localhost:3000/users
```

## Step 07 - PATCH /users/:id

Time to edit user with the given `id`

**Task:**

Add `PATCH /users/:id`, a partial update: only the fields sent are changed.

| Aspect               | Requirement                                    |
| -------------------- | ---------------------------------------------- |
| Editable fields      | `fullname`, `bio`, `isPrivate`                 |
| Not editable         | `username`, `email` — silently ignored if sent |
| Success              | `200` with the updated user                    |
| No user with that id | `404` with `{ "error": "User not found" }`     |

## Step 08 - DELETE /users/:id

Time to delete users with the given `id`

**Task:**

Add `DELETE /users/:id`.

| Case                 | Status | Body                            |
| -------------------- | ------ | ------------------------------- |
| Deleted              | `204`  | Empty                           |
| No user with that id | `404`  | `{ "error": "User not found" }` |

## Step 09 - Refactor: shared Prisma Client in `src/db.js`

**Task:**

1. Create src/db.js. It must create the one Prisma Client for the whole app (including loading .env and the driver adapter) and export it as the default export.
2. Change src/server.js to import the client from db.js instead of creating its own.

**Tests:**

Test all the previous steps.

## Step 10 - Refactor: Users routes in `src/routes/users.routes.js`

**Task:**

1. Create `src/routes/users.routes.js` containing an Express Router with all five user routes. Export the router.
2. Use the exported users router in `src/index.js`

**Tests:**

Test all the previous routes again.

## Step 11 - Refactor: controllers in `src/controllers/users.controller.js`

**Tasks:**

1. Create `src/controllers/users.controller.js` exporting five handler functions: `create`, `list`, `getOne`, `update`, `remove`.
2. Change `src/routes/users.routes.js` so that each route is one line that connects a method and path to a controller function.

**Tests:**

Test all the previous routes again.

## Step 12 - Refactor: services in `src/services/users.service.js`

**Task:**

1. Create `src/services/users.service.js` exporting five functions: `createUser(data)`, `listUsers()`, `getUserById(id)`, `updateUser(id, data)`, `deleteUser(id)`.
2. Move every Prisma query out of the controller and into these functions.
3. The service must not use req or res.
4. Each controller function must now: read from req, call one service function, send the response.

**Tests:**

Test all the previous routes again.

## Step 13-17 - `Post` table endpoints

From here on you work directly in the folder structure. Every post endpoint is built across three files:

- `src/services/posts.service.js`
- `src/controllers/posts.controller.js`
- `src/routes/posts.routes.js, mounted at /posts in app.js`

A `Post` has `imageUrl`, `caption`, `location`, `createdAt` and `updatedAt`.

## Step 13 - `Post /posts`

**Task:**
 
| Aspect | Requirement |
| --- | --- |
| Accepted body fields | `imageUrl` (required); `caption`, `location` (optional) |
| Any other body field | Ignored |
| `imageUrl` missing | `400` with `{ "error": "imageUrl is required" }` |
| Success | `201` with the created post |

## Step 14 - `GET /posts`

**Task**
 
Return every post as a JSON array with `200` status code

## Step 15 - `GET /posts/:id`
 
**Task**
 
| Case | Status | Body |
| --- | --- | --- |
| Post exists | `200` | The post object |
| No post with that id | `404` | `{ "error": "Post not found" }` |

## Step 16 - `PATCH /posts/:id`
 
**Task**
 
| Aspect | Requirement |
| --- | --- |
| Editable fields | `caption`, `location` |
| Not editable | `imageUrl` — silently ignored if sent |
| Success | `200` with the updated post |
| No post with that id | `404` with `{ "error": "Post not found" }` |

## Step 17 - `DELETE /posts/:id`
 
**Task**
 
| Case | Status | Body |
| --- | --- | --- |
| Deleted | `204` | Empty |
| No post with that id | `404` | `{ "error": "Post not found" }` |

## Step 17 - Final test run
 
**Task**
 
Restart the server and run the full sequence.
