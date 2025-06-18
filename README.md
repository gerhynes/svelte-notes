# Svelte Notes

### Components
Svelte allows you to build your app declaratively out of components that combine markup, styles and behaviours. These components are compiled into JavaScript modules.

```Svelte
<script>
</script>

markup

<style>
</style>
```

Define variables in your `<script>` tags and reference them in markup using `{}`. 

You can also use curly braces to control element attributes. Where name and value match you can use a shorthand.

```Svelte
<img src={src}/>
or
<img {src}/>
```

Styles are scoped to the component.

You can import components from other files and include them in markup.

```Svelte
<script>
import Nested from "./Nested.svelte"
</script>

<Nested/>
```

### State
Svelte has a powerful system of reactivity for keeping the DOM in sync with your application state. 

State variables are defined using the `$state` rune. State reacts to reassignments and mutations.

``` Svelte
<script>
let count = $state(0)
let numbers = $state([1,2,3,4])

function increment() {
	count += 1
}

function addNumber() {
	numbers.push(numbers.length + 1)
}
</script>
```

For state derived from other state, use the `$derived` rune. The expression inside the `$derived` declaration will be re-evaluated whenever its dependencies are updated. Unlike normal state, derived state is read-only.

```
let total = $derived(numbers.reduce(t,n) => t + n, 0)

<p>{numbers.join(" + ")} = {total}</p>
```

You can use the `$inspect` rune to automatically log a snapshot of state whenever it changes: `$inspect(numbers)`.

You can also use rune outside components, for example to share global state. These need to be `.svelte.js` or `.svelete.ts` files.

```
import { counter } from "./shared.svelte.ts"
```

### Effects
State reacts to effects, some created by Svelte on your behalf to update the DOM in response to state changes, others you can create yourself with the `$effect` rune. But if you can put your effects in an event handler, that's almost always preferable.

If the effect function doesn't read any state when it runs, it will only run once, when the component mounts. Effects do not run during server-side rendering.

### Props
You can use the `$props` rune to declare and destructure a component's props.

```Svelte
<script lang ="ts">
	let { answer } = $props()
</script>
```

Props can have default values.

```Svelte
// Nested.svelte
let { answer = "a mystery" } = $props()

// App.svelte
<Nested answer={42}/>
<Nested/>
```

You can spread props.

```
<PackageInfo
	name={pkg.name}
	version={pkg.version}
	description={pkg.description}
	website={pkg.website}
/>

<PackageInfo {...pkg}
```

If you don't destructure props, you can get all the props in one variable.

```
let stuff = $props()

console.log(stuff.name, stuff.version)
```

### Logic
HTML doesn't have a way of expressing logic such as with conditionals and loops, but Svelte does.

You can conditionally render markup with an `if` block.

```
{#if count > 10}
	<p>{count} is greater than 10</p>
{/if}
```

You can also use `else` and `else if` blocks

```
{#if count > 10}
	<p>{count} is greater than 10</p>
{:else if count < 5}
	<p>{count} is less than 5</p>
{:else}
	<p>{count} is between 5 and 10</p>
{/if}
```

`{# ...}` opens a block. `{/ ...}`, closes a block, `{: ...}` continues a block.

You can use an `each` block to loop through a collection and render each element. You can use the current index as a second argument.

```
{#each colors as color, i}
	<button
		style="background: {color}"
		aria-label={color}
		aria-current={selected === color}
		onclick={() => selected = color}
    >{i + 1}</button>
{/each}
```

By default, updating the value of an `each` block will add or remove DOM nodes at the end of the block if the size changes and update the remaining DOM.

You can instead add/remove a specific element by specifying a unique key for each iteration of an `each` block.

```
{#each things as thing (thing.id)}
	<Thing name={thing.name}/>
{/each}
```

Svelte lets you await the value of a promise directly in your markup using `await` blocks. Only the most recent promise is considered, meaning you don't need to worry about race conditions.

```
{#await promise}
	<p>Rolling...</p>
{:then number}
	<p>you rolled a {number}!</p>
{:catch error}
	<p style="color: red">{error.message}</p>
{/await}
```

You can omit the first block if you don't want to show anything until the promise resolves. Also, if you know that your promise can’t reject, you can omit the `catch` block.

```
{#await promise then number}
	<p>you rolled a {number}!</p>
{/await}
```

### Events
You can listen to any DOM event on an element (such as click or pointermove) with an `on<name>` function. Like with any other property, if the name matches the value, you can use the short form. 

```
<div {onpointermove}>
	The pointer is at {Math.round(m.x)} x {Math.round(m.y)}
</div>
```

You can also declare event handlers inline.

```
<div
	onpointermove={(event) => {
		m.x = event.clientX;
		m.y = event.clientY;
	}}
>
	The pointer is at {m.x} x {m.y}
</div>
```

Normally event handlers run during the bubbling phase. Sometimes you want handlers to run during the capture phase instead. To do this add `capture` to the end of the event name.

