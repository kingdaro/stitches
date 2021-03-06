<p align="center">
  <img width="300" src="../../react.png">
</p>

Use stitches to create styled components in React.

Read more about stitches at [@stitches/css](https://github.com/christianalfoni/stitches/tree/master/packages/css).

## Get started

`npm install @stitches/styled`

```tsx
// css.ts
import { createCss } from "@stitches/css";
import { createStyled } from "@stitches/styled";

export const css = createCss({});
export const styled = createStyled(css);
```

```tsx
// App.tsx
import * as React from "react";
import { styled } from "./css";

const Header = styled.h1({
  color: "red",
  margin: "2rem",
});

export const App: React.FC = () => {
  return (
    <div>
      {/*
        You can change the underlying element, which also // changes the typing
        of the component
      */}
      <Header as="h2">Hello World!</Header>
    </div>
  );
};
```

**Note!** The styled API, like the core, also supports functional syntax to reduce a bit of overhead. Wherever the documentation shows object syntax you can use `css => css.compose(...)` instead.

## Variants

Instead of providing props to the styled elements, you rather have variants. This fixes two important issues with styled APIs:

1. No invalid props are passed down to the underlying DOM node
2. The css definition has no dynamic behaviour, meaning there is only a single evaluation. This creates a big performance improvement

```tsx
import { styled } from "./css";

export const Button = styled.button(
  {
    borderRadius: 1,
  },
  {
    variant: {
      primary: {
        backgroundColor: "red",
      },
      muted: {
        backgroundColor: "gray",
        opacity: 0.5,
      },
    },
    size: {
      small: {
        fontSize: 2,
      },
      big: {
        fontSize: 4,
      },
    },
  }
);
```

The dynamic behaviour is defined where you consume the styled component:

```tsx
<Button variant="primary" size="small"></Button>

<Button variant={isDisabled ? 'muted' : 'primary'} size="small" disabled={isDisabled}></Button>
```

## Override

All styled components takes a `styled` property. This property allow you to override styles.

```tsx
const override = styled({
  ":hover": {
    color: "blue",
  },
});

export const MyComponent = () => {
  return (
    <div>
      <Button variant="primary" styled={override}></Button>
    </div>
  );
};
```

**Note!**. All overrides should be defined outside of components. Stitches will actually throw an error if you try to dynamically pass in overrides. This also improves performance.

## Server side rendering

```tsx
import { renderToString } from "react-dom/server";
import { css } from "../css";
import { App } from "../App";

const { styles, result } = css.getStyles(() => renderToString(<App />));
```

**styles** is an array of style contents. Each item in the array should result in a style tag passed to the browser. **result** is just the result of the callback, in this example the string returned from **renderToString**.
