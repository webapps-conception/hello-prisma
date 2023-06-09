# Quickstart Prisma

In this Quickstart guide, you'll learn how to get started with Prisma from scratch using a plain TypeScript project and a local SQLite database file. It covers data modeling, migrations and querying a database.

If you want to use Prisma with your own PostgreSQL, MySQL, MongoDB or any other supported database, go here instead:

* [Start with Prisma from scratch](https://www.prisma.io/docs/getting-started/setup-prisma/start-from-scratch/relational-databases-typescript-postgresql)
* [Add Prisma to an existing project](https://www.prisma.io/docs/getting-started/setup-prisma/add-to-existing-project/relational-databases-typescript-postgresql)

## Create TypeScript project and set up Prisma

As a first step, create a project directory and navigate into it:

```bash
mkdir hello-prisma
cd hello-prisma
```

Next, initialize a TypeScript project using npm:
```bash
npm init -y
npm install typescript ts-node @types/node --save-dev
```

This creates a package.json with an initial setup for your TypeScript app.

See [installation instructions](https://www.prisma.io/docs/reference/api-reference/command-reference#installation) to learn how to install Prisma using a different package manager.

Now, initialize TypeScript:
```bash
npx tsc --init
```

Then, install the Prisma CLI as a development dependency in the project:
```bash
npm install prisma --save-dev
```

Finally, set up Prisma with the init command of the Prisma CLI:
```bash
npx prisma init --datasource-provider sqlite
```

This creates a new prisma directory with your Prisma schema file and configures SQLite as your database. You're now ready to model your data and create your database with some tables.

## Model your data in the Prisma schema

The Prisma schema provides an intuitive way to model data. Add the following models to your schema.prisma file:

🗈 prisma/schema.prisma
```sql
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}

model Post {
  id        Int     @id @default(autoincrement())
  title     String
  content   String?
  published Boolean @default(false)
  author    User    @relation(fields: [authorId], references: [id])
  authorId  Int
}
```

Models in the Prisma schema have two main purposes:
* Represent the tables in the underlying database
* Serve as foundation for the generated Prisma Client API

In the next section, you will map these models to database tables using Prisma Migrate.

## Run a migration to create your database tables with Prisma Migrate

At this point, you have a Prisma schema but no database yet. Run the following command in your terminal to create the SQLite database and the User and Post tables represented by your models:

```bash
npx prisma migrate dev --name init
```

This command did two things:

1. It creates a new SQL migration file for this migration in the prisma/migrations directory.

2. It runs the SQL migration file against the database.

Because the SQLite database file didn't exist before, the command also created it inside the prisma directory with the name dev.db as defined via the environment variable in the .env file.

Congratulations, you now have your database and tables ready. Let's go and learn how you can send some queries to read and write data!

## Explore how to send queries to your database with Prisma Client

To send queries to the database, you will need a TypeScript file to execute your Prisma Client queries. Create a new file called script.ts for this purpose:

```bash
touch script1.ts script2.ts script3.ts script3.ts
```

Then, paste the following boilerplate into it:

🗈 script1.ts
```typescript
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  // ... you will write your Prisma Client queries here
}

main()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
  })
```

This code contains a main function that's invoked at the end of the script. It also instantiates PrismaClient which represents the query interface to your database.

### Create a new ``User`` record

Let's start with a small query to create a new User record in the database and log the resulting object to the console. Add the following code to your script.ts file:

🗈 script1.ts
```typescript
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  const user = await prisma.user.create({
    data: {
      name: 'Alice',
      email: 'alice@prisma.io',
    },
  })
  console.log(user)
}

main()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
  })
```

Instead of copying the code, you can type it out in your editor to experience the autocompletion Prisma Client provides. You can also actively invoke the autocompletion by pressing the CTRL+SPACE keys on your keyboard.

Next, execute the script with the following command:
```bash
npx ts-node script1.ts

Hide CLI results:
{ id: 1, email: 'alice@prisma.io', name: 'Alice' }
```

Great job, you just created your first database record with Prisma Client! 🎉

In the next section, you'll learn how to read data from the database.

### Retrieve all User records

Prisma Client offers various queries to read data from your database. In this section, you'll use the findMany query that returns all the records in the database for a given model.

Delete the previous Prisma Client query and add the new findMany query instead:

🗈 script2.ts
```typescript
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  const users = await prisma.user.findMany()
  console.log(users)
}

main()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
  })
```

Execute the script again:
```bash
npx ts-node script2.ts

Hide CLI results
[{ id: 1, email: 'alice@prisma.io', name: 'Alice' }]
```

Notice how the single User object is now enclosed with square brackets in the console. That's because the findMany returned an array with a single object inside.

### Explore relation queries with Prisma

One of the main features of Prisma Client is the ease of working with relations. In this section, you'll learn how to create a User and a Post record in a nested write query. Afterwards, you'll see how you can retrieve the relation from the database using the include option.

First, adjust your script to include the nested query:

🗈 script3.ts
```typescript
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  const user = await prisma.user.create({
    data: {
      name: 'Bob',
      email: 'bob@prisma.io',
      posts: {
        create: {
          title: 'Hello World',
        },
      },
    },
  })
  console.log(user)
}

main()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
  })
```

Run the query by executing the script again:
```bash
npx ts-node script3.ts

Hide CLI results
{ id: 2, email: 'bob@prisma.io', name: 'Bob' }
```

By default, Prisma only returns scalar fields in the result objects of a query. That's why, even though you also created a new Post record for the new User record, the console only printed an object with three scalar fields: id, email and name.

In order to also retrieve the Post records that belong to a User, you can use the include option via the posts relation field:

🗈 script4.ts
```typescript
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  const usersWithPosts = await prisma.user.findMany({
    include: {
      posts: true,
    },
  })
  console.dir(usersWithPosts, { depth: null })
}

main()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
  })
```

Run the script again to see the results of the nested read query:
```bash
npx ts-node script4.ts

Hide CLI results
[
  { id: 1, email: 'alice@prisma.io', name: 'Alice', posts: [] },
  {
    id: 2,
    email: 'bob@prisma.io',
    name: 'Bob',
    posts: [
      {
        id: 1,
        title: 'Hello World',
        content: null,
        published: false,
        authorId: 2
      }
    ]
  }
]
```

This time, you're seeing two User objects being printed. Both of them have a posts field (which is empty for "Alice" and populated with a single Post object for "Bob") that represents the Post records associated with them.

Notice that the objects in the usersWithPosts array are fully typed as well. This means you will get autocompletion and the TypeScript compiler will prevent you from accidentally typing them.

## Next steps

In this Quickstart guide, you have learned how to get started with Prisma in a plain TypeScript project. Feel free to explore the Prisma Client API a bit more on your own, e.g. by including filtering, sorting, and pagination options in the findMany query or exploring more operations like update and delete queries.

### Explore the data in Prisma Studio

Prisma comes with a built-in GUI to view and edit the data in your database. You can open it using the following command:

```bash
npx prisma studio
```

### Set up Prisma with your own database

If you want to move forward with Prisma using your own PostgreSQL, MySQL, MongoDB or any other supported database, follow the Set Up Prisma guides:

* [Start with Prisma from scratch](https://www.prisma.io/docs/getting-started/setup-prisma/start-from-scratch/relational-databases-typescript-postgresql)
* [Add Prisma to an existing project](https://www.prisma.io/docs/getting-started/setup-prisma/add-to-existing-project)

### Explore ready-to-run Prisma examples

Check out the [prisma-examples](https://github.com/prisma/prisma-examples/) repository on GitHub to see how Prisma can be used with your favorite library. The repo contains examples with Express, NestJS, GraphQL as well as fullstack examples with Next.js and Vue.js, and a lot more.

### Build an app with Prisma

The Prisma blog features comprehensive tutorials about Prisma, check out our latest ones:

* [Build a fullstack app with Remix](https://www.prisma.io/blog/fullstack-remix-prisma-mongodb-1-7D0BfTXBmB6r) (5 parts, including videos)
* [Build a REST API with NestJS](https://www.prisma.io/blog/nestjs-prisma-rest-api-7D056s1BmOL0)

## Author

[Prisma](https://www.prisma.io/docs/getting-started/quickstart)

## License

This project is licensed under the MIT license. See the [LICENSE](./LICENSE) file for more info.
