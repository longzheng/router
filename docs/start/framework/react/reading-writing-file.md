---
id: reading-and-writing-file
title: Building a Full Stack DevJokes App with TanStack Start
---

This tutorial will guide you through building a complete full-stack application using TanStack Start. You'll create a DevJokesapp where users can view and add developer-themed jokes, demonstrating key concepts of TanStack Start including server functions, file-based data storage, and React components.

## What You'll Learn
1. Setting up a TanStack Start project
2. Implementing server functions
3. Reading and writing data to files
4. Building a complete UI with React components
5. Using TanStack Router for data fetching and navigation


## Prerequisites

- Basic knowledge of React and TypeScript. 
- Node.js and `pnpm` installed on your machine


## Nice to know
- Server-side rendering <TODO ADD LINK>
- Client-side hydration <TODO ADD LINK>
- TanStack Router concepts<TODO ADD LINK>
- React Query concepts<TODO ADD LINK>

## Setting up a TanStack Start Project

First, let's create a new TanStack Start project:

```bash
pnpx create-start-app devjokes
cd devjokes
```
When this script runs, it will ask you a few setup questions. You can either pick choices that work for you or just press enter to accept the defaults.

Optionally, you can pass in a `--add-on` flag to get options such as Shadcn, Clerk, Convex, TanStack Query, etc.

Once setup is complete, install dependencies and start the development server:

```bash
pnpm i
pnpm dev
```


## Understanding the project structure

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

This structure might seem overwhelming at first, but here are the key files you need to focus on:
1. `router.tsx` - Sets up routing for your application
2. `src/routes/__root.tsx` - The root layout component where you can add global styles and components
3. `src/routes/index.tsx` - Your home page
4. `client.tsx` - Client-side entry point
5. `ssr.tsx` - Handles server-side rendering


Once your project is set up, you can access your app at `localhost:3000`. You should see the default TanStack Start welcome page.

At this point, your app will likely look like this - 

<INSERT PHOTO-1>


## Step 1: Setting Up File-Based Storage and Server Functions

Let's start by creating a file-based storage system for our jokes and implementing server functions to interact with this data.

### Step 1.1: Create a JSON File with Jokes
Let's set up a list of jokes that we can use to render on the page. Create a `data` directory in your project root and a `jokes.json` file within it:

```bash
mkdir -p src/data
touch src/data/jokes.json
```

Now, add some sample jokes to this file:

```json
[
  {
    "id": "1",
    "question": "Why don't keyboards sleep?",
    "answer": "Because they have two shifts"
  },
  {
    "id": "2",
    "question": "Are you a RESTful API?",
    "answer": "Because you GET my attention, PUT some love, POST the cutest smile, and DELETE my bad day"
  },
  {
    "id": "3",
    "question": "I used to know a joke about Java",
    "answer": "But I ran out of memory."
  },
  {
    "id": "4",
    "question": "Why do Front-End Developers eat lunch alone?",
    "answer": "Because, they don't know how to join tables."
  },
  {
    "id": "5",
    "question": "I am declaring a war.",
    "answer": "var war;"
  }
]
```

### Step 1.2: Create Types for Our Data
Let's create a file to define our data types. Create a new file at `src/types/index.ts`:

```typescript
// src/types/index.ts
export interface Joke {
  id: string;
  question: string;
  answer: string;
}

export type JokesData = Joke[];
```

  ### Step 1.3: Create Server Functions to Read the File

  Let's create a server function to perform a read-write operation. We will be using [`createServerFn`](https://tanstack.com/start/latest/docs/framework/react/server-functions) to create server functions that run on the server


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
const JOKES_FILE = "src/data/jokes.json";

const getJokes = createServerFn({ method: "GET" }).handler(async () => {
   const jokes = await fs.promises.readFile(JOKES_FILE, "utf-8");
  return JSON.parse(jokes) as JokesData;
});

```

In this code, we are using `createServerFn` to create a server function that reads the jokes from the JSON file. The `handler` function is where we are using the `fs` module to read the file. 


### Step 1.4: Create Server Functions to Read and Write Jokes

Now to consume this server function, we can simply call this in our code using TanStack Router which already comes with TanStack Start! 
Let's modify our `App.jsx` file to remove the demo code and add a `JokesList` component. 

```jsx
import { createFileRoute } from "@tanstack/react-router";
import { JokesList } from "./JokesList";

export const Route = createFileRoute("/")({
  component: App,
});

function App() {
  return (
    <div className="p-4">
      <h1>DevJokes</h1>
      <JokesList />
    </div>
  );
}

```

The `JokesList` component accepts a list of jokes and renders on the page.

```tsx
// src/components/JokesList.tsx
import { Joke } from "../types";

interface JokesListProps {
  jokes: Joke[];
}

