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
SvelteKit has three readonly state objects available via the `$app/state` module: `page`, `navigation` and `updated`. The one you'll use most often is page, which provides information about the current page.
- `url` - the URL of the current page
- `params` - the current page's parameters
- `route` - an object with an `id` property representing the current route
- `status` - the HTTP status code of the current page
- `error` - the error object of the current page, if any
- `data` - the data for the current page, combining the return values for all the `load` functions
- `form` - the data returned from a form action

Each of these properties is reactive, using `$state.raw` under the hood.

```
// src/routes/+layout.svelte
<script lang="ts">
	import { page } from '$app/state';

	let { children } = $props();
</script>

<nav>
	<a href="/" aria-current={page.url.pathname === '/'}>
		home
	</a>

	<a href="/about" aria-current={page.url.pathname === '/about'}>
		about
	</a>
</nav>

{@render children()}
```

Note: Prior to SvelteKit 2.12, you had to use `$app/stores` for this, which provides a `$page` store with the same information

#### `navigating`
The `navigating` object represents the current navigation. When a navigation starts, because of a link click or a back/forward navigation, or a programmatic `goto`, the value of `navigating` will become an object with the following properties:

- `from` and `to` - objects with `params`, `route` and `url` properties
- `type` - the type of navigation, such as `link`, `popstate`, or `goto`

It can be used to show loading indicators for long-running navigations.

```
<script lang="ts">
	import { page, navigating } from '$app/state';

	let { children } = $props();
</script>

<nav>
	<a href="/" aria-current={page.url.pathname === '/'}>
		home
	</a>

	<a href="/about" aria-current={page.url.pathname === '/about'}>
		about
	</a>

	{#if navigating.to}
		navigating to {navigating.to.url.pathname}
	{/if}
</nav>

{@render children()}
```

#### `updated`
The `updated` state contains `true` or `false` depending on whether a new version of the app has been deployed since the page was first opened. For this to work, your `svelte.config.js` must specify `kit.version.pollInterval`.

Version changes only happen in production, not during development. So `updated.current` will always be false in dev mode.

You can manually check for new versions, regardless of `pollInterval`, by calling `updated.check()`.

```
{#if updated.current}
	<div class="toast">
		<p>
			A new version of the app is available

			<button onclick={() => location.reload()}>
				reload the page
			</button>
		</p>
	</div>
{/if}
```

### Errors and Redirects
There are two types of errors in SvelteKit: expected errors and unexpected errors.

An expected error is one that was thrown via the `error` helper from `@sveltejs/kit`.

```
// src/routes/expected/+page.server.js
import { error } from '@sveltejs/kit';

export function load() {
	error(420, 'Enhance your calm');
}
```

Any other error is treated as unexpected.

When you throw an expected error, you're telling SvelteKit "don't worry, I know what I'm doing here". An unexpected error, by contrast, is assumed to be a bug in your app. When an unexpected error is throw, its message and stack trace will be logged to the console.

Expected error messages are shown to the user, whereas unexpected error messages are redacted and replaced with a generic "Internal Error" and a 500 status code. This is because error messages can contain sensitive data.

#### Error Pages
When something goes wrong inside a `load` function SvelteKit renders an error page.

You can customize the default error page by create a `src/routes/+error.svelte` component.

```
<script lang="ts">
	import { page } from '$app/state';
	import { emojis } from './emojis.js';
</script>

<h1>{page.status} {page.error.message}</h1>
<span style="font-size: 10em">
	{emojis[page.status] ?? emojis[500]}
</span>
```

Notice that the `+error.svelte` component is rendered inside the root `+layout.svelte`. But you can create more granular `+error.svelte` boundaries.

```
// src/routes/expected/+error.svelte
<h1>this error was expected</h1>
```

This component will be rendered for `/expected`, while the root `src/routes/+error/svelte` page will be rendered for any other errors that occur.

#### Fallback Errors
If things go really wrong, such as an error while loading the root layout data, or while rendering the error page, SvelteKit will fall back to a static error page.

You can customize the fallback error page by creating a `src/error.html` file.

```
<h1>Game over</h1>
<p>Code %sveltekit.status%</p>
<p>%sveltekit.error.message%</p>
```

This file can include the following:
- `%sveltekit.status%` - the HTTP status code
- `%sveltekit.error.message%` - the error message

#### Redirects
You can use the `redirect` mechanism to redirect from one page to another.