```
<div onkeydowncapture={(e) => alert(`<div> ${e.key}`)} role="presentation">
	<input onkeydowncapture={(e) => alert(`<input> ${e.key}`)} />
</div>
```

If both capturing and non-capturing handlers exist for a given event, the capturing handlers will run first.

You can pass event handlers to components like any other prop and then wire them up.

```
// Stepper.svelte
let { increment, decrement } = $props();
...
<button onclick={decrement}>-1</button>
<button onclick={increment}>+1</button>

// App.svelte
<Stepper
	increment={() => value += 1}
	decrement={() => value -= 1}
/>
```

You can also spread event handlers directly onto elements.

```
// BigRedButton.svelte
<button {...props}>Push</button>

// App.svelte
<BigRedButton onclick={honk} />
```

### Bindings
As a general rule, data flow in Svelte is top down. A parent component can set props on a child component and a component can set attributes on an element, but not the other way around.

Sometimes, such as with form inputs, it's useful to break that rule by using the `bind:value` directive.

```
<input bind:value={name}>
```

This means that changes to `name` will update the input value but also changes to the input value will update `name`.

`bind:value` coerces input types, such as converting `type="number"` from a string to a number.

```
	<input type="number" bind:value={a} min="0" max="10" />
	<input type="range" bind:value={a} min="0" max="10" />
```

Checkboxes are bound to `input.checked` rather than `input.value`

```
<input type="checkbox" bind:checked={yes}>
```

You can also use `bind:value` with `<select>` elements.

```
<select
	bind:value={selected}
	onchange={() => answer = ''}
>
```

Note that the `<option>` values are objects rather than strings. Svelte doesn't mind.

If you don't set an initial value, the binding will automatically set it to the default value (the first option in the list). But until the binding is initialised, the bound state variable remains undefined so you can't blindly access for example `selected.id` in the template.

If you have multiple type="radio" or type="checkbox" inputs relating to the same value, you can use `bind:group` along with the `value` attribute. Radio inputs in the same group are mutually exclusive, checkbox inputs in the same group form an array of selected values.

```
<input
	type="radio"
	name="scoops"
	value={number}
	bind:group={scoops}
/>
<input
	type="checkbox"
	name="flavours"
	value={flavour}
	bind:group={flavours}
/>
```

A `<select>` element can have a `multiple` attribute, in which case it will populate an array rather than selecting a single value.

```
<select multiple bind:value={flavours}>
	{#each ['cookies and cream', 'mint choc chip', 'raspberry ripple'] as flavour}
		<option>{flavour}</option>
	{/each}
</select>
```

You can omit the `value` attribute on the `<option>` since the value is identical to the element's contents.

The textarea element behaves similarly to a text input in Svelte; use `bind:value`.

```
<textarea bind:value={value}></textarea>
// or
<textarea bind:value></textarea>
```

### Classes and Styles
Like any other attribute, you can specify classes with a JavaScript attribute.

```
<button
	class="card {flipped ? 'flipped' : ''}"
	onclick={() => flipped = !flipped}
>
```

Since adding or removing a class based on some condition is such a common pattern in UI development, Svelte lets you pass an object or array that is converted to a string by clsx.

```
// Always add the card class
// Only add the flipped class if flipped is truthy
<button
	class={["card", { flipped }]}
	onclick={() => flipped = !flipped}
>
```

You can write inline `style` attributes literally.

```
<button
	class="card"
	style="transform: {flipped ? 'rotateY(0)' : ''}; --bg-1: palegoldenrod; --bg-2: black; --bg-3: goldenrod"
	onclick={() => flipped = !flipped}
>
```

If you have a lot of styles, you can use the `style` directive.

```
<button
	class="card"
	style:transform={flipped ? 'rotateY(0)' : ''}
	style:--bg-1="palegoldenrod"
	style:--bg-2="black"
	style:--bg-3="goldenrod"
	onclick={() => flipped = !flipped}
>
```

The `:global` CSS modifier allows you to indiscriminately target elements inside other components. 

```
<style>
	.boxes :global(.box:nth-child(1)) {
		background-color: red;
	}

	.boxes :global(.box:nth-child(2)) {
		background-color: green;
	}

	.boxes :global(.box:nth-child(3)) {
		background-color: blue;
	}
</style>
```

But this is verbose and brittle, if you change the implementation details of the component it could break the selector. It should be an escape hatch of last resort.

Components should be able to decide which styles can be controlled from the outside, just like props. Use CSS custom properties. 

Any parent element can set the value of `--color`, but you can also set it on individual components. The value can be dynamic, like any other attribute.

```
// Box.svelte
<style>
	.box {
		width: 5em;
		height: 5em;
		border-radius: 0.5em;
		margin: 0 0 1em 0;
		background-color: var(--color, #ddd);
	}
</style>

// App.svelte
<div class="boxes">
	<Box --color="red"/>
	<Box --color="green"/>
	<Box --color="blue"/>
</div>
```

