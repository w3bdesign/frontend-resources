# Lessons from 10 Years of React Development

## Introduction

Hi everybody. I am Corey House. I'm from the States, right in the middle of the states in a city called Kansas City where they have great barbecue, lots of developers, and a big developer conference called KCDC. I've been going there for years—it was actually where I spoke for the very first time. As a side note, if you've never spoken before, I'd encourage you to consider it. It totally changed my life and has been a whole lot of fun.

What keeps me busy is React. I've been doing React since it was open sourced—you have to go way back in time. React is over 10 years old now, which is why I titled this "Lessons from 10 Years." I've been using it since they announced it. Like immediately I saw it, I was working in Angular and Knockout at the time and I thought, "This is really compelling." It turned out that was right. Though I said the same thing about Silverlight, so I'm not always right, but these things happen.

## The Future is Unevenly Distributed

The theme of the beginning of this talk is around the future and this old idea I see all the time: the future is unevenly distributed. I feel that all the time. As a person from the States, I come over here and notice you all have a lot more electric cars than the Midwest for sure. They're all over. In fact, the last time I was in Copenhagen, the city feels quieter to me than it did before because there are more electrics. Maybe I'm making it up, but I rode around all last night for about an hour and a half just seeing the city, and it's really fun. It feels like a glimpse of the future that's coming to the States at some point.

I bring this up because as a React developer, I get to live in the future. The reason I get to live in the future is that I'm an independent. I spend my time working with a bunch of different companies, helping them be more effective in React. I don't have to ask anybody's permission when something new comes out—I just go build something with it, learn it on my own, and then help other teams adopt it. I also suggest things based on all the different companies I work with.

## The Evolution of Data Fetching in React

Because of that, over the last 10 years, what I've done for a simple idea like data fetching just continues to pivot, partially for things I cannot control.

### The Class Component Era

If we go back to the beginning, some of you may be old enough to remember that when React first came out, you had to use a class and put a fetch within `componentDidMount`. This worked fine and in some ways felt pretty easy compared to our friend `useEffect`.

### The Hooks Revolution

A few years later, hooks came out. To this day, as somebody who works in React regularly and teaches teams, `useEffect` is probably the biggest footgun. There are so many examples of problems with it, and it has become a lightning rod for everyone to dunk on React because the `useEffect` API is unfortunate in various ways. It's powerful but also written in a way that lends itself to overuse. Having to manage the dependency array is annoying.

When hooks came out in 2019, people weren't so sure about them. Still to this day, many people look at `useEffect` and think, "This might be the worst part of React."

### Creating Custom Solutions

Because of that, pretty quickly I wanted to avoid starting from scratch. I didn't want to work with it directly. A year after hooks came out, I created a custom hook called `useFetch`. I imagine many of you did something similar.

Let me show you an example of what I'm talking about. I have a pinned tweet about lessons from state with a very satisfying animation. This was the sort of thing many people were doing when `useEffect` first came out—look at all the boilerplate:

- State for holding the data we fetched
- State for loading
- State for errors
- The `useEffect` declaration to run the code once

Inside the effect, I chose to use promises, but I could have used async/await. However, if you used async/await in this example, the moment you try to decorate the anonymous arrow inside `useEffect`, you get an error because React requires that to be synchronous. You'd have to declare a function and then invoke it inside.

Other things you need to remember:
- Declare your catch block
- Capture errors because errors within async code are not caught by error boundaries
- Errors within click handlers or event handlers are also not caught by error boundaries
- Put `loading = false` in the finally block
- Display loading indicators
- Throw errors in the root so they bubble to error boundaries
- Handle the happy path

Even this implementation isn't perfect—there are still potential bugs.

### The Custom Hook Solution

I created my own hook that converted all that complexity into something much cleaner. The code still exists, but now it's in a custom hook and genericized. I can't mess it up anymore. I'm basically forced to remember to handle loading state and error state, and I already have state for the data I've fetched back. This was a really nice improvement.

