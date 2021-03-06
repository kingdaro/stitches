<p align="center">
  <img width="300" src="../../stitches.png">
</p>

## Why

- **Atomic mindset**: Each CSS property is a an atomic part of your complete CSS
- **Reusability**: Each CSS property, given the same screen, pseudo and value is considered the same, giving high degree of reusability
- **Optimal injection**: You can compose your styles outside of your UI, but no injection happens until it is actually used
- **Tokenized**: Configure with tokens to give design restrictions
- **Screens**: Define a set of media queries as screens to easily express CSS active within a screen
- **Utils**: Create your own CSS properties
- **Typed**: Fully typed API, even though you are not using Typescript
- **Specificity solved**: No more specificity issues as an atomic mindset opens up a more efficient and straight forward way to solve it
- **Token based theming**: Tokens are CSS variables. Create themes overriding the tokens and expose themes to any parts of your app

## Get started

`npm install @stitches/css`

```ts
import { css } from "@stitches/css";

const button = css({
  color: "gray",
  ":hover": {
    color: "black",
  },
  borderColor: "black",
  padding: "1rem",
});

const alertButton = css.compose(
  button,
  css({
    borderColor: "red",
  })
);

const dynamicButton = (disabled = false) =>
  css.compose(
    button,
    disabled &&
      css({
        opacity: 0.5,
      })
  );
```

## Functional syntax

The functional style arguably gives a better typing and autocomplete experience in the editor. It is also lower level, removing a tiny bit of overhead that all CSS-IN-JS libraries have traversing objects.

```ts
import { css } from "@stitches/css";

const button = css.compose(
  css.color("gray"),
  css.color("black", ":hover"),
  css.borderColor("black"),
  css.padding("1rem")
);

const alertButton = css.compose(button, css.borderColor("red"));

const dynamicButton = (disabled = false) =>
  css.compose(button, disabled && css.opacity(0.5));
```

## Configure an instance

```ts
import { createCss } from "@stitches/css";

export const css = createCss({
  // Optinally add a prefix to all classnames to avoid crashes
  prefix: "my-lib",
  // Maps tokens to properties. Follows the system-ui theme specification: https://system-ui.com/theme
  tokens: {
    colors: {
      RED: "tomato",
    },
    space: {
      0: "1rem",
    },
    fontSizes: {},
    fonts: {},
    fontWeights: {},
    lineHeights: {},
    letterSpacings: {},
    sizes: {},
    borderWidths: {},
    borderStyles: {},
    radii: {},
    shadows: {},
    zIndices: {},
    transitions: {},
  },
  // Create screens with media queries. Note that the media queriy with the
  // highest specificity should go last
  screens: {
    tablet: (rule) => `@media (min-width: 700px) { ${rule} }`,
  },
  // Create your own custom CSS properties. Here the functional syntax
  // shines to handle pseudo selectors
  utils: {
    marginX: (utilCss) => (value: number | string, pseudo?: string) =>
      utilCss.compose(
        utilCss.marginLeft(value, pseudo),
        utilCss.marginRight(value, pseudo)
      ),
  },
});

css({
  color: "RED", // Creates "tomato"
  tablet: {
    color: "blue", // Color is "blue" when media query is active
  },
  marginX: 0, // Creates "1rem", as it composes margin, using "space" from tokens
});
```

## Themes

You can create theme instances which overrides tokens:

```ts
import { createCss } from "@stitches/css";

export const css = createCss({
  tokens: {
    colors: {
      primary: "tomato",
    },
  },
});

export const funnyTheme = css.theme({
  colors: {
    primary: "pink",
  },
});
```

This theme represents a classname which can be added at any point in your DOM tree. You can add multiple themes, which overrides each other by the nested level you apply them.

## Server side rendering

The `createCss` factory automatically detects if you are in a browser environment. That means when you this factory on the server it will rather collect the styling, which you can retrieve with:

```ts
import { createCss } from "@stitches/css";

const css = createCss({});
const { result, styles } = css.getStyles(() => renderSomething(css));
```

Note that server produced CSS does not contain vendor prefixes, as there is no browser environment to look at. If you have a server rendered application you can either manually add the vendor prefixes you need:

```ts
css({
  WebkitFontSmoothing: "antialiased",
  MozOsxFontSmoothing: "grayscale",
});
```

Or you can use a [postcss](https://www.npmjs.com/package/postcss) to do the conversion:

```ts
import { createCss } from "@stitches/css";
import postcss from "postcss";
import autoprefixer from "autoprefixer";

const css = createCss({});
const { result, styles } = css.getStyles(() => renderSomething(css));

Promise.all(
  styles.map((style) =>
    postcss([autoprefixer({ browsers: ["> 1%", "last 2 versions"] })]).process(
      style
    )
  )
).then((styles) => {
  // styles with vendor prefixes
});
```