Not: This works by wrapping the component in an element with `display: contents`, which can affect selectors like `.parent > .child`.

### Actions
Actions are essentially element-level lifecycle functions. They are useful for things like:
- interfacing with third-party libraries
- lazy-loading images
- tooltips
- adding custom event handlers



Actions are added to an element using the `use` directive. 

```
<div class="menu" use:trapFocus>
...
</div>
```

An action function is called when a node is mounted to the DOM. Inside the action you have an effect. When the node is unmounted, you need to do some cleanup, such as removing event listeners.

```
export function trapFocus(node) {
	const previous = document.activeElement;

	function focusable() {
		return Array.from(node.querySelectorAll('button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'));
	}

	function handleKeydown(event) {
		if (event.key !== 'Tab') return;

		const current = document.activeElement;

		const elements = focusable();
		const first = elements.at(0);
		const last = elements.at(-1)

		if (event.shiftKey && current === first) {
			last.focus();
			event.preventDefault();
		}

		if (!event.shiftKey && current === last) {
			first.focus();
			event.preventDefault();
		}
	}

	$effect(() => {
		focusable()[0]?.focus();
		node.addEventListener('keydown', handleKeydown);

		return () => {
			node.removeEventListener('keydown', handleKeydown);
			previous?.focus();
		};
	});
}
```

An action can take an argument, which the action function will be called with alongside the element it belongs to.

```
function tooltip(node, fn) {
	$effect(() => {
		const tooltip = tippy(node, fn());

		return tooltip.destroy;
	});
}
...
<button use:tooltip={() => ({ content })}>
	Hover me
</button>
```

### Transitions
You can make more appealing UIs by gracefully transitioning elements into and out of the DOM. Svelte makes this easy with the `transition` directive.

```
<script lang="ts">
	import { fade } from 'svelte/transition';

	let visible = $state(true);
</script>

<p transition:fade>
	Fades in and out
</p>
```

Transition functions can accept parameters.

```
import { fly } from "svelte/transition";
...
<p transition:fly={{ y: 200, duration: 2000 }}>
	Flies in and out
</p>
```

Note that transitions are reversible. For example, if you toggle a checkbox while the transition is ongoing, it transitions from the current point, rather than from the beginning or end.

Instead of a `transition` directive, an element can have an `in` or `out` directive, or both. In this case, the transitions are not reversed.

```
<p in:fly={{ y: 200, duration: 2000 }} out:fade>
	Flies in, fades out
</p>
```

As well as using Svelte's built-in transitions, you can create your own. The code fort the built-in fade transition is:

```
function fade(node, { delay = 0, duration = 400 }) {
	const o = +getComputedStyle(node).opacity;

	return {
		delay,
		duration,
		css: (t) => `opacity: ${t * o}`
	};
}
```

The function takes two arguments, the node to which the transition is applied, and any parameters that were passed in, and returns a transition object which can have the following properties.

- delay - milliseconds before the transition starts
- duration - length of the transition in milliseconds
- easing - a `p => t` easing function
- css - a `(t, u) => css` function where `u === 1 - t`
- tick - a `(t, u) => {...}` function that has some effect on the node

The `t` value is 0 at the beginning of an intro or the end of an outro, and 1 at the end of an intro or the beginning of an outro.

While you should generally use CSS for transitions as much as possible, there are some effects that can't be achieved without JavaScript, such as a typewriter effect.

It can be useful to know when transitions are beginning and ending. Svelte dispatches events that you can listen to like any other DOM event.

```
<p
	transition:fly={{ y: 200, duration: 2000 }}
	onintrostart={() => status = 'intro started'}
	onoutrostart={() => status = 'outro started'}
	onintroend={() => status = 'intro ended'}
	onoutroend={() => status = 'outro ended'}
>
	Flies in and out
</p>
```

Ordinarily, transitions will only play on elements when their direct containing block is added or destroyed. You can use global transitions which play when any block containing the transitions is added or removed.

```
<div transition:slide|global>
	{item}
</div>
```

Key blocks destroy and recreate their contents when the value of an expression changes. This is useful if you want an element to play its transition whenever a value changes instead of only when the element enters or leaves the DOM.

```
// 
{#key i}
	<p in:typewriter={{ speed: 10 }}>
		{messages[i] || ''}
	</p>
{/key}
```

### Snippets and Render Tags
Snippets allow you to reuse content within a component, without extracting it out into a separate file.

```
{#snippet monkey(emoji, description)}
	<tr>
		<td>{emoji}</td>
		<td>{description}</td>
		<td>\u{emoji.charCodeAt(0).toString(16)}\u{emoji.charCodeAt(1).toString(16)}</td>
		<td>&amp#{emoji.codePointAt(0)}</td>
	</tr>
{/snippet}
```

The snippet is only visible once you render it. Snippets can accept zero or more parameters.

```
{@render monkey('🙈', 'see no evil')}
{@render monkey('🙉', 'hear no evil')}
{@render monkey('🙊', 'speak no evil')}
```