```
// src/routes/a/+page.server.js
import { redirect } from '@sveltejs/kit';

export function load() {
	redirect(307, '/b');
}
```

You can `redirect` inside `load` functions, form actions, API routes and the `handle` hook.

The most common status codes you'll use are:
- `303` - for form actions, following a successful submission
- `307` - for temporary redirects
- `308` - for permanent redirects

Note: `redirect()` throws, like `error()`, meaning no code *after* the redirect will run.

### Hooks
SvelteKit provides several hooks, ways to intercept and override the framework's default behaviour.
#### `handle`
`handle` is the most elementary hook. It lives in `src/hooks.server.js` and receives an `event` object along with a `resolve` function, and returns a `Response`.

Within `resolve` SvelteKit matches the incoming request URL to a route of your app, imports the relevant code (such as from `+page.server.js` and `+page.svelte`), loads the data needed by the route, and generates the response.

```
export async function handle({ event, resolve }) {
	return await resolve(event);
}
```

For pages, as opposed to API routes, you can modify the generated HTML with `transformPageChunk`.

You can also create entirely new routes

```
// src/hooks.server.js
export async function handle({ event, resolve }) {
	if (event.url.pathname === '/ping') {
		return new Response('pong');
	}

	return await resolve(event, {
		transformPageChunk: ({ html }) => html.replace(
			'<body',
			'<body style="color: hotpink"'
		)
	});
}
```

#### The RequestEvent Object
The `event` object passed into `handle` is the same object (an instance of `RequestEvent`) that is passed into API routes in `+server.js` files, form actions in `+page.server.js` files, and `load` functions in `+page.server.js` and `+layout.server.js`.

It contains a number of useful properties and methods:
- `cookies` - the cookies API
- `fetch` - the standard Fetch API with additional powers
- `getClientAddress()` - a function to get the client's IP address
- `isDataRequest` - `true` if the browser is requesting data for a page during client-side navigation, `false` if a page/route is being requested directly
- `locals` - a place to put arbitrary data
- `params` - the route parameters
- `route` - an object with an `id` property representing the route that was matched
- `setHeaders()` - a function for setting HTTP headers on the response
- `url` - a URL object representing the current request

A useful pattern is to add some data to `event.locals` in handle so that it can be read in subsequent `load` functions

```
// src/hooks.server.js
export async function handle({ event, resolve }) {
	event.locals.answer = 42;
	return await resolve(event);
}

// src/routes/+page.server.js
export function load(event) {
	return {
		message: `the answer is ${event.locals.answer}`
	};
}
```

#### `handleFetch`
The `event` object has a `fetch` method that behaves like the standard Fetch API but with superpowers;
- it can be used to make credentialed requests on the server, as it inherits the `cookie` and `authorization` headers from the incoming request
- it can make relative requests on the server (ordinarily, `fetch` requires a URL with an origin when used in a server context)
- internal requests (such as for `+server.js` routes) go directly to the handler function when running on the server, without the overhead of a HTTP call

Its behaviour can be modified with the `handleFetch` hook.

```
// src/hooks.server.js
export async function handleFetch({ event, request, fetch }) {
	return await fetch(request);
}
```

`event.fetch` can also be called from the browser. In this scenario, `handleFetch` is useful if you have requests to a public URL like `https://api.yourapp.com` from the browser, that should be redirected to an internal URL (bypassing whatever proxies and load balancers sit between the API server and the public internet) when running on the server.

#### `handleError`
The `handleError` hook lets you intercept unexpected errors and trigger some behaviour, like pinging a Slack channel or sending data to an error logging service.

The default behaviour is to log the error.

```
export function handleError({ event, error }) {
	console.error(error.stack);
}
```

SvelteKit does not show the error message to the user to avoid exposing sensitive information. The error object available to your application as `page.error` in your `+error.svelte` pages or `%sveltekit.error%` in your `src/error.html` fallback is just:

```
{
	message: 'Internal Error' // or 'Not Found' for a 404
}
```

In some situations, you might want to customize this. To do so, you can return an object from `handlError`

```
// src/hooks.server.js
export function handleError({ event, error }) {
	console.error(error.stack);

	return {
		message: 'everything is fine',
		code: 'JEREMYBEARIMY'
	};
}
```

You can now reference properties other than `message` in a custom error page. 