### React Query Changes Everything

But that was back in 2020. Fast forward a year later, and this wonderful library called React Query came out. React Query was basically that `useFetch` hook I made, but good—really good. Mine was okay, but React Query added so much that I hadn't even considered.

## The Power of Good Abstractions

A good abstraction creates a pit of success. A good abstraction does things that I might forget. React Query (now TanStack Query) provides several benefits:

### Caching
Often when you get requirements for your project, your PM doesn't think about caching. It's a geeky thing, but it's important. Having caching can improve performance. When you use React Query, it requires you to declare a cache key so you can cache requests. When somebody comes back to your product list page, if they've already been there, it's instant because that cache is held in the browser.

### Automatic Refetching
- When you lose connection (like driving through a tunnel) and come back out, it automatically makes a request and shows you the latest data
- When you come back on Monday to a tab you loaded on Friday, it checks for fresh data behind the scenes and shows you new data

### Built-in Features
- Retries happen automatically
- You're probably not hearing requests for retries from project managers, but they're important and improve user experience

All these things are built into React Query, which is why I was really excited when it came out.

## Error Boundaries: A Critical Gap

One year later, I was still using React Query, but then I started using a particular feature: `useErrorBoundary`. React has had error boundaries for a long time, and if there's one lesson that's particularly concerning for me from 10 years in React, it's this:

I audit React codebases all the time. People show me their codebase and ask for feedback. Usually that audit contains 10 to 12 pages of suggestions because I've looked at enough React projects that I can immediately spot issues. I'm not just looking at React—I'm examining performance, security, and more.

What concerns me is that most React applications I look at still have basically no error handling. If an error occurs in React and you don't have an error boundary, what is the user experience? It's a crash. When we say crash, the user sees...

[Note: The transcription appears to cut off mid-sentence here]

## Error Boundaries Implementation Strategy

### Root-Level Error Boundaries
Experience is a white screen - users see nothing. It's just "I was looking at an app and now it's disappeared." It's very important to have an error boundary at the root. That's a given.

### Granular Error Boundaries
The other big opportunity is that I very rarely see people putting error boundaries at granular levels. It's really useful to have an error boundary around every one of your pages.

Imagine if one page fails - I would really like to still be able to see the navigation, the footer, maybe see a sidebar with subnavigation. If I'm looking at a page that has a bunch of widgets - weather, to-do app, and task manager - it'd be nice if every one of those had an error boundary around them.

Imagine if the weather fails - I would like to just see a little message that we couldn't fetch the weather, but I can still use the rest of the app. I literally had one of my clients who had a production outage because the weather API that they hit went down (third party). They had an outage and didn't have an error boundary around it. The whole app just goes white. Not good.

## TanStack Query Error Handling

### Throw on Error Configuration
Why do you want to have use error boundary enabled here? This was renamed fairly recently to "throw on error." Remember the gymnastics I showed you earlier? You had to set the error state at the top and rethrow that error at the bottom of the screen because you need to throw it outside of asynchronous code so that an error boundary will actually catch it. It's an unfortunate fact in React.

The great news is if you are using TanStack Query today and you tell it to throw on error, it will do exactly that. Here's the setting - nothing fancy, one little boolean. I'm telling it that I want it to throw on error if there is an error in a query or an error in a mutation.

### Benefits of Automatic Error Throwing
With that little change, my app code has error handling, but notice that I'm not doing anything - I can't mess it up. I'm a really big fan of APIs that I can't mess up because that's what the API has me do.

If anything fails - my query, my fetch to this URL, if it's not a 200 - it will throw on my behalf. That async problem I was talking about just goes away. This will bubble up to the nearest error boundary, which sits in my root.

## Runtime Data Validation

### TypeScript Limitations at Runtime
Here's a quiz for the room: Who is unfortunately still working in JavaScript, not TypeScript? TypeScript is now nearly 100% with my clients. But here's the key question: Does TypeScript exist at runtime in the browser? No, it does not.