Note: Snippets can be declared anywhere in your component, but, like functions, are only visible to render tags in the same scope or in a child component.

Since snippets, like functions, are just values, they can be passed to components as props.

```
// App.svelte
<FilteredList
	data={colors}
	field="name"
	{header}
	{row}
></FilteredList>

// FilteredList.svelte
let { data, field, header, row } = $props();
...
<div class="header">
	{@render header()}
</div>

<div class="content">
	{#each filtered as d}
		{@render row(d)}
	{/each}
</div>
```

As an authoring convenience, snippets declared directly inside components become props on those components.

```
<FilteredList
	data={colors}
	field="name"
	{header}
	{row}
>
	{#snippet header()}...{/snippet}

	{#snippet row(d)}...{/snippet}
</FilteredList>
```

Any content inside a component that is not part of a declared snippet becomes a special `children` snippet. Since `header` has no parameters, you can turn it into `children` by removing the block tags and renaming the `header` prop to `children` on the other side.

```
<div class="header">
	{@render children()}
</div>
```

### Motion
#### Tweened Values
Motion can be a good way to communicate that a value is changing. Svelte includes classes for adding motion to your UIs.

`Tween` lets a value "tween" smoothly to its target value. The `Tween` class has a writable `target`property and a readonly `current` property. Changes to `tween.target` will cause `tween.current` to move towards it over time, taking account of the `delay`, `duration` and `easing` options.

- delay - milliseconds before the tween starts
- duration - either the duration of the tween in milliseconds or a `(from, to) => milliseconds` function allowing you, for example, to specify longer tweens for larger changes in value
- easing - a `p => t` function
- interpolate - a custom `(from, to) => t => value` function for interpolating between arbitrary values.

```
<script>
	import { Tween } from 'svelte/motion';
	import { cubicOut } from 'svelte/easing';

	let progress = new Tween(0, {
		duration: 400,
		easing: cubicOut
	});
</script>

<progress value={progress.current}></progress>

<button onclick={() => (progress.target = 0)}>
	0%
</button>

<button onclick={() => (progress.target = 0.25)}>
	25%
</button>

<button onclick={() => (progress.target = 0.5)}>
	50%
</button>

<button onclick={() => (progress.target = 0.75)}>
	75%
</button>

<button onclick={() => (progress.target = 1)}>
	100%
</button>
```

#### Springs
`Spring` is an alternative to `Tween` that often works better for values that are frequently changing. As with `Tween`, springs have a writable `target` property and a readonly `current` property.

Changes to `spring.target` will cause `spring.current` to move towards it over time, taking account of the `spring.stiffness` and `spring.damping` parameters.

```
<script>
	import { Spring } from 'svelte/motion';

	let coords = new Spring({ x: 50, y: 50 }, {
		stiffness: 0.1,
		damping: 0.25
	});

	let size = new Spring(10);
</script>

<svg
	onmousemove={(e) => {
		coords.target = { x: e.clientX, y: e.clientY };
	}}
	onmousedown={() => (size.target = 30)}
	onmouseup={() => (size.target = 10)}
	role="presentation"
>
	<circle
		cx={coords.current.x}
		cy={coords.current.y}
		r={size.current}
	/>
</svg>
```

### Context API
The context API provides a mechanism for components to "talk" to each other without passing around data and functions as props, or dispatching lots of events.

You create context using `setContext(key, value)` and access it using `getContext(key)`. The key can be an JavaScript value. `setContext` and `getContext` must be called during component initialization, so that the context can be correctly bound.

Your context object can include anything, including reactive state. This allows you to pass values that change over time to child components.

```
// in a parent component
import { setContext } from 'svelte';

let context = $state({...});
setContext('my-context', context);

...

// in a child component
import { getContext } from 'svelte';

const context = getContext('my-context');
```

### Special Elements
#### `<svelte:window>`
Just as you can add event listeners to any DOM element, you can add event listeners to the `window` object with `svelte:window`

```
function onkeydown(event) {
		key = event.key;
		keyCode = event.keyCode;
	}
	
<svelte:window {onkeydown} />
```

#### `<svelte:document>`
The `<svelte:document>` element lets you listen for events that fire on `document` but not on `window`, such as `selectionchange`.

```
<svelte:document {onselectionchange} />
```

Avoid `mouseenter` and `mouseleave` handlers on this element, as these events are not fired on `document` in all browsers. Use `<svelte:body>` instead.

#### `<svelte:body>`
Similar to svelte:window and svelte:document, `<svelte:body>` lets you listen for events that fire on `document.body`. This is useful for events like  `mouseenter` and `mouseleave` that don't fire on `window`.

```
<svelte:body
	onmouseenter={() => hereKitty = true}
	onmouseleave={() => hereKitty = false}
/>
```

#### `<svelte:head>`
 `<svelte:head>` lets you insert elements inside the `<head>` of your document, such as `<title>` and `<meta>` tags, which are essential for good SEO.

