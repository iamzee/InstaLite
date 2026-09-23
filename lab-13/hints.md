# Helper guide

## Step 01 - Creating an express server

1. Do you really need a hint to create a folder? 🤦‍♂️
2. For rest of the steps on how to install and setup express, just go to the frontpage of `express` doc and you will have everything there. [Express doc](https://expressjs.com)

## Step 02 - Generate Prisma Client

1. Check the [Reference Doc](https://www.prisma.io/docs/orm/v7/prisma-client).
2. Look for the command to geenrate the prisma client.

## Step 03 - Setup prisma client

Again, we will take help of the official documentation to check how to setup prisma.

[Doc to setup prisma client](https://www.prisma.io/docs/orm/v7/prisma-client/setup-and-configuration/introduction).

Find the relevant code to setup the prisma client and paste it inside `src/index.js`.
And don't forget to replace `ESM (import/export)` to `Common JS (require)` in your code.

## Step 04 - POST /users

- Don't forget to add a middleware that is going to parse the request body, otherwise `req.body` is going to `undefined`. [Check Doc](https://expressjs.com/en/5x/guide/using-middleware/#built-in-middleware)
- [Documentation on how to create a row inside a table using prisma](https://www.prisma.io/docs/orm/v7/prisma-client/queries/crud)

## Step 05 - GET /users

[Documentation to get all the data](https://www.prisma.io/docs/orm/v7/prisma-client/queries/crud#read)

## Step 06 - GET /users/:id

[Documentation to access request parameters](https://expressjs.com/en/5x/guide/routing/#route-parameters)

[Documentation to read by id or unique field](https://www.prisma.io/docs/orm/v7/prisma-client/queries/crud#get-record-by-id-or-unique-field)

## Step 07 - PATCH /users/:id

[Prisma doc to update a single record](https://www.prisma.io/docs/orm/v7/prisma-client/queries/crud#update)

## Step 08 - DELETE /users/:id

[Documentation to delete a single record](https://www.prisma.io/docs/orm/v7/prisma-client/queries/crud#delete)

## Step 09 - Refactor: shared Prisma Client in `src/db.js`

Just copy/paste from one file to another. Don't forget to copy the imports too.

## Step 10 - Refactor: Users routes in `src/routes/users.routes.js`

[Express router documentation](https://expressjs.com/en/5x/guide/routing/#expressrouter)

## Step 11 - Refactor: controllers in `src/controllers/users.controller.js`

Ok, for each route we have a callback function defined.

For example, for `POST /users` route, we had the following callback

```js
router.post("/", async (req, res) => {
  try {
    delete req.body;
    if (!req.body.username || !req.body.email || !req.body.fullname) {
      return res
        .status(400)
        .json({ error: "username, email and fullname are required" });
    }

    const result = await prisma.user.create({
      data: req.body,
    });
    res.status(201).json(result);
  } catch (err) {
    res.status(409).json({ error: "username or email already taken" });
  }
});
```

We need to extract the callback each callback into a file called `controllers/users.controllers.js`


## Step 12 - Refactor: services in `src/services/users.service.js`

Copy the Prisma related queries to a new file `src/services/users.services.js`.

And do not use `req` and `res` inside the service file.

Whatever data service file needs, that will be sent by controllers.
