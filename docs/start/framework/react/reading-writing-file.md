---
id: reading-and-writing-file
title: Building a Full Stack DevJokes App with TanStack Start
---

This tutorial will guide you through building a complete full-stack application using TanStack Start. You'll create a DevJokes app that demonstrates key concepts of TanStack Start.

## What You'll Learn

Part 1:
- Implementing server functions 
- Reading and writing data to file 
- Building a complete UI with React components

<!-- Part 2: 
- Implementing server functions 
- Reading and writing data to file 
- Building a complete UI with React components

Part 3: 
<TBD>  -->


## Prerequisites

- Basic knowledge of React and TypeScript. 
- Node.js and `pnpm` installed on your machine
- Familiarity with TanStack Start basics (see [Quick Start Guide](/start/framework/react/quick-start))
- Server side Rendering. Here's a resource - 
- Client side hydration

## Nice to know
- React Query
- TanStack Router

## Settinng up a TanStack Start Project

First, let's create a new TanStack Start project:

```bash
pnpx create-start-app devjokes

cd devjokes
```
When this script is run, it will ask you a few setup questions. Pick a choice that works for you or just hit enter to accept the defaults.

Optionally, you can pass in a `--add-on` flag to get options such as Shadcn, Clerk , Convex, TansTack Query, etc.

Once setup, open your code editor and start building using the following command - 

```bash
pnpm i
pnpm dev
```


## Understand the project structure
At this point, the project structure should look like this -

```
/devjokes
├── src/
│   ├── routes/
│   │   ├── __root.tsx                    # Root layout
│   │   ├── index.tsx                     # Home page
│   │   ├── demo.start.server-funcs.tsx   # Demo server functions
│   │   └── demo.start.api-request.tsx    # Demo API request
│   ├── api/                              # API endpoints
│   ├── components/                       # React components
│   ├── api.ts                            # API handler.
│   ├── client.tsx                        # Client entry point
│   ├── router.tsx                        # Router configuration
│   ├── routeTree.gen.ts                  # Generated route tree
│   ├── ssr.tsx                           # Server-side rendering
│   └── styles.css                        # Global styles
├── public/                               # Static assets
├── app.config.ts                         # TanStack Start configuration
├── package.json                          # Project dependencies
└── tsconfig.json                         # TypeScript configuration
```

<TODO>

This may feel overwhelming, but here are the main files you need to focus on:
1. router.tsx
1. `src/routes/__root.tsx` - The root layout component. This is where you can add global styles, scripts, and other global components.
1. client.tsx
1. ssr.tsx


Once you have your project setup, you are ready to run your app on `localhost:3000`

At this point, your app will likely look like this - 

<INSERT PHOTO-1>


## Step 1: Setting Up File-Based Storage
At this point, we already have a demo server function ready to use. Let's go ahead and make it meaningful.

### Objectives
- Add the capability to read and write jokes to a JSON file
- Learn how to use `createServerFunction` 
- Create type-safe server functions for data management
- Building a responsive Jokes List component

### Implementation

#### Step 1.1: Create a JSON file with jokes
Let's setup a list of jokes that we can use to render on the page. Create a `data` directory in your project root and a `jokes.json` file within it:

```bash
mkdir -p src/data
touch src/data/jokes.json
echo "[{id: '1', content: 'Why did the programmer go broke?', punchline: 'Because he used up all his cache!'}]" > src/data/jokes.json
```

Then create a server function to read and write jokes to a JSON file:

```tsx
// app/server/jokes.ts
import { serverFn } from '@tanstack/react-start/server-functions'
import fs from 'node:fs/promises'
import path from 'node:path'

const JOKES_FILE = path.join(process.cwd(), 'data', 'jokes.json')

// Type for our joke data
export interface Joke {
  id: string
  content: string
  author: string
  createdAt: string
}

// Initialize jokes file if it doesn't exist
export const initJokesFile = serverFn(async () => {
  try {
    await fs.access(JOKES_FILE)
  } catch (error) {
    // File doesn't exist, create it with empty array
    await fs.mkdir(path.dirname(JOKES_FILE), { recursive: true })
    await fs.writeFile(JOKES_FILE, JSON.stringify([]))
  }
})

// Get all jokes
export const getJokes = serverFn(async (): Promise<Joke[]> => {
  await initJokesFile()
  const data = await fs.readFile(JOKES_FILE, 'utf-8')
  return JSON.parse(data)
})

// Add a new joke
export const addJoke = serverFn(async (joke: Omit<Joke, 'id' | 'createdAt'>): Promise<Joke> => {
  const jokes = await getJokes()
  
  const newJoke: Joke = {
    ...joke,
    id: Math.random().toString(36).substring(2, 9),
    createdAt: new Date().toISOString()
  }
  
  await fs.writeFile(JOKES_FILE, JSON.stringify([...jokes, newJoke]))
  
  return newJoke
})
```

## Step 2: Creating the UI Components

Next, let's create the UI for our jokes app.

Continue in the next section...