```
<svelte:head>
	<link rel="stylesheet" href="/tutorial/stylesheets/{selected}.css" />
</svelte:head>
```

Note: In server-side rendering (SSR) mode, the contents of `<svelte:head>` are returned separately from the rest of the HTML. 

#### `<svelte:element>`
Sometimes you don't know in advance which element needs to be rendered. Rather than having a long list of `if` blocks, you can use `<svelte:element>`.

```
<svelte:element this={selected}>
	I'm a <code>&lt;{selected}&gt;</code> element
</svelte:element>
```

The `this` value can be any string, or a falsy value; if it's falsy, no element will be rendered.

#### `<svelte:boundary>`
To prevent errors from leaving your app in a broken state, you can contain them inside an error boundary using the `<svelte:boundary>` element.

```
<svelte:boundary onerror={(e) => console.error(e)}>
	<FlakyComponent />

	{#snippet failed(error, reset)}
		<p>Oops! {error.message}</p>
		<button onclick={reset}>Reset</button>
	{/snippet}
</svelte:boundary>
```

Use a `failed` snippet to provide some UI to show when the error occurs. You can pass a reset function as the second argument to `failed`. 

You can also specify an `onerror` handler, which is called with the same arguments passed to the `failed` snippet. This is useful for sending information about the error to a reporting service, or adding UI code outside the error boundary itself.

### `<script module>`
Normally the `<script>` block contains code that runs when each component instance is initialized.

On rare occasions, you will need to run some code outside of an individual component instance. For example, an audio player where playing one track stops any others that are playing.

Code contained in a `<script module>` block will run once, when the module first evaluates, rather than when a component is instantiated.

Note that this is a separate script tag from the component's usual script tag.

This lets the components "talk" to each other without any state management.

```
<script module>
	let current;
</script>

...

<audio
	src={src}
	bind:currentTime={time}
	bind:duration
	bind:paused
	onplay={(e) => {
		const audio = e.currentTarget;

		if (audio !== current) {
			current?.pause();
			current = audio;
		}
	}}
	onended={() => {
		time = 0;
	}}
/>
```

Anything exported from a `module` script block becomes an export from the module itself. You can't have a default export, because the component is the default export.

```
// AudioPlayer.svelte
<script module>
	let current;

	export function stopAll() {
		current?.pause();
	}
</script>

// App.svelte
<script lang="ts">
	import AudioPlayer, { stopAll } from './AudioPlayer.svelte';
	import { tracks } from './tracks.js';
</script>

...

<button onclick={stopAll}>
	stop all
</button>
```

## SvelteKit
Where Svelte is a component framework, SvelteKit is an app framework (or metaframework) that assists in building a production-ready application by providing routing, server-side rendering, data fetching, TypeScript integration, and more.

SvelteKit apps are server-rendered by default for better first load performance and SEO but can transition to client-side navigation to avoid reloading everything when the user navigates.

They can run anywhere JavaScript runs, though your users may not need to run any JavaScript at all.

### Project Structure
- `package.json` lists the project's dependencies and scripts for interfacing with the SvelteKit CLI
- `svelte.config.js` contains your project's configuration
- `vite.config.js` contains the Vite configuration
- `src/` is where your app's code goes. `src/app.html` is your page template and `src/routes` defines the routes of your app
- `static` contains any static assets, such as favicon or `robots.txt`

### Pages
SvelteKit uses filesystem-based routing so the routes of your app are defined by the directories of your codebase.

Every `+page.svelte` files inside `src/routes` creates a page in your app, such as `src/routes/about/+page.svelte` which maps to `/about`.

Note: unlike traditional multi-page apps, navigating to a page and then back updates the contents of the current page, like with a single-page app.

### Layouts
You can use a `+layout.svelte` file to share common UI between different routes. 

A layout file applies to every child route, including the sibling `+page.svelte`, if it exists. So`src/routes/+layout.svelte` will apply to both `src/routes/+page.svelte` and `src/routes/about/+page.svelte`. 

You can nest layouts to arbitrary depth.

```
// +layout.svelte
<script lang="ts">
	let { children } = $props();
</script>

<nav>
	<a href="/">home</a>
	<a href="/about">about</a>
</nav>

{@render children()}
```

### Route Parameters
To create routes with dynamic parameters, use square brackets around a valid variable name.

For example `src/routes/blog/[slug]/+page.svelte` will create a route that matches `/blog/one`, `/blog/two`, `/blog/three` and so on.

Multiple route parameters can appear within one URL segment as long as they are separated by at least one static character. `foo/[bar]x[bat]` is a valid route where `[bar]` and `[baz]` are dynamic parameters.

### Loading Data
#### Page Data
SvelteKit does three main things:
1. Routing - figuring out which route matches an incoming request
2. Loading - getting the data needed by that route
3. Rendering - generating some HTML (on the server) of updating the DOM (on the client)

Every page of your app can declare a `load` function in a `+page.server.js` file alongside the `+page.svelte` file. As the filename suggests, this module only ever runs on the server, including for client-side navigations.