If I want to make sure that some runtime data meets my types, TypeScript cannot help me. Think about where we get runtime data from:
- Form inputs from users
- JSON APIs (maybe written in C, Java, Ruby)

At any given point, an API could return different JSON and my TypeScript types will not be compatible with that JSON I received.

### Using Ky for HTTP Requests
Notice I'm not using fetch - I am using Ky. Ky is an open-source alternative to Axios. The difference:
- **Ky**: Built as an abstraction over fetch
- **Axios**: Abstraction over XMLHttpRequest

This distinction is important because Ky is lighter since fetch is just a lighter abstraction. Axios is more powerful, but in my experience, you rarely need the extra power of XMLHttpRequest unless you need progress tracking.

What I like about Ky is that it returns the type as `unknown`, so I'm forced to type that properly rather than it being typed as `any`.

## Zod for Runtime Validation

### What is Zod?
Zod is an open-source library with a confusing description: "TypeScript-first schema validation with static type inference." What it actually is: **runtime types**. At runtime, I want to validate my data.

When I get JSON back from some random server, I want to validate that:
- Name is a string
- Email is an email
- URL truly is a URL

### Zod Implementation Example
```javascript
const foodSchema = z.object({
  name: z.string(),
  price: z.number(),
  description: z.string(),
  image: z.string(),
  tags: z.array(z.string())
});
```

This means when I get JSON from an API, I can run it through this schema. If it doesn't meet my requirements, it will fail loudly and tell me exactly what's wrong.

### The Importance of Failing Fast
As developers, one thing that's super helpful is failing fast. I recently worked with a client who paid a third party $3 million to build them a React application. The agency was learning React on the fly and created a mess - they were about 80% done with a whole lot of stuff, but not 100% done with anything.

They were using TypeScript but had 1,350 `any` types in the codebase, so using TypeScript was near zero benefit. We successfully got the project to production by fixing it along the way.

### Zod in Production
When we put Zod into the codebase and got feedback from customers in staging, the client called and said, "I think we need to remove Zod because we're getting errors from Zod in staging."

# Error Handling and Data Validation

You need to understand that when Zod tells you there's an error, it's actually providing valuable feedback. We could ignore errors by turning off error monitoring, just like we could ignore flu symptoms. But when Zod tells you that you received unexpected JSON, it means our code isn't properly aligned with our APIs. This requires fixing.

## Common Mistake: Being Too Strict

A frequent mistake when describing runtime data is being overly strict with validation. For example, if you specify that a name must have a minimum of one character, but your database sometimes contains empty strings, your validation will fail. 

It's important to:
- Consult with your DBA
- Query the database directly
- Examine production data
- Be cautious about overly strict validation

Remember, when Zod throws an error, you need to decide how to handle it. Despite this learning curve, Zod has been an excellent tool that I've enjoyed using.

# Evolution of Data Fetching (2023): Suspense Query

Moving into 2023, I began using `useSuspenseQuery`. This approach is somewhat divisive because of how suspense is typically used.

## Suspense Usage Patterns

Most developers have used suspense for lazy loading pages, but very few have used it for data fetching. Among those who do use suspense for data fetching, most are working with Next.js.

Suspense for data fetching is stable if you're working with:
- Next.js
- Other supporting libraries (though alternatives like TanStack Start or React Router's RSC support aren't 1.0 yet)

## Implementation Example

Here's how suspense for data fetching works in practice:

In the root component, I wrap the outlet with a suspense boundary:
```jsx
<Suspense fallback="Loading...">
  <Outlet />
</Suspense>
```

This means if any child components loading into the outlet are fetching data, it will fall back to the loading state.

On individual pages, I use `useSuspenseQuery` on line 9, which returns a promise. Since it returns a promise, it triggers the suspense boundary. Notice there's no loading logic in the component itself – it's handled compositionally in the parent.

## Broad Compatibility

