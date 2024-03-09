# CSS architecture

The framework tries to achieve *simplicity* for developers on large-scale projects. It is *flexible* and *extensible* in the choices you can make.

**Note**: this framework is implemented in a CSS-starter called [Feo.css](https://feo.crinkles.dev).

## Guiding principles

1. Use CSS Custom Properties as global design tokens
2. Separate structure from skin
3. Define APIs for classes, through (scoped) custom properties

## High-level architecture 
To adhere to these guiding principles, a simple four-layered architecture inspired by [CUBE CSS](https://cube.fyi) is used.

- **Global**: `reset.css`, global styling settings for HTML elements that act as a default. In addition, this layer contains design tokens.
- **Layout**: classes that dictate the browser how to layout elements in relation to each other and the available space. They represent the structure. Examples: [sidebar layout](https://feo.crinkles.dev/layouts/sidebar/), or [switcher](https://feo.crinkles.dev/layouts/switcher/).
- **Blocks**: classes representing a component or organisational structure (where layout classes fall short). 
- **Utilities**: classes that do one job and do one job well. This is a class that alters a single property. But utilities like the [`.click-area` class](https://feo.crinkles.dev/utilities/) cover more than a single property but still do one thing.

**Tip 1**: utilise `@layer` to define layers that can be used for easier organisation of classes. New classes can be added to existing layers, resulting in less cascading issues and more control. 

## Principle 1: Use CSS custom properties as global design tokens
Design tokens, such as *colors*, *spacing*, *font-sizes*, and *breakpoints*, should be defined in CSS custom properties. Based on the design tokens, you can define utility classes, or use them in blocks.  

```css
:root {
	--token-primary: #000;
}
/* Utility class example */
.bg-primary {
	background-color: var(--token-primary);
}
/* Block example */
.site-footer {
	padding: 1rem;
	color: var(--token-primary);
}
```

## Principle 2: Separate structure from skin
The defining principle for the *layout* layer in the architecture. There are many common and overlapping (responsive) layout patterns. By separating these and reusing them as much as possible, you can achieve consistency in implementation, and maintainability in code. [Feo.css](https://feo.crinkles.dev/layouts/) outlines a good set of layout patterns. 

Let’s take the `.switcher` layout class as an example. This patterns switches from a horizontal orientation, to a vertical orientation, if the available space becomes less than a given threshold. 

```css
.switcher {
	display: flex;
	flex-wrap: wrap;
	/* look, a design token */
	gap: var(--token-size-0);
}

.switcher > * {
	/* --token-bp-0 is a breakpoint design token */
	flex-basis: calc((var(--token-bp-0) - 100%) * 999);
}
```

Now on an article page there is a “pagination” at the bottom to go to the previous or next page. You can still define a block called `.pagination` with additional styles or *skin*. 

```css
.pagination {
	width: 100%;
	padding: 1rem;
	/* look, a design token */
	font-size: var(--token-size-2);
	background-color var(--token-primary);
}
```

Now you can apply both the block class, as the generic layout class to the element, and avoid duplicate code. 

```html
<div class="pagination | switcher">...</div>
```

**Tip 2**: you can use characters like “|” to group classes visually (unless you define a class called `.|`). 

## Principle 3: Define APIs for classes, through (scoped) custom properties

Many classes have properties that are not the same in all scenarios. The `.switcher` example you want to be able to adjust the gap or the width that triggers the orientation change.  

You could add a class that alters that property. That is not the case for all APIs. The example `.switcher` pattern used before shows that `var(--token-bp-0)` is used in a calculation. The token is used in a child selector (`>`), not directly on `.switcher`.  Another scenario is a token that is used for multiple properties. 

*To avoid overwriting all these scenarios one by one, we define clear APIs through custom properties.*  

Let’s take the previous `.switcher` layout class as an example. Instead if directly setting the properties, we define two APIs for the class to control it. 

```css
.switcher {
	--layout-gap: var(--token-size-0);
	--layout-width: var(--token-bp-0);
}
```

With this API different methods can be used to control the results. 

### Class utilities
Utility classes do one *independent* thing, not restricted by anything else. Class utilities are similar, as they also do one thing. But, they need to be used with an additional class, as they change a property, or an *API*, of that class. 

```css
.--gap-1 {
	--layout-gap: var(--token-gap-1);
}
```

The above example is a *class utility*. It changes the value of the API of the `.switcher` layout class. In a similar way, this idea can be applied to blocks as well. This allows you, for instance, to change the background color of specific elements of a block with a class utility (similar to a modifier of the BEM methodology). 

```css
.card {
	--card-bg: white;
}

.card-header {
	background-color: var(--card-bg);
}

.--bg-black {
	--card-bg: black;
}
```

**Tip 3**: elements don’t often have multiple layout patterns applied. Most layout patterns share a similar API (e.g. gap, a width modifier, etc.). If you reuse the APIs, you only need one set of class utilities to control them. In the above example you can use `--width-0` for different layout patterns to control the `--layout-width` API. Even if they do different things per pattern. 

**Tip 4**: map class utilities to the global design tokens. If you need values deviating from the global design tokens, it will likely be an exception. Treat it as such.  

### `data-*` properties
Instead of class utilities, you can also use `data-*` properties to invoke a ‘modifier’ (in BEM-terms)on the API values. This is an ideal way to invoke changes on multiple API values. 

```css
.button { ... }
.butten[data-variant="primary"] {
	--button-bg: green;
	--button-text: white;
}
```

**Tip 5**: If you need to change one value, use a class utility. If you need to change multiple values that are always combined, use a `data-*` selector.

### Overwrite in block classes
Sometimes you have exceptional cases where you need a value that deviates from the global design tokens. For instance, you require the `.switcher` to invoke his orientation change at a certain breakpoint, but taking into account a certain padding. Because an API has been defined, you can overwrite it in your block class.

```css
.article {
	--layout-width: calc(var(--token-bp-0) + 2 * var(--token-size-0));
}
```

This allows you to treat these scenarios as exceptions that are implemented without overhead, expanding upon the global design tokens on each exception, etc.