```
// src/routes/blog/+page.server.js
import { posts } from './data.js';

export function load() {
	return {
		summaries: posts.map((post) => ({
			slug: post.slug,
			title: post.title
		}))
	};
}
```

If a user tries to visit an invalid pathname you can respond with a 404 page. 

```
// src/routes/blog/[slug]/+page.server.js
import { error } from '@sveltejs/kit';
import { posts } from '../data.js';

export function load({ params }) {
	const post = posts.find((post) => post.slug === params.slug);

	if(!post) error(404);

	return {
		post
	};
}
```

You can then access the loaded data in the corresponding `+page.svelte` via the `data` prop.

```
// src/routes/blog/+page.svelte
<script lang="ts">
	let { data } = $props();
</script>

<h1>blog</h1>

<ul>
	{#each data.summaries as { slug, title }}
		<li><a href="/blog/{slug}">{title}</a></li>
	{/each}
</ul>
```

```
// src/routes/blog/[slug]/+page.svelte
<script lang="ts">
	let { data } = $props();
</script>

<h1>blog post</h1>
<h1>{data.post.title}</h1>
<div>{@html data.post.content}</div>
```

#### Layout Data
Just as `+layout.svelte` files create UI for every child route, `+layout.server.js` files load data for every child route.

Say you a list of "more posts" in a sidebar on your blog. Instead of loading this for every page, you can load it in the layout. 

When you navigate from one post to another, you only need to load the data for the post itself, the layout data is still valid.

```
// src/routes/blog/+layout.server.js
import { posts } from './data.js';

export function load() {
	return {
		summaries: posts.map((post) => ({
			slug: post.slug,
			title: post.title
		}))
	};
}
```

```
// src/routes/blog/[slug]/+layout.svelte
<script lang="ts">
	let { data, children } = $props();
</script>

<div class="layout">
	<main>
		{@render children()}
	</main>

	<aside>
		<h2>More posts</h2>
		<ul>
			{#each data.summaries as { slug, title }}
				<li>
					<a href="/blog/{slug}">{title}</a>
				</li>
			{/each}
		</ul>
	</aside>
</div>
```

### Headers and Cookies
Inside a load function (as well as in form actions, hooks, and API routes) you have access to a `setHeaders` function, which can set headers on the response. 

Most commonly, you'll use it to customize caching behaviour with the `Cache-Control` response header.

```
export function load({ setHeaders }) {
	setHeaders({
		'Cache-Control': 'max-age=604800'
	});
}
```

The `setHeaders` function can't be used with the `Set-Cookie` header. Instead you should use the `cookies` API.

In your `load` functions, you can read a cookie with `cookies.get(name, options)`:

```
export function load({ cookies }) {
	const visited = cookies.get('visited');

	return {
		visited: visited === 'true'
	};
}
```

To set a cookie, use `cookies.set(name, value, options)`. It's strongly recommended that you explicitly configure the `path` when setting a cookie, since browsers' default behaviour (somewhat uselessly) is to set the cookie on the parent of the current path.

```
export function load({ cookies }) {
	const visited = cookies.get('visited');

	cookies.set('visited', 'true', { path: '/' });

	return {
		visited: visited === 'true'
	};
}
```

Calling `cookies.set(name, ...)` causes a `Set-Cookie` header to be written, but it also updates the internal map of cookies, meaning any subsequent calls to `cookies.get(name)` during the same request will return the updated value.

Under the hood, SvelteKit uses the popular `cookie` package. SvelteKit sets the following defaults to make your cookies more secure:

```
{
	httpOnly: true,
	secure: true,
	sameSite: 'lax'
}
```

### Shared Modules
Because SvelteKit uses directory-based routing, it's easy to place modules and components alongside the routes that use them. A good rule of thumb is to put code close to where it's used.

But sometimes code is used in multiple places. When this happens, `src/lib` serves as a useful place to put code that can be accessed by all routes without needing to prefix imports with `../../../`. Anything in the `src/lib` directory can be accessed by any module in `src` via the `$lib` alias.

```
<script lang="ts">
	import { message } from '$lib/message.js';
</script>

<h1>a deeply nested route</h1>
<p>{message}</p>
```

### Forms
#### The `<form>` element
When you need to send data to the server, you'll use the `<form>` element.

```
<form method="POST">
	<label>
		Add a todo:
		<input
			name="description"
			autocomplete="off"
		/>
	</label>
</form>
```

The browser will make a POST request (because of the `method="POST"` attribute) to the current page. 

You will need to create a server-side **action** to handle the post request.

```
import * as db from '$lib/server/database.js';

export function load({ cookies }) {
	// ...
}

export const actions = {
	default: async ({ cookies, request }) => {
		const data = await request.formData();
		db.createTodo(cookies.get('userid'), data.get('description'));
	}
};
```

The request is a standard `Request` object and so `await request.formData()` returns a `FormData` instance.

