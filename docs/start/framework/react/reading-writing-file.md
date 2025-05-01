---
id: reading-and-writing-file
title: Building a Full Stack DevJokes App with TanStack Start
---

This tutorial will guide you through building a complete full-stack application using TanStack Start. You'll create a DevJokes app that demonstrates key concepts of TanStack Start.

## What You'll Learn

**Part 1**:
1. Implementing server functions 
1. Reading and writing data to file 
1. Building a complete UI with React components

<!-- Part 2: 
- Implementing server functions 
- Reading and writing data to file 
- Building a complete UI with React components

Part 3: 
<TBD>  -->


## Pre-requisites

- Basic knowledge of React and TypeScript. 
- Node.js and `pnpm` installed on your machine
- Familiarity with TanStack Start basics (see [Quick Start Guide](/start/framework/react/quick-start))
- Server side Rendering. Here's a resource - <TODO>
- Client side hydration
- TanStack Router

## Nice to know
- React Query

## Setting up a TanStack Start Project

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

Let's implement this! 

#### Step 1.1: Create a JSON file with jokes
Let's setup a list of jokes that we can use to render on the page. Create a `data` directory in your project root and a `jokes.json` file within it:

```bash
mkdir -p src/data
touch src/data/jokes.json
echo "[\n    {\n      \"id\": 1,\n      \"question\": \"Why don't keyboards sleep?\",\n      \"answer\": \"Because they have two shifts\"\n    },\n    \n        {\n          \"id\": 2,\n          \"question\": \"Are you a RESTful API?\",\n          \"answer\": \"Because you GET my attention, PUT some love, POST the cutest smile, and DELETE my bad day\"\n        },\n        {\n          \"id\": 3,\n          \"question\": \"I used to know a joke about Java\",\n          \"answer\": \"But I run out of memory.\"\n        },\n        {\n          \"id\": 4,\n          \"question\": \"Why do Front-End Developers eat lunch alone?\",\n          \"answer\": \"Because, they don't know how to join tables.\"\n        },\n        {\n          \"id\": 5,\n          \"question\": \"I am declaring a war.\",\n          \"answer\": \"var war;\"\n        }\n      ]"  > src/data/jokes.json
```




Let's clean up our `App.jsx` file to remove the demo component. 


```jsx
import { createFileRoute } from "@tanstack/react-router";

export const Route = createFileRoute("/")({
  component: App,
});

function App() {
  return (
    <div className="p-4">
      <h1>DevJokes</h1>
    </div>
  );
}

```

This is where we will add our Jokes so that when the user lands on the home page, they can see the jokes.

  #### Step 1.2: Create Server Functions to Read the File

  Let's create a server function to perform a read-write operation. This will be used to read jokes from a JSON file and render jokes on the page. We will also create a server function to write a new  joke to the JSON file.
   The starter app already comes with a page called "Start Server Functions". We are going to take inspiration from this to write our own server functions. 
   
 Let's create a server function using [`createServerFunction`](https://tanstack.com/start/latest/docs/framework/react/server-functions). 



```tsx
import { createServerFn } from "@tanstack/react-start";
import * as fs from "node:fs";

interface Joke {
  id: string;
  question: string;
  answer: string;
}

interface JokesData {
  jokes: Joke[];
}
const JOKES_FILE = "./data/jokes.json";

const getJokes = createServerFn({ method: "GET" }).handler(async () => {
   const jokes = await fs.promises.readFile(JOKES_FILE, "utf-8");
  return JSON.parse(jokes) as JokesData;
});

```

In this code, we are using `createServerFn` to create a server function that reads the jokes from the JSON file. The `handler` function is where we are using the `fs` module to read the file. 

Now to consume this server function, we can simply call this in our code using TanStack Router which already comes with TanStack Start! 

```jsx
// App.jsx

function App() {
  const jokes = Route.useLoaderData() || [];

  return (
    <div className="p-4 flex flex-col">
      <h1 className="text-2xl">DevJokes</h1>
      <div>
        {jokes?.map((joke) => (
          <div key={joke?.id}>
            <p>{joke?.question}</p>
            <p>{joke?.answer}</p>
          </div>
        ))}
      </div>
    </div>
  );
}
```
When the page loads, `jokes` will have data from the jokes.json file already! 


With a little Tailwind styling, the app should look like this: 
<TODO: ADD TAILWIND>

<INSERT PHOTO-3>


<EXPLANATION OF WHAT HAPPENED HERE>



## Step 2: DOES THIS WORK? 

Next, let's create the UI for our jokes app.

Continue in the next section...