export function JokesList({ jokes }: JokesListProps) {
  if (!jokes || jokes.length === 0) {
    return <p className="text-gray-500 italic">No jokes found. Add some!</p>;
  }

  return (
    <div className="space-y-4">
      <h2 className="text-xl font-semibold">Jokes Collection</h2>
      {jokes.map((joke) => (
        <div 
          key={joke.id} 
          className="bg-white p-4 rounded-lg shadow-md border border-gray-200"
        >
          <p className="font-bold text-lg mb-2">{joke.question}</p>
          <p className="text-gray-700">{joke.answer}</p>
        </div>
      ))}
    </div>
  );
}
```

Now let's call our server function inside `App.jsx` using TanStack Router. 

```jsx
// App.jsx
import { createFileRoute } from "@tanstack/react-router";
import { getJokes } from "./serverActions/jokesActions";
import { JokesList } from "./JokesList";

export const Route = createFileRoute("/")({
  loader: async () => {
    // Load jokes data when the route is accessed
    return getJokes();
  },
  component: App,
});

const App = () => {
  const jokes = Route.useLoaderData() || [];

  return (
    <div className="p-4 flex flex-col">
      <h1 className="text-2xl">DevJokes</h1>
      <JokesList jokes={jokes} />
    </div>
  );
};
```
When the page loads, `jokes` will have data from the jokes.json file already! 


With a little Tailwind styling, the app should look like this: 

<INSERT PHOTO-3>


## Step 2: Writing Data to a File

We have been able to read from the file successfully! We can use the same approach to write to the `jokes.json` file using `createServerFunction`.

Let's create another server function but this time with a `POST` method to write to the same file. 

```tsx
const addJokes = createServerFn({ method: "POST" })
  .validator((joke: Joke) => joke)
  .handler(async ({ data }) => {
    if (!data.question || !data.answer) return;

       try {
      // Read the existing jokes from the file
      const jokesData = await getJokes();

      // Create a new joke with a unique ID
      const newJoke: Joke = {
        id: uuidv4(),
        question: data.question,
        answer: data.answer,
      };

      // Add the new joke to the list
      const updatedJokes = [...jokesData, newJoke];
      
      // Write the updated jokes back to the file
      await fs.promises.writeFile(
        JOKES_FILE,
        JSON.stringify(updatedJokes, null, 2),
        "utf-8"
      );

      return newJoke;
    } catch (error) {
      console.error("Failed to add joke:", error);
      throw new Error("Failed to add joke");
    }
  });

```

Make sure to install the UUID package:

```bash
pnpm add uuid
pnpm add -D @types/uuid
```

In this code: 
- We are going to first use `validator` to validate the input data. This is a good practice to ensure that the data we are receiving is in the correct format. We are then going to use the `handler` function to write the data to the file. 
- We are using `createServerFn` to create server functions that run on the server. `getJokes` reads the jokes from our JSON file. `addJoke` validates the input data and adds a new joke to our file. We're using `uuidv4()` to generate unique IDs for our jokes.


## Step 2: Setting Up Our Home Page with TanStack Router

Now, let's modify our home page to display jokes and provide a form to add new ones. Let's create a new component called `JokeForm.jsx` and add the following form to it:

```tsx
// src/components/JokeForm.tsx
import { useState } from "react";
import { useRouter } from "@tanstack/react-router";
import { addJoke } from "../serverActions/jokesActions";

export function JokeForm() {
  const router = useRouter();
  const [question, setQuestion] = useState("");
  const [answer, setAnswer] = useState("");
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault(); // Prevent default form submission
    
    if (!question || !answer) {
      setError("Both question and answer are required");
      return;
    }
    
    if (isSubmitting) return;
    
    try {
      setIsSubmitting(true);
      setError(null);
      
      await addJoke({
        data: { question, answer },
      });

      // Clear form
      setQuestion("");
      setAnswer("");

      // Refresh data
      router.invalidate();
    } catch (error) {
      console.error("Failed to add joke:", error);
      setError("Failed to add joke. Please try again.");
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="mb-8">
      <h2 className="text-xl font-semibold mb-4">Add New Joke</h2>
      
      {error && (
        <div className="bg-red-100 text-red-700 p-2 rounded mb-4">
          {error}
        </div>
      )}
      
      <div className="space-y-4">
        <div>
          <label htmlFor="question" className="block text-sm font-medium text-gray-700 mb-1">
            Question
          </label>
          <input
            id="question"
            type="text"
            placeholder="Enter joke question"
            className="w-full p-2 border rounded focus:ring focus:ring-blue-300"
            value={question}
            onChange={(e) => setQuestion(e.target.value)}
            required
          />
        </div>
        
        <div>
          <label htmlFor="answer" className="block text-sm font-medium text-gray-700 mb-1">
            Answer
          </label>
          <input
            id="answer"
            type="text"
            placeholder="Enter joke answer"
            className="w-full p-2 border rounded focus:ring focus:ring-blue-300"
            value={answer}
            onChange={(e) => setAnswer(e.target.value)}
            required
          />
        </div>
        
        <button
          type="submit"
          disabled={isSubmitting}
          className="bg-blue-500 hover:bg-blue-600 text-white font-medium py-2 px-4 rounded disabled:opacity-50"
        >
          {isSubmitting ? "Adding..." : "Add Joke"}
        </button>
      </div>
    </form>
  );
}
```

With this, our UI should look like this: 
![alt text](PHOTO-3.jpeg)

Now, let's add the functionality to submit the form and add a new joke to the file. We will use the `addJokes` server function to add a new joke to the file.

```ts
  export const addJoke = createServerFn({ method: "POST" })
    .validator((joke: { question: string; answer: string }) => joke)
    .handler(async ({ data }) => {
      if (!data?.question || !data?.answer) return;
  
      try {
      // Read the existing jokes from the file
      const jokesData = await getJokes();

      // Add the new joke to the list
      const updatedJokes = {
        jokes: [
          ...jokesData,
          {
            id: uuidv4(),
            question: data.question,
            answer: data.answer,
          },
        ],
      };
      
  
      // Write the updated jokes back to the file
      await fs.promises.writeFile(
        JOKES_FILE,
        JSON.stringify(updatedJokes, null, 2),
        "utf-8"
      );
    } catch (error) {
        throw new Error("Failed to add joke", { cause: error });
    }
    });
