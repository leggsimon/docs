---
extends: _layouts.documentation
title: "Upgrading from v0.x to v1.0"
description: "A guide for upgrading your existing Tailwind CSS projects to v1.0."
titleBorder: true
---

Tailwind v1.0 is mostly focused on changing things from 0.x that I would have done differently had I known in advance where the feature set would be at today.

So while there's not a ton of exciting new features, you can at least be excited about the fact that we now have a really stable base to build on, and that you can feel comfortable using Tailwind in production if the 0.x label gave you pause.

## Upgrade steps for all users

These changes affect all users, whether you are using Tailwind with PostCSS and your own custom config file, or using the default config file or CDN.

1. [Update Tailwind](#1-update-tailwind)
2. [Update your config file](#2-update-your-config-file)
3. [Rename `tailwind.js` to `tailwind.config.js`](#3-rename-tailwind-js-to-tailwind-config-js)
4. [Replace `@@tailwind preflight` with `@@tailwind base`](#4-replace-tailwind-preflight-with-tailwind-base)
5. [Replace `config()` with `theme()`](#5-replace-config-with-theme)
6. [Explicitly style any headings](#6-explicitly-style-any-headings)
7. [Explicitly style any lists that should have bullets/numbers](#7-explicitly-style-any-lists-that-should-have-bullets-numbers)
8. [Remove any usage of `.list-reset`](#8-remove-any-usage-of-list-reset)
9. [Replace `.pin-{side}` with `.{top|left|bottom|right|inset}-{value}`](#9-replace-pin-side-with-top-left-bottom-right-inset-value)
10. [Replace `.roman` with `.not-italic`](#10-replace-roman-with-not-italic)
11. [Replace `.flex-no-grow/shrink` with `.flex-grow/shrink-0`](#11-replace-flex-no-grow-shrink-with-flex-grow-shrink-0)
12. [Explicitly add color and underline styles to links](#12-explicitly-add-color-and-underline-styles-to-links)
13. [Add `inline` to any replaced elements (`img`, `video`, etc.) that should not be `display: block`](#13-add-inline-to-any-replaced-elements-img-video-etc-that-should)
14. [Adjust the line-height and padding on your form elements](#14-adjust-the-line-height-and-padding-on-your-form-elements)
15. [Adjust the text color on your form elements](#15-adjust-the-text-color-on-your-form-elements)
16. [Double check your default font family](#16-double-check-your-default-font-family)
17. [Double check your default line-height](#17-double-check-your-default-line-height)

<h3 class="no-toc">1. Update Tailwind</h3>

Install the latest version of Tailwind:

```bash
npm install tailwindcss@^1.0 --save-dev
```

Or using Yarn:

```bash
yarn add -D tailwindcss@^1.0
```

<h3 class="no-toc">2. Update your config file</h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: All users, Effort: Moderate</p>

This is really the big change in v1.0 — you can read all about the new config file format and motivation behind it in [the initial pull request](https://github.com/tailwindcss/tailwindcss/pull/637).

The new general config structure looks like this:

```js
module.exports = {
  prefix: '',
  important: false,
  separator: ':',
  theme: {
    colors: { ... },
    // ...
    zIndex: { ... },
  },
  variants: {
    appearance: ['responsive'],
    // ...
    zIndex: ['responsive'],
  },
  plugins: [
    // ...
  ],
}
```

See the new [default config file](https://github.com/tailwindcss/tailwindcss/blob/master/stubs/defaultConfig.stub.js) for a complete example.

There are a lot of changes here but they are all fairly cosmetic and entirely localized to this one file, so while it may look intimidating it's actually only 10-15 minutes of work.

#### 2.1. Move all design-related top-level keys into a new section called `theme`

Every key except `options`, `modules`, and `plugins` should be nested under a new `theme` key.

Your config file should look generally like this at this point:

```diff
  let defaultConfig = require('tailwindcss/defaultConfig')()

  let colors = {
    // ...
  }

  module.exports = {
-   colors: colors,
-   screens: {
-     // ...
-   },
-   // ...
-   zIndex: {
-     // ...
-   },
+   theme: {
+     colors: colors,
+     screens: {
+       // ...
+     },
+     // ...
+     zIndex: {
+       // ...
+     },
+   },
    modules: {
      appearance: ['responsive'],
      // ...
      zIndex: ['responsive'],
    },
    plugins: [
      require('tailwindcss/plugins/container')({
        // ...
      }),
    ],
    options: {
      prefix: '',
      important: false,
      separator: ':',
    }
  }
```

#### 2.2. Rename `modules` to `variants`

"Modules" was a word we just kinda grabbed because we needed *something*, and we wanted to use that section of the config to both specify variants and disable modules if necessary.

Now that all of Tailwind's internal "modules" are now core plugins, I've decided to deprecate this terminology entirely, and make this section of the config purely about configuring variants for core plugins.

After making this change, your config file should look like this:

```diff
  let defaultConfig = require('tailwindcss/defaultConfig')()

  let colors = {
    // ...
  }

  module.exports = {
    theme: {
      // ...
    },
-   modules: {
+   variants: {
      appearance: ['responsive'],
      backgroundAttachment: ['responsive'],
      backgroundColors: ['responsive', 'hover', 'focus'],
      // ...
      zIndex: ['responsive'],
    },
    plugins: [
      require('tailwindcss/plugins/container')({
        // ...
      }),
    ],
    options: {
      prefix: '',
      important: false,
      separator: ':',
    }
  }
```

#### 2.3. Move your `options` settings to the top-level

The advanced options have been moved to the top-level of the config file instead of being nested under the redundant `options` key.

After making this change, your config file should look like this:

```diff
  let defaultConfig = require('tailwindcss/defaultConfig')()

  let colors = {
    // ...
  }

  module.exports = {
+   prefix: '',
+   important: false,
+   separator: ':',
    theme: {
      // ...
    },
    variants: {
      appearance: ['responsive'],
      backgroundAttachment: ['responsive'],
      backgroundColors: ['responsive', 'hover', 'focus'],
      // ...
      zIndex: ['responsive'],
    },
    plugins: [
      require('tailwindcss/plugins/container')({
        // ...
      }),
    ],
-   options: {
-     prefix: '',
-     important: false,
-     separator: ':',
-   }
  }
```

#### 2.4. Move your `negativeMargin` config to `margin`

Negative margins are now handled by the same core plugin that handles positive margins.

Take any values you had configured under `negativeMargin` and move them to `margin`, being sure to add a leading dash to the keys and making the actual value negative.

```diff
  margin: {
    'auto': 'auto',
    'px': '1px',
    '0': '0',
    '1': '0.25rem',
    '2': '0.5rem',
    '3': '0.75rem',
    '4': '1rem',
    '5': '1.25rem',
    '6': '1.5rem',
    '8': '2rem',
    '10': '2.5rem',
    '12': '3rem',
    '16': '4rem',
    '20': '5rem',
    '24': '6rem',
    '32': '8rem',
+   '-px': '-1px',
+   '-1': '-0.25rem',
+   '-2': '-0.5rem',
+   '-3': '-0.75rem',
+   '-4': '-1rem',
+   '-5': '-1.25rem',
+   '-6': '-1.5rem',
+   '-8': '-2rem',
+   '-10': '-2.5rem',
+   '-12': '-3rem',
+   '-16': '-4rem',
+   '-20': '-5rem',
+   '-24': '-6rem',
+   '-32': '-8rem',
  },
- negativeMargin: {
-   'px': '1px',
-   '0': '0',
-   '1': '0.25rem',
-   '2': '0.5rem',
-   '3': '0.75rem',
-   '4': '1rem',
-   '5': '1.25rem',
-   '6': '1.5rem',
-   '8': '2rem',
-   '10': '2.5rem',
-   '12': '3rem',
-   '16': '4rem',
-   '20': '5rem',
-   '24': '6rem',
-   '32': '8rem',
- },
```

Note that the class names themselves have not changed — while you might expect that a key like `-6` would generate classes like `mx--6`, the `margin` plugin is intelligent enough to detect those keys and create the class names you're used to like `-mx-6`.

#### 2.5. Update the sections under `theme` to their new names

As part of an effort to make the naming in the config file more consistent, many of the sections under `theme` have been renamed.

These are the sections that need to be updated:

| Old | New |
|---|---|
| `fonts` | `fontFamily` |
| `textSizes` | `fontSize` |
| `fontWeights` | `fontWeight` |
| `leading` | `lineHeight` |
| `tracking` | `letterSpacing` |
| `textColors` | `textColor` |
| `backgroundColors` | `backgroundColor` |
| `borderWidths` | `borderWidth` |
| `borderColors` | `borderColor` |
| `shadows` | `boxShadow` |
| `svgFill` | `fill` |
| `svgStroke` | `stroke` |

These names need to change in the `variants` section as well, so feel free to do a find and replace across the whole file.

#### 2.6. Update the sections under `variants` to their new names

As alluded to in the previous step, many of the sections under `variants` have been renamed as well.

These are the sections that need to be renamed *(it is the same as the list above)*:

| Old | New |
|---|---|
| `fonts` | `fontFamily` |
| `textSizes` | `fontSize` |
| `fontWeights` | `fontWeight` |
| `leading` | `lineHeight` |
| `tracking` | `letterSpacing` |
| `textColors` | `textColor` |
| `backgroundColors` | `backgroundColor` |
| `borderWidths` | `borderWidth` |
| `borderColors` | `borderColor` |
| `shadows` | `boxShadow` |
| `svgFill` | `fill` |
| `svgStroke` | `stroke` |

Several sections under `variants` have also been **split** into multiple sections, for example `lists` has been split into `listStylePosition` and `listStyleType`:

```diff
  // ...

  module.exports = {
    // ...
    variants: {
      // ...
-     lists: ['responsive'],
+     listStylePosition: ['responsive'],
+     listStyleType: ['responsive'],
    }
  }
```

Here is a complete list of the sections that been split into multiple sections:

| Old | New |
| --- | --- |
| `flexbox` | `flexDirection`<br>`flexWrap`<br>`alignItems`<br>`alignSelf`<br>`justifyContent`<br>`alignContent`<br>`flex`<br>`flexGrow`<br>`flexShrink` |
| `lists` | `listStylePosition`<br>`listStyleType` |
| `position` | `position`<br>`inset` |
| `textStyle` | `fontStyle`<br>`fontSmoothing`<br>`textDecoration`<br>`textTransform` |
| `whitespace` | `whitespace`<br>`wordBreak` |

Note that in some cases (`position`, `whitespace`) the original section still exists, while in others (`flexbox`, `textStyle`), the original section has been completely removed.

You should reference the new [default config file](https://github.com/tailwindcss/tailwindcss/blob/master/stubs/defaultConfig.stub.js) if you are ever unsure if you are making the right changes.

One way to make these changes is to copy the value you were using for the old section (something like `['responsive']`) to all of the new sections that replace that section, but if you choose you can also use this as an opportunity to cull generated utilities you don't actually need.

For example, if you never use the responsive variants of `antialiased` or `subpixel-antialiased`, you could set `fontSmoothing` to `[]` while still using `['responsive']` for `fontStyle`, `textDecoration`, and `textTransform`.

#### 2.7. Add any disabled ~~modules~~ core plugins to `corePlugins`

In v0.x, you could disable a ~~module~~ core plugin by setting it to `false` in what is now the `variants` section.

In v1.0, to disable a plugin you need to set it to `false` in the `corePlugins` section instead:

```diff
  let defaultConfig = require('tailwindcss/defaultConfig')()

  let colors = {
    // ...
  }

  module.exports = {
    prefix: '',
    important: false,
    separator: ':',
    theme: {
      // ...
    },
    variants: {
      // ...
-     float: false,
      // ...
    },
+   corePlugins: {
+     float: false,
+   },
    plugins: [
      require('tailwindcss/plugins/container')({
        // ...
      }),
    ],
  }
```

This change was made to make it possible to disable other core plugins where `variants` are irrelevant, like `preflight` or `container` (more on this later).

#### 2.8. Remove the `container` plugin from `plugins` and move any configuration to `theme`

In v1.0, the `container` plugin is a core plugin like `padding`, `margin`, etc. and should not be listed in your `plugins` section:

```diff
  let defaultConfig = require('tailwindcss/defaultConfig')()

  let colors = {
    // ...
  }

  module.exports = {
    prefix: '',
    important: false,
    separator: ':',
    theme: {
      // ...
    },
    variants: {
      // ...
    },
    plugins: [
-     require('tailwindcss/plugins/container')({
-       center: true,
-       padding: '1rem',
-     }),
    ],
  }
```

If you had already removed the container plugin because you don't want those classes in your project, you should explicitly disable it using the `corePlugins` option:

```diff
  let defaultConfig = require('tailwindcss/defaultConfig')()

  let colors = {
    // ...
  }

  module.exports = {
    prefix: '',
    important: false,
    separator: ':',
    theme: {
      // ...
    },
    variants: {
      // ...
    },
+   corePlugins: {
+     container: false
+   },
  }
```

If you are taking advantage of the `center` or `padding` options exposed by the `container` plugin, you should specify those options under `theme.container` instead of as direct arguments to the plugin.

```diff
  let defaultConfig = require('tailwindcss/defaultConfig')()

  let colors = {
    // ...
  }

  module.exports = {
    prefix: '',
    important: false,
    separator: ':',
    theme: {
      // ...
+     container: {
+       center: true,
+       padding: '1rem',
+     }
    },
    variants: {
      // ...
    },
    plugins: [
-     require('tailwindcss/plugins/container')({
-       center: true,
-       padding: '1rem',
-     }),
    ],
  }
```

#### 2.9. Inline your `colors` variable into `theme.colors`

In v1.0, it's possible to specify that parts of your theme _depend_ on other parts of your theme, and because of that it's no longer necessary to hold your `colors` in a separate variable.

Start by inlining your `colors` variable directly into `theme.colors`:

```diff
  let defaultConfig = require('tailwindcss/defaultConfig')()

- let colors = {
-   'transparent': 'transparent',
-   'black': '#22292f',
-   // ...
-   'pink-lightest': '#ffebef','
- }

  module.exports = {
    prefix: '',
    important: false,
    separator: ':',
    theme: {
-     colors: colors,
+     colors: {
+       'transparent': 'transparent',
+       'black': '#22292f',
+       // ...
+       'pink-lightest': '#ffebef','
+     },
      // ...
    },
    variants: {
      // ...
    },
    plugins: [],
  }
```

Next, update any sections that were referencing the `colors` variable using the new closure syntax:

```diff
  let defaultConfig = require('tailwindcss/defaultConfig')()

  module.exports = {
    prefix: '',
    important: false,
    separator: ':',
    theme: {
      colors: {
        'transparent': 'transparent',
        'black': '#22292f',
        // ...
        'pink-lightest': '#ffebef','
      },
      // ...
-     backgroundColor: colors,
+     backgroundColor: theme => theme('colors'),
      // ...
-     textColor: colors,
+     textColor: theme => theme('colors'),
      // ...
-     borderColor: global.Object.assign({ default: colors['grey-light'] }, colors),
+     borderColor: theme => ({
+       default: theme('colors.grey-light'),
+       ...theme('colors'),
+     }),
      // ...
    },
    variants: {
      // ...
    },
    plugins: [],
  }
```

#### 2.10. Don't invoke the default config as a function

In v0.x, `require('tailwindcss/defaultConfig')` returned a function that returned the default config when invoked.

In v1.0, it returns the object:

```diff
- let defaultConfig = require('tailwindcss/defaultConfig')()
+ let defaultConfig = require('tailwindcss/defaultConfig')

  module.exports = {
    prefix: '',
    important: false,
    separator: ':',
    theme: {
      // ...
    },
    variants: {
      // ...
    },
    plugins: [],
  }
```

#### 2.11. Remove any configuration you haven't customized

One of the philosophical changes in v1.0 is that we are encouraging people to use their configuration files solely for specifying _changes_ from the default config, rather than including the entire default config _plus_ their changes.

Every single key in the config file is optional (in fact the file itself is optional too), so if there are things you've never customized, you're encouraged to remove them entirely.

For example, if you aren't specifying a custom separator or prefix or enabling the `important` option, you can remove them entirely:

```diff
  let defaultConfig = require('tailwindcss/defaultConfig')

  module.exports = {
-   prefix: '',
-   important: false,
-   separator: ':',
    theme: {
      // ...
    },
    variants: {
      // ...
    },
    plugins: [],
  }
```

Similarly, if you aren't referencing the `defaultConfig` variable anywhere, remove that too:

```diff
- let defaultConfig = require('tailwindcss/defaultConfig')

  module.exports = {
    theme: {
      // ...
    },
    variants: {
      // ...
    },
    plugins: [],
  }
```

If you haven't customized the `opacity` values, remove them:

```diff
  module.exports = {
    theme: {
      // ...
-     opacity: {
-       '0': '0',
-       '25': '.25',
-       '50': '.5',
-       '75': '.75',
-       '100': '1',
-     },
      // ...
    },
    variants: {
      // ...
    },
    plugins: [],
  }
```

We will not change any of this configuration outside of a major version bump, so you are totally safe to depend on inheriting the default values.

The way your configuration is merged with the defaults is designed to be intuitive and mostly just work, but for the curious:

- `prefix` is replaced
- `separator` is replaced
- `important` is replaced
- `theme` is merged one level deep, so if you provide an object for `theme.opacity` it replaces the default `theme.opacity` object
- `variants` is merged one level deep, so if you provide an array for `variants.opacity` it replaces the default `variants.opacity` object
- `plugins` is merged, but the default is an empty array so it's really the same as replacing

It's worth noting that you are not _required_ to remove any redundant configuration, so if you'd prefer to own the entire system and be able to see it all in one place, you're absolutely welcome to keep everything in your config file.

**It's very important to realize that many of the theme values have changed from v0.7.4 to v1.0**, so just because you never customized a value that shipped by default in v0.x, that doesn't guarantee that you are safe to remove it from your config file.

A perfect example of this is colors. The default color palette is completely new in v1.0 with a new naming scheme, so even if you were using the default color palette in v0.x, you're actually using a custom color palette in v1.0.

**Always double check that anything you want to remove is identical to the _new_ default config file values before you remove it.**

<h3 class="no-toc">3. Rename <code>tailwind.js</code> to <code>tailwind.config.js</code></h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: N/A, Effort: Trivial</p>

This is entirely optional but recommended — if you are using the old default config file name (`tailwind.js`), rename it to `tailwind.config.js`.

If you use that file name and keep the file in the root of your project, Tailwind will pick up your config file by default without having to specify the path in your build scripts/configuration.

Here's an example of what I mean using Laravel Mix:

```diff
  mix.postCss('resources/css/app.css', 'public/css/app.css', [
-     require('tailwindcss')('./tailwind.js'),
+     require('tailwindcss'),
    ])
```

If you keep your config file in a different folder, you'll still need to provide the path:

```diff
  mix.postCss('resources/css/app.css', 'public/css/app.css', [
-     require('tailwindcss')('./resources/tailwind.js'),
+     require('tailwindcss')('./resources/tailwind.config.js'),
    ])
```

<h3 class="no-toc">4. Replace <code>@@tailwind preflight</code> with <code>@@tailwind base</code></h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: All users, Effort: Trivial</p>

One of the new features in v1.0 is the ability for plugins to register base styles. As a result, our `preflight` styles are actually now another core plugin, and the general "bucket" for base styles has been renamed from `preflight` to `base`.

Replace any instance of `@@tailwind preflight` in your CSS files with `@@tailwind base`:

```diff
- @@tailwind preflight;
+ @@tailwind base;

  @@tailwind components;

  @@tailwind utilities;
```

If you are using `postcss-import` and relying on our imports instead of the `@@tailwind` directive, replace `@@import "tailwindcss/preflight"` with `@@import "tailwindcss/base"`:

```diff
- @@import "tailwindcss/preflight";
+ @@import "tailwindcss/base";

  @@import "tailwindcss/components";

  @@import "tailwindcss/utilities";
```

<h3 class="no-toc">5. Replace <code>config()</code> with <code>theme()</code></h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: Moderate, Effort: Low</p>

The `config()` helper function that Tailwind makes available to your CSS files has been replaced with a new `theme()` function that is automatically scoped to the `theme` section of your config file and should work as a drop-in replacement:

```diff
  .btn {
-   padding: config('padding.3');
+   padding: theme('padding.3');
    // ...
  }
```

A find and replace across your CSS files that switches `config(` to `theme(` should do it.

<h3 class="no-toc">6. Explicitly style any headings</h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: Moderate, Effort: Moderate</p>

If you are using our `preflight` styles, all `h1-h6` elements are unstyled by default in v1.0.

That means that out of the box, they all have a font-size of `1em` (whatever the parent font size is) and a font-weight of `inherit`, so they look exactly like a `p` tag.

We do this because in web application development it's very common for some piece of text to be a heading semantically, but actually be styled in a much less "in your face" way because it's meant to look more like a subtle label on a section of UI. Because of this, well-designed applications often need to "undo" the user agent styles for headings anyways, and we think it's better for styling to feel additive in Tailwind instead of subtractive.

By using the user agent styles for headings, we also made it far too easy to accidentally deviate from your own design system. If the browser says that an `h1` should be `2em`, it could compute to a size that isn't part of your `fontSize` scale.

By unstyling headings by default, we make it a lot easier to avoid this pitfall by ensuring that any size or weight you set is explicit and intentional.

This change might not affect you at all if you are already specifying a font-weight and font-size on all your headings, but if you aren't, you’ll need to assign an explicit size and weight wherever it's missing:

```diff
- <h1>Manage Account<h1>
+ <h1 class="text-xl font-semibold">Manage Account<h1>
```

The exact changes you need to make will be highly specific to what you want to accomplish with your design, so you'll have to assess each situation independently.

This is a bit of an annoying change, but if it breaks your site, you could argue that it's actually revealing bugs in your markup.

<h3 class="no-toc">7. Explicitly style any lists that should have bullets/numbers</h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: Moderate, Effort: Moderate</p>

If you are using our `preflight` styles, all `ul` and `ol` elements are unstyled by default in v1.0.

That means if you have any lists that depend on the default browser styling (bullets/numbers and a bit of left padding), you need to explicitly style those lists using the new `.list-disc/decimal` utilities and the existing padding utilities:

```diff
- <ul>
+ <ul class="list-disc pl-4">
    <!-- ... -->
  </ul>
```

If you really don't want to do this manually and would prefer that lists be styled by default, you can override our base styles with your own custom CSS by adding a couple of rules like this:

```css
@@tailwind base;

ul {
  list-style-type: disc;
  padding-left: theme('padding.4');
}

ol {
  list-style-type: decimal;
  padding-left: theme('padding.4');
}

@@tailwind components;

@@tailwind utilities;
```

<h3 class="no-toc">8. Remove any usage of <code>.list-reset</code></h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: Moderate, Effort: Low</p>

Since lists are now unstyled by default, `.list-reset` has been removed. You technically don't need to change anything, but you're encouraged to remove any usage of it as it's now dead code:

```diff
- <ul class="list-reset"><!-- ... --></ul>
+ <ul><!-- ... --></ul>
```

If you chose override our base styles and give lists a default style, you can use the new `.list-none` utility as well as `.p-0` as a replacement for `.list-reset` to remove that base styling as needed:

```diff
- <ul class="list-reset"><!-- ... --></ul>
+ <ul class="list-none p-0"><!-- ... --></ul>
```

Again, **if you are using our `preflight` styles unmodified (you probably are), you can remove `list-reset` from your markup and nothing will change**.

This change only really affects you if you are _not_ using our `preflight` styles, or overriding our global list reset.

<h3 class="no-toc">9. Replace <code>.pin-{side}</code> with <code>.{top|left|bottom|right|inset}-{value}</code></h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: High, Effort: Moderate</p>

Utilities like `.pin`, `.pin-x`, and `.pin-t` have been removed in favor of less cleverly named classes like `.top-0`, `.right-0`, etc.

See the [pull request](https://github.com/tailwindcss/tailwindcss/pull/764) for more details on the motivation behind this change.

Here is a complete list of the changes:

| Old | New |
| --- | --- |
| `.pin-none`  | `.inset-auto`  |
| `.pin`  | `.inset-0`  |
| `.pin-y`  | `.inset-y-0`  |
| `.pin-x`  | `.inset-x-0`  |
| `.pin-t`  | `.top-0`  |
| `.pin-r`  | `.right-0`  |
| `.pin-b`  | `.bottom-0`  |
| `.pin-l`  | `.left-0`  |

Six new classes have been added as well:

| Class |
| --- |
| `.inset-y-auto`  |
| `.inset-x-auto`  |
| `.top-auto`  |
| `.right-auto`  |
| `.bottom-auto`  |
| `.left-auto`  |

These are all now customizable in `theme.inset` too, whereas the `pin-{side}` utilities were not.

This is an annoying change, sorry.

<h3 class="no-toc">10. Replace <code>.roman</code> with <code>.not-italic</code></h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: Low, Effort: Low</p>

Previously we used the name `.roman` for `font-style: normal` because of a bug in `postcss-selector-not` that prevented us from using `.not-italic`. That bug has been fixed, so this name has been changed.

```diff
- <div class="roman">
+ <div class="not-italic">
    <!-- ... -->
  </div>
```

I would be surprised if more than 5 people are even affected by this, I've never used this class once myself.

<h3 class="no-toc">11. Replace <code>.flex-no-grow/shrink</code> with <code>.flex-grow/shrink-0</code></h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: High, Effort: Low</p>

In order to make these utilities more easily customizable, their names have changed to match our existing conventions.

```diff
- <div class="flex-no-grow">
+ <div class="flex-grow-0">
    <!-- ... -->
  </div>

- <div class="flex-no-shrink">
+ <div class="flex-shrink-0">
    <!-- ... -->
  </div>
```

These utilities are also now customizable in the `theme.flexGrow` and `theme.flexShrink` sections of your config file.

<h3 class="no-toc">12. Explicitly add color and underline styles to links</h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: High, Effort: Moderate</p>

In v1.0, `a` tags automatically inherit the parent `color` and `text-decoration` styles which means that by default links are no longer blue and do not have an underline.

You are likely already adding a text color class like `text-green-dark` or similar to your links because you probably didn't want the default browser-blue color, but if not you'll need to add a color explicitly:

```diff
- <a href="#">
+ <a href="#" class="text-blue">
    <!-- ... -->
  </a
```

Similarly, if you have any links that need underlines, you'll have to add them manually:

```diff
- <a href="#">
+ <a href="#" class="underline">
    <!-- ... -->
  </a
```

On the flip side, if you are using `no-underline` in a million places across your project just to unstyle links, you can now safely remove that class:

```diff
- <a href="#" class="no-underline">
+ <a href="#">
    <!-- ... -->
  </a
```

If you really don't like these new defaults, you can add your own base link styles after `@@tailwind base`:

```css
@@tailwind base;

a {
  color: theme('colors.blue');
  text-decoration: underline;
}

@@tailwind components;
@@tailwind utilities;
```

<h3 class="no-toc">13. Add <code>inline</code> to any replaced elements (<code>img</code>, <code>video</code>, etc.) that should not be <code>display: block</code></h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: Moderate, Effort: Moderate</p>

In v1.0, all replaced elements (like `img`, `svg`, `video`, `canvas`, `iframe`, etc.) are set to `display: block` by default. This is counter to the browser default which is `inline`.

If you have any instances in your project where you actually want these elements to be `inline`, you'll need to add that class:

```diff
  <span>
-   <img src="..." class="h-4 w-4">
+   <img src="..." class="h-4 w-4 inline">
    Manage
  </span>
```

I don't think this will actually affect many people or projects, as you almost always want these elements to be `block` or you have them nested inside a flex container where it doesn't matter.

<h3 class="no-toc">14. Adjust the line-height and padding on your form elements</h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: High, Effort: Moderate</p>

**If you are already setting an explicit line-height on form elements, this change will not affect you.**

In v0.x, we used a line-height of 1.15 for form elements by default, sort of incidentally by depending on normalize.css.

This made it very easy to forget to add an explicit line-height like `leading-tight` or `leading-normal` to form elements, introducing a new line-height (1.15) into your project that doesn't match any of the `leading-{size}` utilities.

In v1.0, all form elements use a value of `inherit` for their line-height, so the line-height will match the parent element by default.

That means if you had some markup like this:

```html
<div class="leading-normal ...">
  <!-- ... -->
  <input type="text" class="px-4 py-3">
</div>
```

...your `input` element will be slightly taller in v1.0 because the line-height has increased from 1.15 to 1.5.

You can fix this by adjusting any vertical padding to account for the new line-height, and optionally adding an explicit `leading-{size}` class if you don't want to match the line-height of the parent:

```diff
  <div class="leading-normal ...">
    <!-- ... -->
-   <input type="text" class="px-4 py-3">
+   <input type="text" class="px-4 py-2 leading-tight">
  </div>
```

You might not get the exact same height you had before, but that's likely because the old height was some weird fractional number like 42.4px (line-height of 1.15 * font-size of 16px + 24px of padding). With the new system you are much more likely to land on reasonable whole numbers, like 40px of 44px, depending on your chosen line-height and padding values.

If you really want to use 1.15 as your default line-height for form elements (I would recommend against it), you can add a rule like this to your own base styles:

```css
@@tailwind base;

button,
input,
optgroup,
select,
textarea {
  line-height: 1.15;
}

@@tailwind components;
@@tailwind utilities;
```

<h3 class="no-toc">15. Adjust the text color on your form elements</h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: Low, Effort: Moderate</p>

**If you are already setting an explicit text color on form elements, this change will not affect you.**

In v0.x, form elements used black text by default, even though true black was not part of the default color palette.

In v1.0, form elements inherit their text color from the parent, which means if you have any markup like this:

```html
<div class="text-red">
  <input type="text">
</div>
```

...your `input` would have red text instead of black text.

You can fix this by setting a text color on form elements explicitly:

```diff
  <div class="text-red">
-   <input type="text">
+   <input type="text" class="text-grey-darkest">
  </div>
```

<h3 class="no-toc">16. Double check your default font family</h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: Low, Effort: Trivial</p>

**If you are already setting a default font family on your project (either with a class on `html`/`body` or using custom CSS), this change will not affect you.**

In v1.0, the default font family has changed from `sans-serif` to our system font stack.

It's very unlikely that you weren't already overriding this with your own font, but if not you'll notice your site looks a bit different, and honestly probably better.

You don't really have to change anything unless for some unexplainable reason you want to use `sans-serif` as your default font family, in which case you can add a rule to your base styles:

```css
@@tailwind base;

html {
  font-family: sans-serif;
}

@@tailwind components;
@@tailwind utilities;
```

<h3 class="no-toc">17. Double check your default line-height</h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: Moderate, Effort: Moderate</p>

**If you are already setting a default line-height on your project (either with a class on `html`/`body` or using custom CSS), this change will not affect you.**

In v0.x, the default line-height was 1.15 (inherited from normalize.css). Since that value isn't part of Tailwind's default theme, we've opted to change it to 1.5 for v1.0 so the default line-height matches a value in the line-height scale.

This means that if you are not setting a line-height either using a `leading-{size}` class on the `html` or `body` tags or by adding some base styles to your CSS, most things on your site are going to appear a little bit taller.

The easiest solution is to reset the line-height to 1.15 by default:

```css
@@tailwind base;

html {
  line-height: 1.15;
}

@@tailwind components;
@@tailwind utilities;
```

However, a better long-term solution would be to pick a default line-height that matches a value in your line-height scale, and audit your site to find situations where it makes the design look worse and tweak those one at a time.

<hr class="my-16">

<h2>Additional steps for CDN users, or others without a config file</h2>

These steps only affect users that are depending on the 0.x configuration file values. This includes CDN users, or anyone that is omitting sections from their config file, referencing our config file, or not using a config file at all.

1. [Update any usage of `text/bg/border-{color}` classes](#1-update-any-usage-of-text-bg-border-color-classes)
2. [Replace `tracking-tight/wide` with `tracking-tighter/wider`](#2-replace-tracking-tight-wide-with-tracking-tighter-wider)
3. [Check your design against the updated default breakpoints](#3-check-your-design-against-the-updated-default-breakpoints)
4. [Double check any usage of the default `shadow-{size}` utilities](#4-double-check-any-usage-of-the-default-shadow-size-utilities)
5. [Update any usage of the default `max-w-{size}` utilities](#5-update-any-usage-of-the-default-max-w-size-utilities)

<h3 class="no-toc mt-12">1. Update any usage of <code>text/bg/border-{color}</code> classes</h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: Low, Effort: High</p>

**This change only affects you if you don't have a color palette defined in your config file or you are using Tailwind through a CDN.**

Tailwind v1.0 comes with an entirely new color palette that provides 9 shades for each color instead of 7 ([\#737](https://github.com/tailwindcss/tailwindcss/pull/737)).

The naming scheme has changed from using words like `darkest` and `lighter` to a numeric scaled inspired by Material Design that starts at `100` for the lightest shade and ends at `900` for the darkest shade.

There is no way to map the old colors to the new colors 1:1 because the new palette includes more shades, so if you are using the v0.x default color palette and would like to upgrade to the new color palette, you are in for some fun (you're not).

I would recommend starting with the following substitutions and then adjusting colors up or down a shade on a case-by-case basis as you feel is needed.

For grays (note that `grey` has changed to `gray`):

| Old | New |
| --- | --- |
| black | gray-900 |
| grey-darkest | gray-800 |
| grey-darker | gray-700 |
| grey-dark | gray-600 |
| grey | gray-500 |
| grey-light | gray-400 |
| grey-lighter | gray-200 |
| grey-lightest | gray-100 |
| white | white |

For other colors:

| Old | New |
| --- | --- |
| {color}-darkest | {color}-900 |
| {color}-darker | {color}-800 |
| {color}-dark | {color}-600 |
| {color} | {color}-500 |
| {color}-light | {color}-400 |
| {color}-lighter | {color}-200 |
| {color}-lightest | {color}-100 |

Again, this change only affects you if you do not have your own color palette specified in your config file, or you are using the default Tailwind build through a CDN. If you are using the v0.x color palette in your project, you can absolutely keep using it. You do not need to make these changes unless you have a hard dependency on our default color palette in some way.

<h3 class="no-toc">2. Replace <code>tracking-tight/wide</code> with `tracking-tighter/wider`</h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: Low, Effort: Low</p>

**This change only affects you if you don't have a tracking/letter-spacing scale defined in your config file or you are using Tailwind through a CDN.**

In v1.0, the default letter-spacing scale has changed:

```diff
 letterSpacing: {
+  tighter: -.05em,
-  tight: -.05em,
+  tight: -.025em,
   normal: 0,
-  wide: .05em,
+  wide: .025em,
+  wider: .05em,
+  widest: .1em,
 }
```

That means that if you want your project to look the same, you'll want to replace any existing occurrences of `tracking-tight` with `tracking-tighter`, and `tracking-wide` with `tracking-wider`.

Again, this only applies if you do not have a letter-spacing scale defined in your config file or if you are using the default Tailwind build through a CDN.

If you started with a complete config file, your old scale will continue to work the same way in v1.0 and you don't need to make any changes.

<h3 class="no-toc">3. Check your design against the updated default breakpoints</h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: Low, Effort: Low</p>

**This change only affects you if you don't have `screens` defined in your config file or you are using Tailwind through a CDN.**

The default breakpoints have changed a bit in v1.0:

| Screen | Old | New |
| --- | --- | --- |
| sm | 576px | 640px |
| md | 768px | 768px *(unchanged)* |
| lg | 992px | 1024px |
| xl | 1200px | 1280px |

If your config file doesn't have any screens defined or you are using the default Tailwind build through a CDN, you'll want to audit your design and make sure that nothing breaks because of these changes. No breakpoints got smaller so you are very unlikely to run into any issues, but it's worth checking either way.

Again, this only applies if you do not have any screens defined in your config file or if you are using the default Tailwind build through a CDN.

If you started with a complete config file, your old screens values will continue to work the same way in v1.0 and you don't need to make any changes.

<h3 class="no-toc">4. Double check any usage of the default <code>shadow-{size}</code> utilities</h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: Low, Effort: Low</p>

**This change only affects you if you don't have a box-shadow scale defined in your config file or you are using Tailwind through a CDN.**

Tailwind v1.0 introduces two new box-shadow sizes (`xl`, and `2xl`) and the rest of the shadows have been adjusted as well ([\#691](https://github.com/tailwindcss/tailwindcss/pull/691)).

If your config file doesn't have a box-shadow scale defined or you are using the default Tailwind build through a CDN, you should double check that you are still happy with how your shadows look. You may want to replace some instances of `lg` with `xl` or `2xl`, as the new `lg` shadow is a bit tighter than the old one.

Again, this only applies if you do not have a box-shadow defined in your config file or if you are using the default Tailwind build through a CDN.

If you started with a complete config file, your old box-shadow values will continue to work the same way in v1.0 and you don't need to make any changes.


<h3 class="no-toc">5. Update any usage of the default <code>max-w-{size}</code> utilities</h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: Low, Effort: Low</p>

**This change only affects you if you don't have a max-width scale defined in your config file or you are using Tailwind through a CDN.**

Tailwind v1.0 introduces an all-new max-width scale that is much more usable than the previous default max-width scale ([\#701](https://github.com/tailwindcss/tailwindcss/pull/701)).

If your config file doesn't have a max-width scale defined or you are using the default Tailwind build through a CDN, you should audit your project for any usage of the existing `max-w-{size}` utilities and change the sizes as needed. In general, the new values are smaller than the old ones, so `max-w-md` for example may need to be `max-w-xl` or `max-w-2xl` in the new scale

Again, this only applies if you do not have a max-width defined in your config file or if you are using the default Tailwind build through a CDN.

If you started with a complete config file, your old max-width values will continue to work the same way in v1.0 and you don't need to make any changes.

<hr class="my-16">

## Additional steps for plugin authors

This step only affects users who have authored their own plugins.

<h3 class="no-toc mt-6">Escape the class portion of any custom variants you have created</h3>

<p class="italic font-normal text-gray-600 mt-4">Impact: Low, Effort: Low</p>

In v1.0, you are required to manually escape the class name portion of any selectors you create when adding a new variant using a plugin.

For example:

```diff
- function({ addVariant }) {
+ function({ addVariant, e }) {
    addVariant('first-child', ({ modifySelectors, separator }) => {
      modifySelectors(({ className }) => {
-       return `.first-child${separator}${className}:first-child`
+       return `.${e(`first-child${separator}${className}`)}:first-child`
      })
    })
  },
```

This is just like what you need to do when adding utilities or components that may include user-provided strings.

Unfortunately there is no super simple way to support both v0.x and v1.0 at the same time without checking which version of Tailwind the user has installed and conditionally applying the escape function.
