---
extends: _layouts.documentation
title: "Adding New Utilities"
description: "Extending Tailwind with your own custom utility classes."
titleBorder: true
---

Although Tailwind provides a pretty comprehensive set of utility classes out of the box, you may run into situations where you need to add a few of your own.

Deciding on the best way to extend a framework can be paralyzing, so here are some best practices to help you add your own utilities in the most idiomatic way possible.

---

## Using CSS

One way to add your own utilities to Tailwind is to add them to your CSS.

@component('_partials.tip-good')
Add any custom utilities to the end of your CSS file
@endcomponent

```css
@@tailwind base;
@@tailwind components;
@@tailwind utilities;

.rotate-0 {
  transform: rotate(0deg);
}
.rotate-90 {
  transform: rotate(90deg);
}
.rotate-180 {
  transform: rotate(180deg);
}
.rotate-270 {
  transform: rotate(270deg);
}
```

Since declaration order is important in CSS, always make sure you are adding new utilities to the _end_ of your CSS file to avoid specificity issues.

@component('_partials.tip-bad')
Don't add custom utilities to the beginning of your CSS file
@endcomponent

```css
/* Adding utilities to the beginning of your CSS can cause specificity issues */
.rotate-0 {
  transform: rotate(0deg);
}
.rotate-90 {
  transform: rotate(90deg);
}
.rotate-180 {
  transform: rotate(180deg);
}
.rotate-270 {
  transform: rotate(270deg);
}

@@tailwind base;
@@tailwind components;
@@tailwind utilities;
```

If you're using `postcss-import` or a preprocessor like Less, Sass, or Stylus, consider keeping your utilities in a separate file and importing them:

```css
/* Using postcss-import */
@@import "tailwindcss/base";
@@import "tailwindcss/components";
@@import "tailwindcss/utilities";
@@import "./custom-utilities.css";

/* Using Sass or Less */
@@tailwind base;
@@tailwind components;
@@tailwind utilities;
@@import "./custom-utilities";
```

### Generating responsive variants

If you'd like to create [responsive variants](/docs/responsive-design) of your own utilities based on the breakpoints defined in your `tailwind.config.js` file, wrap your utilities in the `@@responsive` directive:

```css
@@tailwind base;
@@tailwind components;
@@tailwind utilities;

@@responsive {
  .rotate-0 {
    transform: rotate(0deg);
  }
  .rotate-90 {
    transform: rotate(90deg);
  }
  .rotate-180 {
    transform: rotate(180deg);
  }
  .rotate-270 {
    transform: rotate(270deg);
  }
}
```

Tailwind will automatically generate prefixed versions of each custom utility that you can use to conditionally apply those styles at different breakpoints:

```html
<!-- Rotate 180 degrees by default, but remove that rotation on medium screens and up -->
<div class="rotate-180 md:rotate-0"></div>
```

Learn more about responsive utilities in the [responsive design documentation](/docs/responsive-design).

### Generating pseudo-class variants

If you'd like to create [pseudo-class variants](/docs/pseudo-class-variants) of your own utilities, wrap your utilities in the `@@variants` directive:

```css
@@tailwind base;
@@tailwind components;
@@tailwind utilities;

@@variants hover, focus {
  .rotate-0 {
    transform: rotate(0deg);
  }
  .rotate-90 {
    transform: rotate(90deg);
  }
  .rotate-180 {
    transform: rotate(180deg);
  }
  .rotate-270 {
    transform: rotate(270deg);
  }
}
```

Tailwind will automatically generate prefixed versions of each custom utility that you can use to conditionally apply those styles at different states:

```html
<div class="rotate-0 hover:rotate-90"></div>
```

Pseudo-class variants are generated in the same order you list them in the `@@variants` directive, so if you'd like one pseudo-class to take precedence over another, make sure you list that one last:

```css
/* Focus will take precedence over hover */
@@variants hover, focus {
  .rotate-0 {
    transform: rotate(0deg);
  }
  /* ... */
}

/* Hover will take precedence over focus */
@@variants focus, hover {
  .rotate-0 {
    transform: rotate(0deg);
  }
  /* ... */
}
```

If you'd like to generate both responsive _and_ pseudo-class variants of your custom utilities, wrap `@@variants` with `@@responsive`:

```css
@@tailwind base;
@@tailwind components;
@@tailwind utilities;

@@responsive {
  @@variants hover, focus {
    .rotate-0 {
      transform: rotate(0deg);
    }
    .rotate-90 {
      transform: rotate(90deg);
    }
    .rotate-180 {
      transform: rotate(180deg);
    }
    .rotate-270 {
      transform: rotate(270deg);
    }
  }
}
```

Learn more about pseudo-class variants utilities in the [pseudo-class variant documentation](/docs/pseudo-class-variants).

---

## Using a plugin

In addition to adding new utility classes directly in your CSS files, you can also add utilities to Tailwind by writing your own plugin:

```js
// tailwind.config.js
module.exports = {
  plugins: [
    function({ addUtilities }) {
      const newUtilities = {
        '.rotate-0': {
          transform: 'rotate(0deg)',
        },
        '.rotate-90': {
          transform: 'rotate(90deg)',
        },
        '.rotate-180': {
          transform: 'rotate(180deg)',
        },
        '.rotate-270': {
          transform: 'rotate(270deg)',
        },
      }

      addUtilities(newUtilities, ['responsive', 'hover'])
    }
  ]
}

```

This can be a good choice if you want to publish your custom utilities as a library or make them available to be shared across multiple projects.

Learn more in the [utility plugin documentation](/docs/plugins#adding-utilities).