```

And finally, we just have to call this server action in our form and wire everything together. 

```tsx
//JokeForm.tsx 
import { useState } from "react";
import { addJoke } from "../serverActions/jokesActions";
import { useRouter } from "@tanstack/react-router";

export function JokeForm() {
  const router = useRouter();
  const [question, setQuestion] = useState("");
  const [answer, setAnswer] = useState("");
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleSubmit = async () => {
    if (!question || !answer || isSubmitting) return;
    try {
      setIsSubmitting(true);
      await addJoke({
        data: { question, answer },
      });

      // Clear form
      setQuestion("");
      setAnswer("");

      // Refresh data
      router.invalidate();
    } catch (error) {
      console.error("Failed to add joke:", error);
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form className="flex flex-row gap-2 mb-6">
      <input
        type="text"
        name="question"
        placeholder="Question"
        className="p-1 border rounded w-full"
        required
        onChange={(e) => setQuestion(e.target.value)}
        value={question}
      />
      <input
        type="text"
        name="answer"
        placeholder="Answer"
        className="p-1 border rounded w-full"
        required
        onChange={(e) => setAnswer(e.target.value)}
        value={answer}
      />
      <button
        className="bg-blue-500 text-white p-1 rounded hover:bg-blue-600"
        onClick={handleSubmit}
        disabled={isSubmitting}
      >
        Add
      </button>
    </form>
  );
}


```

## Understanding How It All Works Together

Let's break down how the different parts of our application work together:

1. **Server Functions**: These run on the server and handle data operations
   - `getJokes`: Reads the jokes from our JSON file
   - `addJoke`: Adds a new joke to our JSON file

2. **TanStack Router**: Handles routing and data loading
   - The loader function fetches jokes data when the route is accessed
   - `useLoaderData` makes this data available in our component
   - `router.invalidate()` refreshes the data when we add a new joke

3. **React Components**: Build the UI of our application
   - `JokesList`: Displays the list of jokes
   - `JokeForm`: Provides a form for adding new jokes

4. **File-Based Storage**: Stores our jokes in a JSON file
   - Reading and writing are handled by Node.js `fs` module
   - Data is persisted between server restarts



## How Data Flows Through the Application

When a user visits the home page:
1. The `loader` function in the route calls `getJokes()` server function
2. The server reads `jokes.json` and returns the jokes data
3. This data is passed to the `HomePage` component through `useLoaderData()`
4. The `HomePage` component passes the data to the `JokesList` component

When a user adds a new joke:
1. They fill out the form and submit it
2. The `handleSubmit` function calls the `addJoke()` server function
3. The server reads the current jokes, adds the new joke, and writes the updated data back to `jokes.json`
4. After the operation completes, we call `router.invalidate()` to refresh the data
5. This triggers the loader again, fetching the updated jokes
6. The UI updates to show the new joke in the list


Here's a demo of the app in action: 
<TODO: INSERT VIDEO>


## Conclusion

Congratulations! You've built a full-stack DevJokes app using TanStack Start. In this tutorial, you've learned:

- How to set up a TanStack Start project
- How to implement server functions for data operations
- How to read and write data to files
- How to build React components for your UI
- How to use TanStack Router for routing and data fetching

This simple application demonstrates the power of TanStack Start for building full-stack applications with a minimal amount of code. You can extend this app by adding features like:

- Joke categories
- Ability to edit and delete jokes
- User authentication
- Voting for favorite jokes

The complete code for this tutorial is available on [GitHub](https://github.com/shrutikapoor08/devjokes).


## Troubleshooting Common Issues

### File Paths
If you encounter issues with file paths, make sure the `JOKES_FILE` path in `jokesActions.ts` is correct relative to where your server is running.

### Server Functions Not Working
Ensure that you've configured TanStack Start correctly. Check the `app.config.ts` file to ensure server functions are enabled.

### Data Not Refreshing
If the UI doesn't update after adding a joke, make sure you're calling `router.invalidate()` after the operation completes.

### TypeScript Errors
Check that your types are consistent across the application. The `Joke` interface should be the same in all files.
