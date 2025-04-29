---
id: reading-and-writing-file
title: Build a Full Stack Jokes App
---

# Building a Full Stack Jokes App with TanStack Start

This tutorial will guide you through building a complete full-stack application using TanStack Start. You'll create a jokes app that demonstrates key concepts for building modern web applications.

## What You'll Learn

- Implementing server functions for secure data operations
- Reading and writing data to files for persistent storage
- Making API calls to external services
- Creating and consuming API routes within your application
- Building a complete UI with React components

## Prerequisites

- Basic knowledge of React and TypeScript
- Node.js and pnpm installed on your machine
- Familiarity with TanStack Start basics (see [Quick Start Guide](/start/framework/react/quick-start))

## Project Setup

First, create a new TanStack Start project:

```bash
pnpx create-start-app your-app

cd your-app
```

## Step 1: Setting Up File-Based Storage

### Objectives
- Add the capability to read and write jokes to a JSON file
- Learn how to use `createServerFunction` for secure server-side operations
- Create type-safe server functions for data management
- Building a responsive joke list component

### Implementation

#### 1.1: Create a JSON file with jokes
Create a `data` directory in your project root and a `jokes.json` file within it:

```bash
mkdir -p data
touch data/jokes.json
echo "[{id: '1', content: 'Why did the programmer go broke?', punchline: 'Because he used up all his cache!'}]" > data/jokes.json
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
