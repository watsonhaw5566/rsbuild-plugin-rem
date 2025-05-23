# @rsbuild/plugin-rem

<p>
  <a href="https://npmjs.com/package/@rsbuild/plugin-rem">
   <img src="https://img.shields.io/npm/v/@rsbuild/plugin-rem?style=flat-square&colorA=564341&colorB=EDED91" alt="npm version" />
  </a>
  <img src="https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square&colorA=564341&colorB=EDED91" alt="license" />
</p>

English | [简体中文](./README.zh-CN.md)

The Rem plugin implements the rem adaptive layout for mobile pages, allowing the font size to be dynamically adjusted according to the screen size, making web pages adaptively displayed on screens of different sizes.

The plugin provides the following capabilities:

- Integrates [postcss-pxtorem](https://npmjs.com/package/postcss-pxtorem) to convert the `px` units in CSS to `rem`.
- Inserts runtime code into the HTML template to set the fontSize of the root element.

## Usage

Install:

```bash
npm add @rsbuild/plugin-rem -D
```

Add plugin to your `rsbuild.config.ts`:

```ts
// rsbuild.config.ts
import { pluginRem } from "@rsbuild/plugin-rem";

export default {
  plugins: [pluginRem()],
};
```

## Options

### Default Options

```js
const defaultOptions = {
  enableRuntime: true,
  rootFontSize: 50,
  screenWidth: 375,
  rootFontSize: 50,
  maxRootFontSize: 64,
  widthQueryKey: "",
  excludeEntries: [],
  supportLandscape: false,
  useRootFontSizeBeyondMax: false,
  pxtorem: {
    rootValue: 50,
    unitPrecision: 5,
    propList: ["*"],
  },
};
```

### Details

### enableRuntime

- **Type:** `boolean`
- **Default:** `true`

Whether to generate runtime code to calculate and set the font size of the root element.

### inlineRuntime

- **Type:** `boolean`
- **Default:** `true`

Whether to inline the runtime code to HTML. If set to `false`, the runtime code will be extracted into a separate `convert-rem.[version].js` file and output to the dist directory.

### rootFontSize

- **Type:** `number`
- **Default:** `50`

The root element font size.

```js
pluginRem({
  rootFontSize: 30,
});
```

### maxRootFontSize

- **Type:** `number`
- **Default:** `64`

The root element max font size.

### widthQueryKey

- **Type:** `string`
- **Default:** `''`

Get clientWidth from the url query based on `widthQueryKey`.

### screenWidth

- **Type:** `number`
- **Default:** `375`

The screen width for UI design drawings (Usually, `fontSize = (clientWidth * rootFontSize) / screenWidth`).

### excludeEntries

- **Type:** `string[]`
- **Default:** `[]`

To exclude some page entries from injecting runtime code, the item is the page entry name. It is usually used with `pxtorem.exclude`.

```ts
// rsbuild.config.ts
import { pluginRem } from "@rsbuild/plugin-rem";

export default {
  source: {
    entry: {
      page1: "./src/page1/index.tsx",
      page2: "./src/page2/index.tsx",
    },
  },
  plugins: [
    pluginRem({
      excludeEntries: ["page2"],
    }),
  ],
};
```

### supportLandscape

- **Type:** `boolean`
- **Default:** `false`

Use height to calculate rem in landscape.

### useRootFontSizeBeyondMax

- **Type:** `boolean`
- **Default:** `false`

Whether to use `rootFontSize` when large than `maxRootFontSize`.

### pxtorem

- **Type:** `object`
- **Default:**
  - `rootValue`: Default is the same as `rootFontSize`
  - `unitPrecision`: `5`
  - `propList`: `['*']`

Customize the [postcss-pxtorem](https://github.com/cuth/postcss-pxtorem#options) options.

```js
pluginRem({
  pxtorem: {
    propList: ["font-size"],
  },
});
```

## Usage Guide

### CSS conversion properties

By default, rootFontSize is 50. So the CSS styles value are converted according to the ratio of `1rem : 50px`.

```css
/* input */
h1 {
  margin: 0 0 16px;
  font-size: 32px;
  line-height: 1.2;
  letter-spacing: 1px;
}

/* output */
h1 {
  margin: 0 0 0.32rem;
  font-size: 0.64rem;
  line-height: 1.2;
  letter-spacing: 0.02rem;
}
```

By default, Rsbuild converts all CSS properties from px to rem. If you want to convert only the `font-size` property, you can setting `pxtorem.propList` is `['font-size']`.

```ts
pluginRem({
  pxtorem: {
    propList: ["font-size"],
  },
});
```

### How to ignore some CSS properties converted to REM?

[pxtorem.propList](https://github.com/cuth/postcss-pxtorem#options) in addition to specifying which attributes need to be converted, you also can specify which elements are not converted through `!`:

```ts
pluginRem({
  pxtorem: {
    propList: ["*", "!border-width"], // not convert 'border-width'
  },
});
```

If you only want some individual CSS properties not to be converted to REM, you can also refer to the following writing method:

```css
/* `px` is converted to `rem` */
.convert {
  font-size: 16px; // converted to 1rem
}

/* `Px` or `PX` is ignored by `postcss-pxtorem` but still accepted by browsers */
.ignore {
  border: 1px solid; // ignored
  border-width: 2px; // ignored
}
```

More info about [postcss-pxtorem](https://github.com/cuth/postcss-pxtorem#a-message-about-ignoring-properties).

### Setting the page rootFontSize

The formula for calculating the font size of the page root element is:

```
pageRootFontSize = clientWidth * rootFontSize / screenWidth
```

In a mobile browser with a screen width of 390, the default value for rootFontSize is 50 and the screenWidth of the UI design is 375.

The calculated font size for the root element of the page is 52 (`390 * 50 / 375`).

At this point, 1 rem is 52px, 32px (0.64 rem) in the CSS style, the actual size in page is 33.28 px.

```ts
pluginRem({
  rootFontSize: 50,
  screenWidth: 375,
});
```

### Customize maxRootFontSize

In the desktop browser, the page rootFontSize obtained from the calculation formula is often too large. When the calculated result large than the maxRootFontSize, the maxRootFontSize will used as the page rootFontSize.

In the desktop browser with a screen width of 1920, the calculated rootFontSize is 349, which exceeds the default maxRootFontSize of 64. 64 is used as the current root element font value.

```ts
pluginRem({
  maxRootFontSize: 64,
});
```

### How to get the rootFontSize value that is actually in effect on the page?

The actual rootFontSize in effect for the page is calculated dynamically based on the current page. It can be seen by printing `document.documentElement.style.fontSize` or obtained by `window.ROOT_FONT_SIZE`.

### How to specify the actual rootFontSize value of the page?

By default, the actual rootFontSize of the page will be dynamically calculated based on the situation of the current page. If you want to specify the actual rootFontSize of the page, you can turn off the `enableRuntime` configuration and set it in [Custom HTML template](/config/html/template) and inject `document.documentElement.style.fontSize = '64px'` by yourself.

```ts
export default {
  html: {
    template: "./static/index.html",
  },
  plugins: [
    pluginRem({
      enableRuntime: false,
    }),
  ],
};
```

### How to determine if REM is in effect？

1. CSS: Check the generated `.css` file to see if the value of the corresponding property is converted from px to rem.
2. HTML: Open the Page Console to see if a valid value exists for `document.documentElement.style.fontSize`.

## License

[MIT](./LICENSE).
