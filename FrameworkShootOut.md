# A Comparison of Frameworks (And why you should choose Svelte for your next project)

## What is [Svelte](https://svelte.dev/)?

- Svelte is a frontend framework that compiles your code to a single JavaScript bundle. One might call it a "disappearing framework".
- Instead of using a virtual DOM to do state management, Svelte compiles to JavaScript that directly updates the DOM. This makes it lightweight and fast with bundle sizes that can be approximately 50x smaller in the case of Angular and 2x smaller in the case of React.
- As far as frameworks go, it's the closest to writting vanilla JavaScript/HTML/CSS making onboarding quick for new developers.
- TypeScript is supported.

<!-- TODO: Compare vue, react and svelte -->

## Writing Components, A Comparison

Let's look at writing a simple button component in the four different frameworks. The requirements for the component are as follows:

- Must take a label property that will be displayed as the button's text.
- Must handle a click event.

### React

```javascript
var React = require("react");
var Button = React.createClass({
  render: function () {
    return (
      <button onClick="{this.props.handleClick}">{this.props.label}</button>
    );
  },
});
module.exports = Button;
```

### Vue

```html
<template>
  <button>
    <slot />
  </button>
</template>

<script>
  export default {};
</script>
```

### Svelte

```html
<script lang="ts">
  export let title: string;
</script>

<button on:click>{title}</button>
```