```
// src/routes/+error.svelte
<script lang="ts">
	import { page } from '$app/state';
</script>

<h1>{page.status}</h1>
<p>{page.error.message}</p>
<p>error code: {page.error.code}</p>
```

### Page Options
As well as exporting `load` functions, you can export various page options from `+page.js`, `+page.server.js`, `+layout.js`, `+layout.server.js`:
- `ssr` - whether or not pages should be server-rendered
- `csr` - whether to load the SvelteKit client
- `prerender` - whether to prerender pages at build time, instead of per-request
- `trailingSlash` - whether to strip, add, or ignore trailing slashes in URLS

Page options can be applied to individual pages (if exported from `+page.js` or `+page.server.js`) or groups of pages (if exported from `+layout.js` or `+layout.server.js`). To define an option for the whole app, export it from the root layout.

Child layouts and pages override values set in parent layout, so, for example, you can enable prerendering for your entire app and then disable it for pages that need to be dynamically rendered.

You can mix and match these options in different areas of your app, for example: prerendering marketing pages, dynamically server-rendering data-driven pages, and treating your admin pages as a client-rendered SPA.

#### Server-side Rendering (SSR)
Server-side rendering is the process of generating HTML on the server. This is what SvelteKit does by default. It's important for performance and resilience, and is very beneficial for SEO.

That said, some components cannot be rendered on the server, perhaps because they expect to be able to access browser globals like `window` immediately. If you can, you should change those components so that they can render on the server, but if you can't then you can disable SSR.

```
// src/routes/+page.server.js
export const ssr = false;
```

Note: Setting `ssr` to `false` inside your root `+layout.server.js` effectively turns your entire app into a single-page app (SPA).

#### Client-side Rendering (CSR)
Client-side rendering is what makes the page interactive, such as incrementing a counter when you click a button, and enables SvelteKit to update the page upon navigation without a full-page reload.

As with `ssr`, you can disable client-side rendering altogether:

```
// src/routes/+page.server.js
export const csr = false;
```

This means that no JavaScript is served to the client, but it also means that your components re no longer interactive. It can be a useful way to test if your application is usable for people who, for whatever reason, cannot use JavaScript.

#### Prerendering
Prerendering means generating HTML for a page once, at build time, rather than dynamically for each request.

The advantage is that serving static data is extremely cheap and performant, allowing you to easily serve large numbers of users without worrying about cache-control headers (which are easy to get wrong).

The tradeoff is that the build process takes longer, and prerendered content can only be updated by building and deploying a new version of the application.

To prerender a page, set `prerender` to `true`.

```
// src/routes/+page.server.js
export const prerender = true;
```

Not all pages can be prerendered. The basic rule is: for content to be prerenderable, any two users hitting it directly must get the same content from the server, and the page must not contain any form actions. Pages with dynamic route parameters can be prerendered as long as they are specified in the `prerender.entries` configuration or can be reached by following links from pages that *are* in `prerender.entries`. 

Note: Setting `prerender` to `true` inside your root `+layout.server.js` effectively turns SvelteKit into a static site generator (SSG).

#### Trailing Slash
Two URLs like `/foo` and `/foo/` might look the same but they're actually different. A relative URL like `./bar` will resolve to `/bar` in the first case and `/foo/bar` in the second, and search engines will treat them as separate entries, harming your SEO.

By default, SvelteKit strips trailing slashes, meaning that a request for `/foo/` will result in a redirect to `/foo`.

If you instead want to ensure that a trailing slash is always present, you can specify the `trailingSlash` option:

```
// src/toutes/always/+page.server.js
export const trailingSlash = 'always';
```

To accommodate both cases (which is not recommended), use `ignore`.

```
// src/routes/ignore/+page.server.js
export const trailingSlash = 'ignore';
```

The default value is `'never'`.

Whether or not trailing slashes are applied affects prerendering. A URL like `/always/` will be saved to disk as `always/index.html` whereas a URL like `/never` will be saved as `never.html`.

### Link Options
#### Preloading
You can't always make your data load more quickly but SvelteKit can speed up navigations by anticipating them.

When an `<a>` element has a `data-sveltekit-preload-data` attribute, SvelteKit will begin the navigation as soon as the user hovers over the link (on desktop) or taps it (on mobile).

```
<nav>
	<a href="/">home</a>
	<a href="/fast" data-sveltekit-preload-data>fast</a>
	<a href="/slow">slow</a>
</nav>
```

