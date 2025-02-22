---
extends: _layouts.documentation
title: "Preflight"
description: "An opinionated set of base styles for Tailwind projects."
titleBorder: true
---

<h2 style="font-size: 0" class="invisible m-0 -mb-6">Overview</h2>

Built on top of [normalize.css](https://github.com/necolas/normalize.css/), Preflight is a set of base styles for Tailwind projects that are designed to smooth over cross-browser inconsistencies and make it possible for you work within the constraints of your design system.

Tailwind automatically injects these styles when you include `@@tailwind base` in your CSS:

```css
@@tailwind base; /* Preflight will be injected here */

@@tailwind components;

@@tailwind utilities;
```

While most of the styles in Preflight are meant to go unnoticed — they are designed to make things behave more like you'd expect them to — some are more opinionated and can be surprising when you first encounter them.

For a complete reference of all the styles applied by Preflight, [see the stylesheet](https://unpkg.com/tailwindcss/dist/base.css).

---

## Default margins are removed

Preflight removes all of the default margins from elements like headings, blockquotes, paragraphs, etc.

```css
blockquote,
dl,
dd,
h1,
h2,
h3,
h4,
h5,
h6,
figure,
p,
pre {
  margin: 0;
}
```

This makes it harder to accidentally rely on margin values applied by the user-agent stylesheet that are not part of your spacing scale.

---

## Headings are unstyled

All heading elements are completely unstyled by default, and have the same font-size and font-weight as normal text.

```css
h1,
h2,
h3,
h4,
h5,
h6 {
  font-size: inherit;
  font-weight: inherit;
}
```

The reason for this is two-fold:

- **It helps you avoid accidentally deviating from your type scale**. By default, the browsers assigns sizes to headings that don't exist in Tailwind's default type scale, and aren't guaranteed to exist in your own type scale.
- **In UI development, headings should often be visually de-emphasized**. Making headings unstyled by default means any styling you apply to headings happens consciously and deliberately.

You can always add default header styles to your project by [adding your own base styles](/docs/adding-base-styles).

---

## Lists are unstyled

Ordered and unordered lists are unstyled by default, with no bullets/numbers and no margin or padding.

```css
ol,
ul {
  list-style: none;
  margin: 0;
  padding: 0;
}
```

If you'd like to style a list, you can do so using the [list-style-type](/docs/list-style-type) and [list-style-position](/docs/list-style-position) utilities:

```html
<ul class="list-disc list-inside">
  <li>One</li>
  <li>Two</li>
  <li>Three</li>
</ul>
```

You can always add default list styles to your project by [adding your own base styles](/docs/adding-base-styles).


---

## Images are block-level

Images and other replaced elements (like `svg`, `video`, `canvas`, and others) are `display: block` by default.

```css
img,
svg,
video,
canvas,
audio,
iframe,
embed,
object {
  display: block;
  vertical-align: middle;
}
```

This helps to avoid unexpected alignment issues that you often run into using the browser default of `display: inline`.

If you ever need to make one of these elements `inline` instead of `block`, use the `inline` utility:

```html
<img class="inline" src="..." alt="...">
```

---

## Border styles are reset globally

In order to add a border by adding the `border` class, Tailwind overrides the default border styles for all elements with the following rules:

```css
*,
*::before,
*::after {
  border-width: 0;
  border-style: solid;
  border-color: theme('borderColor.default', currentColor);
}
```

Since the `border` class only sets the `border-width` property, this reset ensures that adding that class always adds a solid 1px border using your configured default border color.

This can cause some unexpected results when integrating certain third-party libraries, like [Google maps](https://github.com/tailwindcss/tailwindcss/issues/484) for example.

When you run into situations like this, you can work around them by overriding the Preflight styles with your own custom CSS:

```css
.google-map * {
  border-style: none;
}
```

---

## Extending Preflight

If you'd like to add your own base styles on top of Preflight, add them to your CSS after `@@tailwind base`:

```css
@@tailwind base;

h1 {
  @@apply text-2xl;
}
h2 {
  @@apply text-xl;
}
h3 {
  @@apply text-lg;
}
a {
  @@apply text-blue-600 underline;
}

@@tailwind components;

@@tailwind utilities;
```

Learn more in the [adding base styles documentation](/docs/adding-base-styles).

---

## Disabling Preflight

If you'd like to completely disable Preflight — perhaps because you're integrating Tailwind into an existing project or because you'd like to provide your own base styles — all you need to do is set `preflight` to `false` in the `corePlugins` section of your `tailwind.config.js` file:

@component('_partials.customized-config', ['key' => 'corePlugins'])
+ preflight: false,
@endcomponent
