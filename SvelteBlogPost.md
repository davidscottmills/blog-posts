# Why I'm Excited About Svelte

In this blog post, I'll share with you why I'm excited about Svelte and give you a glimpse into what svelte is an how it works. My hope is to convince you to give Svelte a chance for your next project.

For an in depth tutorial, visit [Svelte's Tutorial Site](https://svelte.dev/tutorial/basics) where you can learn Svelte right in your browser.

If you just want to read the API specification, you can do that [here](https://svelte.dev/docs).

## What is [Svelte](https://svelte.dev/)?

- Svelte is a frontend framework that compiles your code to a single JavaScript bundle. One might call it a "disappearing framework".
- Instead of using a virtual DOM to do state management, Svelte compiles to JavaScript that directly updates the DOM. This makes it lightweight and fast with bundle sizes that can be approximately 50x smaller in the case of Angular and 2x smaller in the case of React.
- As far as frameworks go, it's the closest to writting vanilla JavaScript/HTML/CSS making onboarding quick for new developers.
- TypeScript is supported.

## What does a Svelte application look like?

- A Svelte app has a single index.html that is served up by your server. That index.html file will then load your javascript bundle and any css files you might need. Your bundler will take care of generating the javascript and css bundles that your index.html references.
- During development, a Svelte app might look more like your traditional SPA, being made up of routes (if you're using an internal router) and components.
- Looking to get started quickly in your IDE? Get the starter template [here](https://github.com/sveltejs/template).
- Note: If you're a fan of templates, the Svelte community has a lot of different template available (think Svelte with Apollo GraphQL, Tailwind, etc.). A quick Google search will lead you to a multitude of templates.

Here's an example of a simple Svelte application.

```javascript
// main.ts

import App from "./App.svelte";

const app = new App({
  target: document.body,
});

export default app;
```

```html
<!-- App.svelte -->
<script lang="ts">
  let name = "World";
</script>

<main>
  <h1>Hello {name}!</h1>
</main>

<style></style>
```

## Reactivity

Svelte's reactivity is dead simple. To demonstrate this, we'll make a simple counter component. When the count variable is changed, the DOM updates.

```html
<script lang="ts">
  let count = 0;

  function subtract() {
    count--;
  }

  function add() {
    count++;
  }
</script>

<h1>The count is: {count}</h1>
<div>
  <button on:click={subtract}>-</button>
  <button on:click={add}>+</button>
</div>
<!-- Even simpler, use arrow functions -->
<div>
  <button on:click={() => count--}>-</button>
  <button on:click={() => count++}>+</button>
</div>
```

## Displaying Data

Svelte has many built in "blocks" that help with rendering HTML. Read about them here:

- [if](https://svelte.dev/docs#if)
- [each](https://svelte.dev/docs#each)
- [await](https://svelte.dev/docs#await)
- [key](https://svelte.dev/docs#key)
- [html](https://svelte.dev/docs#html)
- [debug](https://svelte.dev/docs#debug)

We'll look at using the "each" block to display data from an array.

```html
<script lang="ts">
  interface link {
    name: string;
    href: string;
  }

  let links: link[] = [
    { name: "Svelte", href: "https://svelte.dev/" },
    { name: "Made with Svelte", href: "https://madewithsvelte.com/" },
    { name: "Svelte Native", href: "https://svelte-native.technology/" },
  ];
</script>

{#each links as link}
<ul>
  <li>
    <a target="_blank" href="{link.href}">{link.name}</a>
  </li>
</ul>
{/each}
```

## Sharing Data between Components

There are a number of ways to share data between Svelte components. I'll cover two: Props and Stores.

### Props

Sharing data is easy using props. Simply export the variable that you want another component to be able to control using the "export" key word. In my opinion these are best used for parent-child components or component libraries.

```html
<!-- Button.Svelte -->
<script lang="ts">
  export let title: string;
</script>

<!-- Note: on:click here allows for event forwarding -->
<button on:click>{title}</button>
```

Using this component is easy.

```html
<!-- SomeOtherComponent.svelte -->
<script lang="ts">
  import Button from "./components/Button.svelte";

  function handleClick() {
    alert("hello");
  }
</script>

<button title="Say Hello" on:click="{handleClick}" />
```

### Stores

Stores are useful when you have state that needs to be accessed by multiple, unrelated components. In this example, we'll create a use a user store to share the state of the user.

```javascript
// user.store.ts
import { writable } from "svelte/store";

export interface User {
  Name: string;
  Email: string;
  LoggedIn: boolean;
}

function createUserStore() {
  // A writable store has three methods Subscribe, Set, Update
  return writable < User > { Name: "", Email: "", LoggedIn: false };
}

export const userStore = createUserStore();
```

```html
<script lang="ts">
  import { onDestroy } from "svelte";
  import { userStore } from "../stores/user.store";
  import type { User } from "../stores/user.store";
  let user: User;

  const unsubscribe = userStore.subscribe((x) => {
    user = x;
  });

  function login() {
    userStore.set({
      Name: "John Smith",
      Email: "john.smith@test.com",
      LoggedIn: true,
    });
  }

  onDestroy(() => {
    unsubscribe();
  });
</script>

<!-- Svelte if blocks are useful for conditionally rendering markup. -->
{#if user && user.LoggedIn}
<div>Name: {user.Name}</div>
<div>Email: {user.Email}</div>
{:else}
<button on:click="{login}">Login</button>
{/if}
```

If you're familiar with Angular, you may have written something like this using RxJS observables. In my opinion, doing this with Svelte stores is much easier.

Let's refactor the store to only export the subscribe method, along with a login method that a component will call to log the user in. We'll create a separate login form component to display a login form and call the store's login method on submission.

```javascript
// user.store.ts
import { writable } from "svelte/store";

export interface User {
  Name: string;
  Email: string;
  LoggedIn: boolean;
}

function createUserStore() {
  // A writable store has three methods Subscribe, Set, Update
  const { subscribe, set, update } =
    writable < User > { Name: "", Email: "", LoggedIn: false };
  return {
    subscribe,
    login: async (email: string, password: string) => {
      // await some backend call here
      if (email === "john.smith@test.com" && password === "password") {
        set({
          Name: "John Smith",
          Email: "john.smith@test.com",
          LoggedIn: true,
        });
      }
    },
  };
}

export const userStore = createUserStore();
```

```html
<!-- LoginForm.svelte -->
<script lang="ts">
  import { userStore } from "../stores/user.store";

  let email: string;
  let password: string;
</script>

<form on:submit|preventDefault={() => userStore.login(email, password)}>
  <input type="text" placeholder="email" bind:value={email} />
  <input type="password" placeholder="password" bind:value={password} />
  <button type="submit"> Login </button>
</form>

```

```html
<!-- UserProfile.svelte -->
<script lang="ts">
  import { onDestroy } from "svelte";
  import { userStore } from "../stores/user.store";
  import type { User } from "../stores/user.store";
  let user: User;
  let email: string;
  let password: string;

  const unsubscribe = userStore.subscribe((x) => {
    user = x;
  });

  onDestroy(() => {
    unsubscribe();
  });
</script>

{#if user && user.LoggedIn}
<div>Name: {user.Name}</div>
<div>Email: {user.Email}</div>
{:else}
<LoginForm />
{/if}
```

## Conclusion

I hope I've been able to convince you to give Svelte a try. I've only just scratched the surface and would encourage you to go through the Svelte tutorials.