Not waiting for a `click` event to be registered typically saves 200ms or more, which is enough to be the difference between feeling sluggish and snappy.

You can put the attribute on individual links, or any element that contains links. The default project template includes the attribute on the `<body>` element.

```
<body data-sveltekit-preload-data>
	%sveltekit.body%
</body>
```

You can customize the behaviour further by specifying one of the following values for the attribute:
- `"hover"` (default, falls back to `"tap"` on mobile)
- `"tap"` - only begin preloading on tap
- `"off"` - disable preloading

Using `data-sveltekit-preload-data` may sometimes result in false positives, such as loading data in anticipation of a navigation that doesn't then happen, which might be undesirable.

As an alternative, `data-sveltekit-preload-code` allows you to preload the JavaScript needed by a given route without eagerly loading its data. This attribute can have the following values:

- "eager" - preload everything on the page following a navigation
- "viewport" - preload everything as it appears in the viewport
- "hover" - (default) same as above
- "tap" - same as above
- "off" - same as above

You can also initiate preloading programmatically with `preloadCode` and `preloadData` imported from `$app/navigation`:

```
import { preloadCode, preloadData } from '$app/navigation';

// preload the code and data needed to navigate to /foo
preloadData('/foo');

// preload the code needed to navigate to /bar, but not the data
preloadCode('/bar');
```

#### Reloading the page
Ordinarily, SvelteKit will navigate between pages without refreshing the page. In rare cases you might want to disable this behaviour. You can do so by adding the `data-sveltekit-reload` attribute on an individual link, or nay element that contains links:

```
<nav data-sveltekit-reload>
	<a href="/">home</a>
	<a href="/about">about</a>
</nav>
```

### Advanced Routing
#### Optional Parameters
Sometimes it's helpful to make a parameter optional. A classic example is when you use the pathname to determine the locale, `/fr/...`, `/de/...` and so on, but you also want to have a default locale.

To do that, use double brackets, for example `src/routes/[[lang]]/+page.svelte`.

```
// src/routes/[[lang]]/+page.server.js
const greetings = {
	en: 'hello!',
	de: 'hallo!',
	fr: 'bonjour!'
};

export function load({ params }) {
	return {
		greeting: greetings[params.lang ?? 'en']
	};
}
```

#### Rest Parameters
To match an unknown number of path segments, use a `[...rest]` parameter, so named for its resemblance to rest parameters in JavaScript.

So `src/routes/[...path]` would match any path.

Note: Other, more specific routes will be tested first making rest parameters useful as "catch-all" routes. For example, if you needed a custom 404 page for pages inside `/categories/...` you could add `src/routes/categories/[...catchall]/+error.svelte` and `src/routes/categories/[...catchall]/+page.server.js`.

Rest parameters do not need to go at the end. A route like `/items/[...path]/edit` or `/items/[...path].json` would be totally valid.

#### Param Matchers
To prevent the router from matching on an invalid input, you can specify a matcher. For example, you might want a route like `/colors/[value]` to match hex values like `/colors/ff3e00` but not named colours like `/colors/octarine` or any other arbitrary input.

Export a `match` function from `src/params`

```
src/params/hex.js
export function match(value) {
	return /^[0-9a-f]{6}$/.test(value);
}
```

Then, to use the matcher, name the route to include the matcher `src/routes/colors/[color=hex]`

Now whenever someone navigates to that route, SvelteKit will verify that `color` is a valid `hex` value. If not, SvelteKit will try to match other routes, before eventually returning a 404.

Note: Matchers run on both the server and in the browser.

#### Route Groups
Layouts let you share UI and data loading logic between different routes. Sometimes it's useful to use layouts without affecting the route. For example you might need `/app` and `/account` to be behind authentication but `/about` to be open to the world. You can do this with a route group, which is a directory in parentheses.

For example, you can create an `(authed)` group: `src/routes/(authed)/+layout.server.js`

```
import { redirect } from '@sveltejs/kit';

export function load({ cookies, url }) {
	if (!cookies.get('logged_in')) {
		redirect(303, `/login?redirectTo=${url.pathname}`);
	}
}
```

If you try to visit pages like `(authed)/app` or `(authed)/account`, you'll be redirected to the `/login` route where you can use a form action to set a `logged_in` cookie.

You can also add some UI to the authed routes using `src/routes/(authed)/+layout.svelte`.