Everything discussed up to this point works with:
- Recent versions of React (doesn't require React 19)
- React Router
- TanStack Query
- Next.js
- Other modern React setups

This broad applicability makes these patterns very valuable.

# The Bleeding Edge (2024): React Server Components

Now we enter more experimental territory. I call this "bleeding edge" because cutting-edge technology can be painful to work with.

## Current State of Adoption

React Server Components have been available for a while and stable in Next.js, with experimental implementations in other frameworks. However, there's a notable love-hate relationship with them, and often the hate seems to be winning.

At developer conferences, only about 20% of attendees have even tried React Server Components, despite being available since 2024. Even fewer are using Next.js with the app router, which is required for full RSC support.

## The Politics Behind the Technology

Much of the criticism around React Server Components may stem from optics rather than technical issues. Vercel hired React team members and helped steward React development, which some view as self-serving since RSCs benefit companies selling hosting services.

While there's some truth to this concern, there's also unfair criticism of Vercel's contributions.

## Compelling API Example

Despite the controversy, the core API is compelling:

```jsx
// Page component with suspense boundary
<Suspense fallback={<Loading />}>
  <FetchTodos />
</Suspense>

// Async server component (only possible on server)
async function FetchTodos() {
  const response = await fetch('/api/todos');
  const data = zodSchema.parse(await response.json()); // Zod validation
  return <TodoList todos={data} />;
}
```

## Performance Benefits

React Server Components become especially compelling when dealing with heavy dependencies. For example, using dayjs and localization libraries:

```jsx
import dayjs from 'dayjs';
import localizedFormat from 'dayjs/plugin/localizedFormat';
```

These heavy dependencies never reach the browser – only the resulting string with the server's timestamp is sent to the client.

### Real-World Use Case

I'm currently working with a client struggling with a chart containing 30,000 data points. They're downloading all that JSON and rendering it client-side, which isn't performant. With React Server Components, we could:
- Render the graph on the server
- Send only the resulting image to the client
- Eliminate the need to download the chart library and massive JSON payload

This demonstrates the efficiency gains possible with server-side rendering.

# The Future: Sync Engines

The most exciting development I want to discuss is sync engines. I recently gave an entire talk on this topic because I've spent months diving deep into this area.

## Current Status

Sync engines are compelling but cutting-edge – you probably won't be able to use them at your job in the next few months. However, they represent an important trend in application development.

## Local-First Applications

Sync engines are closely related to local-first application concepts. To demonstrate this, I've built a project for managing a fictitious car dealership's inventory.

The application showcases how sync engines can enable:
- Offline functionality
- Real-time synchronization across multiple clients
- Improved user experience through local data availability

This represents the next evolution in how we think about data management and synchronization in modern applications.

# React State Management and Sync Engines

## Real-Time Data Synchronization Demo

Here we're looking at two windows - a list of cars on top and a form to add a new vehicle on the bottom. Imagine the left tab is my iPhone and the right tab is your iPhone, and we're both at work today.

When I negotiate a deal for this 300ZX Turbo and mark it as sold by clicking save, watch the left browser. The moment I hit save, it immediately gets marked as sold on both screens. That's a sync engine in a nutshell.

The core idea is compelling: What if your React application always showed the latest data from the database the very moment that a SQL commit occurred? 

This may not be a big deal if your application has infrequent reads or minimal collaboration. However, many of us are writing apps where multiple users might be looking at the same record simultaneously. When one person edits it, others should know right away.

## Traditional vs. Modern Approaches

### Polling Solution
One simple way to achieve this is through polling - writing a loop that makes an Ajax call every 5 seconds to check for the latest data.

### Sync Engine Solution
Instead, I'm using a sync engine called Convex. Convex is just one of many options available. You could also look at:
- Zero
- Jazz
- Electric SQL
- And many others

I chose Convex because it's actually stable, while many other solutions are nearly stable but not quite ready.

## Why You Might Not Be Able to Use This Yet

Convex is selling you a hosted database - it's a schemaless (or schema-optional) database. When you fire this up, they're hosting the database and handling the synchronization.

## Offline-First Functionality

Let me demonstrate something interesting. I'll turn off Wi-Fi and change this car from "sold" back to "on sale", then hit save.

Notice it says "vehicle updated" even though I'm not online. It wrote locally. The left tab is now out of sync because this person just saved data while offline.

When I reconnect to Wi-Fi, the synchronization should happen automatically. After reconnecting, you can see the data syncs across both tabs.

## Local-First Development Benefits

### Performance Advantages
When you write locally - whether to IndexedDB in the browser, memory, or even SQLite via WASM in the browser - every interaction with your application can be instantaneous. You're writing to your local machine, so it doesn't matter if the network call takes 5 or 10 seconds.

This approach:
- Writes to memory locally
- Holds the data
- Eventually sends it up to the server

Even on slow 3G connections, changes save instantly because they're writing locally. The sync engine automatically keeps tabs in sync locally, even across different browsers.

## Developer Experience Benefits

### Simplified Code Structure
Sync engines aren't just about UX (performance, offline syncing, collaboration, conflict reduction) - they're also about developer experience.

Here's the code for my app - line 20 shows my code for getting vehicles: just one line of code.

The actual API logic lives in a Convex folder that only runs on the server. This TypeScript never reaches the client. I'm effectively creating REST-like API endpoints:
- An endpoint for getting a list
- An endpoint for deleting a vehicle  
- An endpoint for adding a vehicle

The code is minimal - I specify what arguments can be passed in, their types, and what should happen inside.

### Built-in Features
What you get for free:
- Optimistic UI
- Always fast since writing locally
- Offline capability with eventual sync
- Real-time collaboration

## The Broader Landscape

I haven't been this excited about a technology in quite a while. I'm not necessarily saying "go with Convex" specifically, although they are stable and well-designed. There are many other options available.

### Local First Podcast
I recommend the Local First podcast by Johannes Schickling. He's been interviewing various people in this space, and I'll be appearing on it soon after he saw my slides from a recent talk.

The podcast's landscape tab gives you a sense of how vast this space is. I've spent months digging through all these options and still haven't tried everything - and this isn't even an exhaustive list.

### My Prediction
In the next few years, sync engines are what we're going to be using. I'm actually more excited about this than React Server Components, and it's the first thing I've seen that I prefer over TanStack Query.

## TanStack DB: A Practical Option

TanStack DB recently emerged as another sync engine solution. The beauty is you could use this today with your existing setup:
- Works with REST APIs
- Works with GraphQL APIs
- No backend changes required
- No API changes needed
- Works with any database

TanStack DB is being developed in partnership with Electric SQL, an interesting project by one of Gatsby's creators. It's a stable, lower-level sync engine option.

## Understanding Remote State

I've focused heavily on data fetching because this relates to "remote state" - data retrieved from a server. In my experience, 90-something percent of the data in your application is fetched from a server.

When you get remote state management right, most of your application falls into place. Many React problems disappear because you end up primarily using React for:
- JSX for creating components
- The simple component API
- The vast ecosystem of components

## React State Categories

There are over 30 different state management approaches for React. I think about React state in terms of eight categories, which I cover in detail in my five-hour Pluralsight course.

I deliberately put URL at the top because people often underutilize the URL. It's easy to miss this powerful state management tool.

For example, one of my current clients has around 25 pages in their application but only three routes. They have three top-level routes and use state to decide what sub-routes to show, which is unfortunate since the URL could handle much of this routing logic.

# State Management for React Developers

## URL-Based State Management

You should reflect the whole thing. You should be able to share that URL with others. Be very careful about underutilizing URL-based state, as React developers often overlook this powerful option.

## Web Storage Options

As a React developer, it's easy to forget about web storage. Web storage is great, and local storage is often a useful tool.

## Local State Best Practices

Local state is a good default. When referring to local state, this means using `useState`. 

**Key Tip:** Try to keep state as local as possible. When declaring `useState`, put it in a single component and pass it down via props for a while. Lift it to a parent only if you need to share it with two siblings, but avoid moving it way up the tree. The higher you have state, the worse your performance gets and the more you have to drill props through components. Default to local state and only lift when necessary.

## Lifted State

Lift to the nearest parent when you have to. This isn't complicated - technically, it's just a cut and paste operation.

## Derived State

Derived state is key. Sometimes developers make the mistake of creating another piece of state that should be derived from existing state.

### Example: Vehicle Form
If you have `newVehicle` and `setNewVehicle` with properties like make, model, and year, avoid creating another piece of state like `makeWithModel` and `setMakeWithModel`. Instead, derive this information from the existing state.

### Example: Names
If you had first name and last name in state, you wouldn't want another piece of state called full name. Instead, derive full name from first name and last name by concatenating strings.

While these examples sound obvious, in large components, it's an easy mistake to make. People don't realize they could derive the information they need from existing state.

## Forms and Derived State

Forms are a great place to use derived state. **You do not need a form library.** While libraries like Formik, React Hook Form, or TanStack Form are fine and useful, they're not generally necessary if you have good patterns.

### Form State Examples
You can derive:
- Whether the form is valid
- Whether there are errors  
- Whether the form is dirty

Instead of storing errors in state, evaluate the form data on each render:

```javascript
// Line 79 - validate function
const errors = validate(formData);
// Check if name length is zero - that's an error
// Check if price is less than zero - that's an error
```

You look at the data and calculate errors on each render rather than storing them in state. This is derived state.

**Performance Consideration:** While this might seem wasteful, the benefits outweigh the minor performance impact, especially with uncomplicated validation logic. If needed, you can always memoize it to calculate only when necessary.

## Other State Types

### Refs
Refs provide a way to make DOM references and hold state that doesn't trigger renders. Use refs when you don't need to show something on the page but want to hold onto it.

**Example:** For undo/redo functionality, hold previous state in a ref since you don't need it to trigger a render, but you want to keep that information for potential state reversion.

### Context
Context is built into React but has limitations for frequently changing data.

### Third-Party Libraries
For remote state, TanStack Query is preferred, though SWR and Apollo are also great options.

## Why Use Third-Party State?

Built-in state hooks have limitations:

### useState Limitations
- Can't protect state - anyone can set it to anything
- No granular control

### useReducer Limitations  
- Clunky developer experience
- People find it confusing
- Underutilized (very few developers use it)
- While useful for unit testing changes in isolation, it hasn't caught on

### useRef Limitations
- Doesn't trigger renders

### useContext Limitations
- Best for data that changes **infrequently**
- Performance problems with frequently changing data
- Especially problematic when wrapped around entire application
- No granular render optimizations
- Only accessible within React

## Benefits of Third-Party State

Consider third-party options when you need:
- Performance and render optimization
- Access outside React (before React renders)
- State protection
- Better developer experience (less code, dev tools)
- Same data or actions across many components
- Many fetch calls
- Complex forms

## State Management Options Overview

### Built into React
Many developers have only used half of the available React state options. Much of the newer functionality comes with React 19, which is safe to adopt and offers compelling features worth considering.

### Web Platform Options
These apply to React developers and shouldn't be overlooked.

### General State Management
**Recommendation:** While Redux was previously popular (and the subject of a top Pluralsight course), **don't use Redux** anymore. Better alternatives have emerged.

**Preferred Option:** **Zustand** - Does most of what Redux does with a simpler API. It's particularly good as a context replacement for performance and API reasons.

### Remote State Management
**Preferred Option:** **TanStack Query**
**Avoid:** RTK Query - too tied to Redux and more complicated than TanStack Query or SWR

### Router-Based State
Modern routers (newer React Router versions and TanStack Router) have loaders that allow earlier fetch firing and state management.

### Form State Libraries
**Options:** TanStack Form and React Hook Form are both fantastic.
**Reality:** You don't need them - you might do fine without them. Usually prefer going without, but sometimes these libraries are nice to have.

## State Selection Flowchart

Here's a decision tree for choosing state approaches:

1. **Start with URL**: Is it shareable via URL? If yes, use URL-based state.

2. **Check if it's fetched data**: Is this data that's been fetched?
   - For multiple routes: Use remote state manager
   - For single route: Consider router's state manager

3. **Check component scope**: Is this for many components?
   - If yes: Use shared state manager (options include: props passing, useContext, local storage, IndexedDB, Redux, etc.)

**Note:** URL-based state is at the top of this flowchart for a reason - people underutilize URLs and should start there.

# State Management in React: A Comprehensive Guide

## State Management Options Overview

There are numerous state management solutions available for React applications, including:
- Zustand
- MobX
- Balsio
- XState
- And many more...

## The Props Passing Strategy

### When to Pass Props
In most cases, the recommended approach is to pass props down through components. This method works well because passing props is essentially passing arguments to functions - a fundamental programming concept.

### The Breaking Point
Props passing becomes problematic at approximately **five or six levels deep**. While it's acceptable to pass props through several layers, beyond this point it becomes:
- Annoying to maintain
- Difficult to scale
- Cumbersome to work with

### Alternative Solutions
When props passing becomes unwieldy:
- **Zustand** for global state management
- **React Context** as an alternative to Zustand

## Form State Management

You have several options for handling form state:
- Dedicated form libraries
- Plain `useState` hooks
- `useReducer` for more complex scenarios

The choice depends on your specific requirements and complexity needs.

## State Management Decision Flowchart

For non-form state management, the various state management libraries mentioned earlier (Zustand, MobX, etc.) work effectively. The key is choosing the right tool based on your application's specific needs.

## React 19: Paving the Cow Paths

### The Cow Path Metaphor
React 19 represents a significant shift in React's approach. Like pedestrians creating dirt paths across grass corners where official sidewalks don't exist, developers have been using React in ways the team didn't initially anticipate. React 19 "paves these cow paths" by providing official solutions for common patterns.

### What React 19 Addresses
React 19 introduces primitives to make the following easier:
- Server-side components
- Action state handling
- API creation
- Form state management
- Optimistic UI implementation

### React 19 Features Count
Interestingly, React 19 contains exactly **19 changes**, making for a coincidental but memorable version number.

## Best Practices and Recommendations

### Component and State Management
- **Keep components small** - This is the key to performance
- **Start local, then lift state** - Begin with local state and elevate only when necessary
- **Avoid overusing Context** - It's frequently overused in React applications
- **Consider alternatives** like Jotai for state management

### Development Patterns
- **Derive state on render** - Calculate values during rendering when possible
- **Form libraries aren't always necessary** - Simple forms often work fine with basic hooks
- **Explore Stady nums** - An awesome feature worth investigating
- **Use Zod** for validation - It's a powerful and free tool

### Code Reusability Guidelines
- **JSX duplication** → Create a component
- **Logic duplication** → Create a custom hook
- **For reusable components** → Copy and paste initially, then refine the API after 2-3 iterations

This approach mirrors how Stack Overflow was developed - they copied and pasted code to create Server Fault, then abstracted the reusable parts.

## Key Takeaways

### State Management Summary
- **30+ state management options** are available
- **Modern useEffect abstractions** like TanStack Query should be considered
- **Most application state is remote** - plan accordingly
- **Eight categories of state** exist (as previously outlined)
- **URL state is underutilized** - remember to leverage it

### React 19 and Future Technologies
- **React 19 paves cow paths** - making common patterns easier
- **Sync technology** is exciting but not yet ready for production
- **2026 timeline** - likely when Sync becomes more ubiquitous
- **Available today** - you can experiment with it now

### General Principles
- **Keep state local and low** - maintain state at the lowest possible level in your component tree
- This fundamental principle will serve you well across all React applications