The page will reload with the new data automatically, you don't need to fetch it. And because you're using a `<form>` element, the app would work even in the user disabled client-side JavaScript.

#### Named Form Actions
You will usually have multiple form actions on a page, for example for CRUD functionality, rather than just the `default` action. 

```
export const actions = {
	create: async ({ cookies, request }) => {
		const data = await request.formData();
		db.createTodo(cookies.get('userid'), data.get('description'));
	},

	delete: async ({ cookies, request }) => {
		const data = await request.formData();
		db.deleteTodo(cookies.get('userid'), data.get('id'));
	}
};
```

Default actions cannot coexist with named actions.

The `<form>` element has an optional `action` attribute, which is similar to an `<a>` element's `href` attribute.

```
<form method="POST" action="?/create">
	<label>
		Add a todo:
		<input
			name="description"
			autocomplete="off"
		/>
	</label>
</form>
```

The `action` attribute can be any URL. If the action was defined on another page, you might have something like `/todos?/create`. If the action is on the same page, you can omit the pathname altogether, hence the leading `?` character.

```
<form method="POST" action="?/delete">
	<input type="hidden" name="id" value={todo.id} />
	<span>{todo.description}</span>
	<button aria-label="Mark as complete"></button>
</form>
```
#### Validation
Users will submit all kinds of nonsense data if given the chance. To prevent them from causing chaos, it's important to validate form data.

The first line of defence is the browser's built-in form validation, such as marking an `<input>` as `required`.

You also need server-side validation.

```
export function createTodo(userid, description) {
	if (description === '') {
		throw new Error('todo must have a description');
	}

	const todos = db.get(userid);

	if (todos.find((todo) => todo.description === description)) {
		throw new Error('todos must be unique');
	}

	todos.push({
		id: crypto.randomUUID(),
		description,
		done: false
	});
}
```

SvelteKit will take users to a generic error page. On the server, you will see the specific error but SvelteKit hides unexpected error messages from users because they often contain sensitive data.

It's better UX to stay on the same page and provide an indication of what went wrong so that the user can fix it. To do this, use the `fail` function to return data from the action along with an appropriate HTTP status code.

```
import { fail } from '@sveltejs/kit';
import * as db from '$lib/server/database.js';

export function load({ cookies }) {...}

export const actions = {
	create: async ({ cookies, request }) => {
		const data = await request.formData();

		try {
			db.createTodo(cookies.get('userid'), data.get('description'));
		} catch (error) {
			return fail(422, {
				description: data.get('description'),
				error: error.message
			});
		}
	}
```

In the `+page.svelte` file, you can access the returned value via the `form` prop, which is only ever populated after a form submission.

```
<script lang="ts">
	let { data, form } = $props();
</script>

<div class="centered">
	<h1>todos</h1>

	{#if form?.error}
		<p class="error">{form.error}</p>
	{/if}

	<form method="POST" action="?/create">
		<label>
			add a todo:
			<input
				name="description"
				value={form?.description ?? ''}
				autocomplete="off"
				required
			/>
		</label>
	</form>
```

Note: You can also return data from an action without wrapping it in `fail`, for example to show a "success" message when data was saved, and it will also be available via the `form` prop.

#### Progressive Enhancement
Since you are using `<form>`, your app works even if the user doesn't have JavaScript. Since most users do have JavaScript, you can progressively enhance the experience.

Import the `enhance` prop and add the `use:enhance` directive to the `<form>` elements:

```
<script>
	import { fly, slide } from 'svelte/transition';
	import { enhance } from '$app/forms';
	
	let { data, form } = $props();
</script>

<div class="centered">
	<h1>todos</h1>

	{#if form?.error}
		<p class="error">{form.error}</p>
	{/if}

	<form method="POST" action="?/create" use:enhance>
		<label>
			add a todo:
			<input
				name="description"
				value={form?.description ?? ''}
				autocomplete="off"
				required
			/>
		</label>
	</form>

	<ul class="todos">
		{#each data.todos as todo (todo.id)}
			<li in:fly={{ y: 20 }} out:slide>
				<form method="POST" action="?/delete" use:enhance>
					<input type="hidden" name="id" value={todo.id} />
					<span>{todo.description}</span>
					<button aria-label="Mark as complete"></button>
				</form>
			</li>
		{/each}
	</ul>
</div>
```

Now, when JavaScript is enabled, `use:enhance` will emulate the browser-native behaviour expect for full-page reloads. It will:
- update the `form` prop
- invalidate all data on a successful response, causing `load` functions to re-run
- navigate to the new page on a redirect response
- render the nearest error page if an error occurs

Since you are updating the page rather than reloading it, you can include transitions.

#### Customizing `use:enhance`
With `use:enhance`, by providing a callback, you can add things like pending states and optimistic UI. 

If creating or deleting a resource will take some time, you can use local state to disable the input during creation or update the UI immediately on deletion.