```
<script lang="ts">
	let { children } = $props();
</script>

{@render children()}

<form method="POST" action="/logout">
	<button>log out</button>
</form>
```

#### Breaking Out of Layouts
Ordinarily a page inherits every layout above it. So `src/routes/a/b/c/+page.svelte` inherits layouts from:

-  `src/routes/+layout.svelte`
- `src/routes/a/+layout.svelte`
- `src/routes/a/b/+layout.svelte`
- `src/routes/a/b/c/+layout.svelte`

Occasionally, it's useful to break out of the current layout hierarchy. You can do this by adding the `@` sign followed by the name of the parent segment to "reset" to. For example, `+page@b.svelte` would put `a/b/c` inside `src/routes/a/b/+layout.svelte` while `+page@a.svelte` would put it inside `src/routes/a/+layout.svelte`. 

`+page@.svelte` would reset all the way to the root layout.

Note: The root layout applies to every page of your app, you cannot break out of it.

### Advanced Loading
#### Universal Load Functions
Loading data from the server using `+page.server.js` and `+layout.server.js` is very convenient if you need to do things like getting data directly from a database, or reading cookies.

Sometimes it doesn't make sense to load data from the server when doing a client-side navigation. For example:

- You're loading data from an external API
- You want to use in-memory data if it's available
- You want to delay navigation until an image has been preloaded, to avoid pop-in
- You need to return something from `load` that can't be serialized (SvelteKit uses devalue to turn server data into JSON), such as a component or store

To turn a server `load` function into a universal `load` function, rename it from `+page.server.js` to `+page.js`. Now the function will run on the server during server-side rendering, but will also run int he browser when the app hydrates or the user performs a client-side navigation.

#### Using Both Load Functions
Occasionally you might need to use a server load function and a universal load function together. For example, you might need to return data from the server, but also return a value that can't be serialized as server data.

As an example, you want to return a different component from `load` depending on whether the data you got from `src/routes/+page.server.js` is `cool` or not.

You can access server data in `src/routes/+page.js` via the `data` property.

```
// src/routes/+page.js
export async function load({ data }) {
	const module = data.cool
		? await import('./CoolComponent.svelte')
		: await import('./BoringComponent.svelte');

	return {
		component: module.default,
		message: data.message
	};
}
```

Note: the data isn't merged, you must explicitly return `message` from the universal `load` function.

#### Using Parent Data
`+page.svelte` and `+layout.svelte` components have access to everything returned from their parent `load` functions.

Occasionally it's useful for the `load` functions themselves to access data from their parents. This can be done with `await parent()`

```
// src/routes/+layout.server.js
export function load() {
	return { a: 1 };
}
```

```
// src/routes/sum/+layout.js
export async function load({ parent }) {
	const { a } = await parent();
	return { b: a + 1 };
}
```

```
// src/routes/sum/+page.js
export async function load({ parent }) {
	const { a, b } = await parent();
	return { c: a + b };
}
```

Note: a universal `load` function can get data from a parent server `load` function. The reverse is not true. A server `load` function can only get parent data from another server `load` function.

Take care not to introduce waterfalls when using `await parent()`. If you can `fetch` other data that is not dependent on parent data, do that first.

#### Invalidation
When a user navigates from one page to another, SvelteKit calls your `load` functions, but only if it thinks something has changed.

You can manually invalidate by using the `invalidate(...)` function, which takes a URL and re-runs any `load` functions that depend on that. For example, if a `load` function in `src/routes/+layout.js` calls `fetch("/api/now")` it depends on `/api/now`.

In this `src/routes/[...timezone]/+page.svelte` example, you can add an `onMount` callback that calls `invalidate(/api/now)` once a second:

```
// src/routes/[...timezone]/+page.svelte
<script lang="ts">
	import { onMount } from 'svelte';
	import { invalidate } from '$app/navigation';

	let { data } = $props();

	onMount(() => {
		const interval = setInterval(() => {
			invalidate('/api/now');
		}, 1000);

		return () => {
			clearInterval(interval);
		};
	});
</script>

<h1>
	{new Intl.DateTimeFormat([], {
		timeStyle: 'full',
		timeZone: data.timezone
	}).format(new Date(data.now))}
</h1>
```

Note: You can also pass a function to `invalidate` in case you want to invalidate based on a pattern and not specific URLs.

#### Custom Dependencies
Calling `fetch(url)` inside a `load` function registers `url` as a dependency. Sometimes it's not appropriate to use `fetch`, in which case you can specify a dependency manually with the `depends(url)` function.

