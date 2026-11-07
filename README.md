# Library Management System

A library catalog data layer built with NestJS, PostgreSQL and Prisma ORM.

## Technology stack

- Node.js
- NestJS
- TypeScript
- PostgreSQL (hosted on Neon or Supabase)
- Prisma ORM

## Project structure

```text
library-management-system/
├── README.md
└── api/
    ├── prisma/
    │   ├── schema.prisma
    │   └── seed.ts
    ├── src/
    │   ├── app.module.ts
    │   ├── app.controller.ts
    │   ├── app.service.ts
    │   ├── main.ts
    │   └── prisma/
    │       ├── prisma.module.ts
    │       └── prisma.service.ts
    ├── .env.example
    ├── .gitignore
    ├── nest-cli.json
    ├── package.json
    ├── tsconfig.build.json
    └── tsconfig.json
```

## Database models

- **User** stores library users. The `Role` enum restricts a user's role to `PATRON` or `LIBRARIAN`.
- **Genre** stores book categories.
- **Book** stores catalog information and references a genre through `genreId`.
- **Review** is an optional extension. It connects a user to a book and stores a rating and comment.

## Schema Decisions

The `Book` model references the `Genre` model through the `genreId` foreign key instead of embedding genre details in every book record.

This creates a one-to-many relationship: one genre can be associated with many books, while each book belongs to one genre. Keeping genres in their own table avoids repeating the genre name and description in every book record. It also makes updates more consistent: changing a genre's name only requires updating one Genre row.

The `genre` relation field in Prisma describes the relationship in application code. The `genreId` scalar field stores the foreign key in the database. The `@relation(fields: [genreId], references: [id])` attribute tells Prisma that `genreId` references the `id` field on Genre.

Users and books are also connected through the optional Review model. A user can write multiple reviews, and a book can receive multiple reviews. Each review belongs to exactly one user and one book.

## Prisma versus a document database

In Prisma with PostgreSQL, related data is represented using separate relational tables and foreign keys. In a document database, related information is often embedded inside a document or connected using references. The relational approach makes relationships and referential integrity explicit, but retrieving related data may require joins or Prisma's `include` option.

## Setup

### 1. Install prerequisites

Install a current Node.js LTS release. Confirm the installation:

```bash
node -v
npm -v
```

Create a free PostgreSQL database with [Neon](https://neon.com/) or [Supabase](https://supabase.com/). Copy its PostgreSQL connection string.

### 2. Install dependencies

Open a terminal in the `api` directory:

```bash
npm install
```

### 3. Configure the database

Copy `.env.example` to `.env` and replace the placeholder with your hosted PostgreSQL connection string:

```env
DATABASE_URL="postgresql://USER:PASSWORD@HOST/DATABASE?sslmode=require"
```

Do not commit `.env`. The `.env.example` file is safe to commit because it contains only a placeholder.

### 4. Create and apply the migration

From the `api` directory:

```bash
npx prisma migrate dev --name init
```

This creates the PostgreSQL tables and records the migration in `prisma/migrations`.

### 5. Seed the database

```bash
npx prisma db seed
```

The seed script creates four genres and sixteen books. It uses genre upserts and skips duplicate ISBNs, so rerunning it does not create duplicate sample records.

### 6. Inspect the database

```bash
npx prisma studio
```

Prisma Studio opens a browser interface where you can inspect the `User`, `Genre`, `Book`, and `Review` tables.

### 7. Run the NestJS application

```bash
npm run start:dev
```

The API starts at `http://localhost:3000`. The Prisma module connects to PostgreSQL when Nest initializes.

## Useful commands

| Command | Purpose |
|---|---|
| `npm run start:dev` | Run NestJS with watch mode |
| `npx prisma validate` | Validate the Prisma schema |
| `npx prisma migrate dev --name init` | Create/apply a development migration |
| `npx prisma db seed` | Run the seed script |
| `npx prisma studio` | Browse database records |
| `npx prisma generate` | Generate Prisma Client |

## Before submitting

- Confirm `api/prisma/schema.prisma` contains `Role`, `User`, `Genre`, `Book`, and the optional `Review`.
- Confirm the migration was created and committed.
- Run `npx prisma validate`.
- Run `npx prisma db seed`.
- Confirm there are at least four genres and sixteen books in Prisma Studio.
- Confirm `api/.env` is not tracked by Git.
- Commit the code, migration files, and README. Never commit real database credentials.