```
<script>
	import { fly, slide } from 'svelte/transition';
	import { enhance } from '$app/forms';

	let { data, form } = $props();

	let creating = $state(false);
	let deleting = $state([]);
</script>

<div class="centered">
	<h1>todos</h1>

	{#if form?.error}
		<p class="error">{form.error}</p>
	{/if}

	<form
		method="POST"
		action="?/create"
		use:enhance={() => {
			creating = true;

			return async ({ update }) => {
				await update();
				creating = false;
			};
		}}
	>
		<label>
			add a todo:
			<input
				disabled={creating}
				name="description"
				value={form?.description ?? ''}
				autocomplete="off"
				required
			/>
		</label>
	</form>

	<ul class="todos">
		{#each data.todos.filter((todo) => !deleting.includes(todo.id)) as todo (todo.id)}
			<li in:fly={{ y: 20 }} out:slide>
				<form
					method="POST"
					action="?/delete"
					use:enhance={() => {
						deleting = [...deleting, todo.id];
						return async ({ update }) => {
							await update();
							deleting = deleting.filter((id) => id !== todo.id);
						};
					}}
				>
					<input type="hidden" name="id" value={todo.id} />
					<span>{todo.description}</span>
					<button aria-label="Mark as complete"></button>
				</form>
			</li>
		{/each}
	</ul>

	{#if creating}
		<span class="saving">saving...</span>
	{/if}
</div>
```

`use:enhance` is very customizable. You can `cancel()` submissions, handle redirects, control whether the form is reset, and so on.

### API Routes
SvelteKit allows you to create more than just pages. You can create API routes by adding a `+server.js` file that exports functions corresponding to HTTP methods: `GET`, `PUT`, `POST`, `PATCH`, and `DELETE`.
#### GET Handlers
For example, to return data from a `/roll` API route, create a route by adding a `src/routes/roll/+server.js` file.

```
import { json } from '@sveltejs/kit';

export function GET() {
	const number = Math.floor(Math.random() * 6) + 1;

	return json(number);
}
```

Request handlers must return a `Response` object. Since it's common to return JSON from an API route, SvelteKit provides a convenience `json()` function for generating these responses.

#### POST Handlers
You can use POST handlers to mutate data. In most cases, you should use form actions instead. You'll end up writing less code and it'll work without JavaScript, making it more resilient.

```
// src/routes/+page.svelte
<label>
	Add a todo:
	<input
		type="text"
		autocomplete="off"
		onkeydown={async (e) => {
			if (e.key !== 'Enter') return;

			const input = e.currentTarget;
			const description = input.value;
			
			const response = await fetch('/todo', {
				method: 'POST',
				body: JSON.stringify({ description }),
				headers: {
					'Content-Type': 'application/json'
				}
			});

			const { id } = await response.json();

			const todos = [...data.todos, {
				id,
				description
			}];

			data = { ...data, todos };

			input.value = '';
		}}
	/>
</label>

// src/routes/todo/+server.js
import { json } from '@sveltejs/kit';
import * as database from '$lib/server/database.js';

export async function POST({ request, cookies }) {
	const { description } = await request.json();

	const userid = cookies.get('userid');
	const { id } = await database.createTodo({ userid, description });

	return json({ id }, { status: 201 });
}

```

As with `load` functions and form actions, the `request` is a standard `Request` object. `await request.json()` returns the data that you posted from the event handler.

Here you're retuning a response with a 201 Created stats and the id of the newly created todo in your database.

Note: You should only update `data` in such a way that you'd get the same result by loading the page. The `data` prop is not deeply reactive, so you need to replace it. Mutations like `data.todos = todos` will not cause a re-render.

#### Other Handlers
Similarly, you can add handlers for other HTTP verbs, such as `PUT` and `DELETE`.

```
// src/routes/todo/[id]/+server.js
import * as database from '$lib/server/database.js';

export async function PUT({ params, request, cookies }) {
	const { done } = await request.json();
	const userid = cookies.get('userid');

	await database.toggleTodo({ userid, id: params.id, done });
	return new Response(null, { status: 204 });
}

export async function DELETE({ params, cookies }) {
	const userid = cookies.get('userid');

	await database.deleteTodo({ userid, id: params.id });
	return new Response(null, { status: 204 });
}
```

Since you don't need to return any actual data to the browser, you're retuning an empty `Response` with a 204 No Content status.

You can then interact with these endpoints inside your event handlers.

```
<label>
	<input
		type="checkbox"
		checked={todo.done}
		onchange={async (e) => {
			const done = e.currentTarget.checked;

			await fetch(`/todo/${todo.id}`, {
				method: 'PUT',
				body: JSON.stringify({ done }),
				headers: {
					'Content-Type': 'application/json'
				}
			});
		}}
	/>
	<span>{todo.description}</span>
	<button
		aria-label="Mark as complete"
		onclick={async (e) => {
			await fetch(`/todo/${todo.id}`, {
				method: 'DELETE'
			});

			const todos = data.todos.filter((t) => t !== todo);

			data = { ...data, todos };
		}}
></button>
</label>
```

### `$app/state`