Since any string that begins with an `[a-z]+:` pattern is a valid URL, you can create custom invalidation keys like `data:now`.

```
// src/routes/+layout.js
export async function load({ depends }) {
	depends('data:now');

	return {
		now: Date.now()
	};
}
```

```
// src/routes/[...timezone]/+page.svelte
<script lang="ts">
	import { onMount } from 'svelte';
	import { invalidate } from '$app/navigation';

	let { data } = $props();

	onMount(() => {
		const interval = setInterval(() => {
			invalidate('data:now');
		}, 1000);

		return () => {
			clearInterval(interval);
		};
	});
</script>
```

#### `invalidateAll`
`invalidateAll` is kind of the nuclear option. This will indiscriminately re-run all `load` functions for the current page, regardless of what they depend on.

```
// src/routes/[...timezone]/+page.svelte
<script lang="ts">
	import { onMount } from 'svelte';
	import { invalidateAll } from '$app/navigation';

	let { data } = $props();

	onMount(() => {
		const interval = setInterval(() => {
			invalidateAll();
		}, 1000);

		return () => {
			clearInterval(interval);
		};
	});
</script>
```

So the `depends` call is no longer necessary.

Note: `invalidate(() => true)` and `invalidateAll` are *not* the same. `invalidateAll` also re-runs `load` functions without any `url` dependencies, which `invalidate(() => true)` does not.

### Environment Variables
#### `$env/static/private`
Environment variables, like API keys and database credentials, can be added to a `.env` file and they will be made available to your SvelteKit app.

Note: You can also use `.env.local` or `.env.[mode]` files (see the Vite docs). Make sure you add these sensitive files to your `.gitignore` file. Environment variables in `process.env` are also available via `$env/static/private`.

Environment variables can be imported from `$env/static/private`:

```
// src/routes/+page.server.js
import { redirect, fail } from '@sveltejs/kit';
import { PASSPHRASE } from '$env/static/private';

export function load({ cookies }) {
	if (cookies.get('allowed')) {
		redirect(307, '/welcome');
	}
}

export const actions = {
	default: async ({ request, cookies }) => {
		const data = await request.formData();

		if (data.get('passphrase') === PASSPHRASE) {
			cookies.set('allowed', 'true', {
				path: '/'
			});

			redirect(303, '/welcome');
		}

		return fail(403, {
			incorrect: true
		});
	}
};
```

It is important that sensitive data doesn't accidentally end up being sent to the browser, where it could easily be stolen.

SvelteKit prevents this. If you try to import a variable from `$env/static/secret` into a `+page.svelte` file, you will get an error overlay popup telling you that `$env/static/private` cannot be imported into client-side code. It can only be imported into server modules:

- `+page.server.js`
- `+layout.server.js`
- `+server.js`
- any module ending with `.server.js`
- any modules inside `src/lib/server`

In turn, these modules can only be imported by *other* server modules.

The `static` in `$env/static/private` indicates that these values are known at build time and can be statically replaced. This enables useful optimizations.

```
import { FEATURE_FLAG_X } from '$env/static/private';

if (FEATURE_FLAG_X === 'enabled') {
	// code in here will be removed from the build output
	// if FEATURE_FLAG_X is not enabled
}
```

#### `$env/dynamic/private`
If you need to read the values of environment variables when the app runs, as opposed to when the app is built, you can use `$env/dynamic/private` instead of `$enc/static/private`.

```
import { env } from '$env/dynamic/private';
...
if (data.get('passphrase') === env.PASSPHRASE) {
	cookies.set('allowed', 'true', {
		path: '/'
	});

	redirect(303, '/welcome');
}
```

#### `$env/static/public`
Some environment variables can be safely exposed to the browser. These are distinguished from private environment variables with a `PUBLIC_` prefix.

These can then be imported from `$env/static/public`.

```
import {
		PUBLIC_THEME_BACKGROUND,
		PUBLIC_THEME_FOREGROUND
	} from '$env/static/public';
```

#### `$env/dynamic/public`
As with private environment variables, it's preferable to use static values if possible, but if necessary you can use dynamic values instead.

```
import { env } from '$env/dynamic/public';
...
<main
	style:background={env.PUBLIC_THEME_BACKGROUND}
	style:color={env.PUBLIC_THEME_FOREGROUND}
>...</main>
```