# Adding Prisma to a Turborepo as a Package

This guide walks you through setting up **Prisma** as a shared package in a **Turborepo** monorepo structure.

## Steps

### 1. Create a New Package for Prisma

First, navigate to the `package` directory inside your Turborepo and create a new `database` package:

```bash
cd package
mkdir database
```

### 2. Initialize the Package with Bun

Navigate into the `database` directory and initialize the package using **Bun**:

```bash
cd database
bun init -y
```

This will create a `package.json` file with default settings.

### 3. Initialize Prisma

Run the following command to initialize **Prisma** in the `database` package:

```bash
bunx prisma init
```

This will generate a `prisma` folder containing the `schema.prisma` file and a `.env` file for the database URL.

### 4. Set Up the `src` Folder and `index.ts`

Next, create a `src` folder inside the `database` package:

```bash
mkdir src
cd src
touch index.ts
```

Inside `src/index.ts`, add the following code to configure and export the Prisma client:

```ts
import { PrismaClient } from "@prisma/client";

declare global {
  var prisma: PrismaClient | undefined;
}

export const prisma = global.prisma || new PrismaClient();

if (process.env.NODE_ENV !== "production") global.prisma = prisma;

export * from "@prisma/client";
```

### 5. Set Up TypeScript Configuration

Now, create the `tsconfig.json` file in the `package/database` directory:

```bash
touch tsconfig.json
```

Add the following configuration to `tsconfig.json`:

```json
{
  "extends": "@repo/typescript-config/base.json",
  "compilerOptions": {
    "outDir": "dist"
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

### 6. Configure Database Environment Variable

Create an `.env` file in the `package/database` directory to store the database connection string:

```bash
touch .env
```

Inside the `.env` file, define the `DATABASE_URL`:

```bash
DATABASE_URL=""
```

Make sure to fill in the correct database URL for your project (e.g., PostgreSQL connection string).

### 7. Update `package.json` in `database` Package

Open the `package.json` file in `package/database` and make the following changes:

1. Change the `name` field:

```json
"name": "@repo/database",
```

2. Add **Prisma** as a dependency and dev dependency:

```json
"dependencies": {
  "@prisma/client": "^5.19.1"
},
"devDependencies": {
  "prisma": "^5.19.1"
}
```

3. Add **exports** to specify the Prisma client:

```json
"exports": {
  "./client": "./src/index.ts"
}
```

4. Add useful **scripts** for Prisma commands:

```json
"scripts": {
  "db:generate": "prisma generate",
  "db:push": "prisma db push --skip-generate",
  "db:migrate:deploy": "prisma migrate deploy",
  "db:migrate:dev": "prisma migrate dev"
}
```

### 8. Update `turbo.json`

Add the following configuration to your `turbo.json` file to manage caching and dependencies for Prisma commands:

```json
{
  "tasks": {
    "db:generate": {
      "cache": false
    },
    "db:push": {
      "cache": false
    },
    "db:migrate": {
      "cache": false
    },
    "build": {
      "dependsOn": ["^db:generate", "^build"],
      "inputs": ["$TURBO_DEFAULT$", ".env*"],
      "outputs": [".next/**", "!.next/cache/**"]
    }
  }
}
```

### 9. Add Root-Level Scripts

In the root `package.json`, add scripts to manage Prisma commands for the entire monorepo:

```json
{
  "scripts": {
    "db:migrate": "cd packages/database && bun run db:migrate:dev",
    "db:generate:root": "cd packages/database && bun run db:generate"
  }
}
```

These scripts allow you to run Prisma migrations and generation from the root of the monorepo.

### 10. Running Prisma Commands

You can now use the following commands to manage your Prisma setup:

- `bun run db:generate`: Generates the Prisma client from within the `packages/database` directory.
- `bun run db:push`: Pushes the schema to the database.
- `bun run db:migrate`: Deploys migrations.
- `bun run db:migrate:dev`: Creates and applies new migrations in development.

From the root of the monorepo, you can use:

- `bun run db:migrate`: Runs migrations using the root script.
- `bun run db:generate:root`: Generates the Prisma client using the root script.
