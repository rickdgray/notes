---
title: CSS
lastmod: 2023-07-06T13:48:37-05:00
---
# CSS
Some notes on CSS stuff I often forget. [More info](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors) to summarize in the future.
## Selector Operators
| Relation | Operator | Description | Example |
|---|---|---|---|
| Same Element | Concat | Selects all elements with both elements | `div.blue` will match all `<div>` elements that also contain the `blue` class. |
| Grouping | `,` | Selects all elements regardless of relation | `div, span` will match all `<div>` elements and `<span>` elements. |
| Descendant | ` ` (space) | Selects __all__ descendants of the first element. | `div span` will match all `<span>` elements that are inside a `<div>` element. |
| Child | `>` | Selects immediate descendants (children) of the first element. | `ul > li` will match all `<li>` elements that are directly nested in a `<ul>` element. |
| Siblings | `~` | Selects all siblings __after__ the first element. | `p ~ span` will match all `<span>` elements that follow a `<p>` element, but not before. |
| Adjacent Sibling | `+` | Selects the first sibling __after__ the first element. | `h2 + p` will match all `<p>` elements that immediately follow an `<h2>` element, but not before. |
## Sane Defaults
Here's some defaults that are good to start with.
* Fixing box-sizing prevents weird, archaic padding issues.
* The red outline is helpful for debugging your elements.
* Setting margin to 0 is a must-have; browsers have differing default margins on the body element.
* Setting height should be avoided all around. It may be necessary to set `min-height` if you're building a SPA (single page app). Also, `vh` units can't cause problems on mobile browsers, which is fixed by using `dvh`. Unfortunately, it doesn't have full browser support yet, so you can set both so that `vh` acts as a backup.
* Using a flexbox on the body is a bit more intuitive for sizing elements than block style display in my experience.
* Don't use the default font; it's usually Times New Roman which is a serif font. You want a sans-serif for better screen readability. Roboto from [Google Fonts](https://fonts.google.com/) is a common choice. FYI, it is better to self-host your font than just paste a link into your header.
* Setting the font size of all elements to one rem is good for resetting, but each header size will need a new size to be set.
* Line height should always be increased, but how much depends on your font size; this is a good default but adjust on your element when needed.
* Underlines on links are ugly; this is up for debate.
* The nav and main sections are just good starting points for a simple page with a nav menu up top.
```css
:root {
    --background-color: #000000;
    --primary-color: #ffffff;
}

* {
    box-sizing: border-box;
    /* outline to see bounds */
    outline: red solid 1px;
}

body {
    background-color: var(--background-color);
    margin: 0;
    /* min-height: 100vh; */
    /* min-height: 100dvh; */
    display: flex;
    flex-direction: column;
}

h1, h2, h3, h4, h5, h6, p, a, li, ul, ol, span, div {
    font-family: 'Roboto', sans-serif;
    font-weight: 400;
    font-size: 1rem;
    line-height: 1.5;
    color: var(--primary-color);
}

a {
    text-decoration: none;
}

nav {
    display: flex;
    width: 100%;
}

nav > .nav-section {
    display: flex;
    gap: 1rem;
    justify-content: center;
    align-items: center;
}

main {
    flex-grow: 1;
}
```
## Data Attributes
You can get data attributes from the HTML using the `attr()` function. You can also add text to the return of the function. This example will set the content of the after element to the `my-data` data attribute concatenated with `": "`. This function currently only exists on content.
```css
div::after {
	content: attr(data-my-data) ": ";
}
```
## Max Width
Modern CSS has better ways to handle setting max width on a container. Setting `100%` makes the container responsive, subtracting `2rem` adds one rem of margin to both sides, taking the minimum between that and `600px` effectively makes the content not grow any wider than 600 px, and setting `margin-inline` to `auto` centers the content within the container.
```css
.container {
	width: min(100% - 2rem, 600px);
	margin-inline: auto;
}
```
## Fonts
### Font Size
```css
h1 {
	font-size: clamp(4rem, 10vw + 0.5rem, 9rem);
}
```
## SCSS
### Bootstrap
Here is a good default starter for pulling bootstrap into your project. This example uses version `5.3`. Note that by importing the Bootstrap `scss` file into your own, you do not need to import any `bootstrap.css` files in your HTML. The `scss` file is everything; it is not "split" between the two. You should import the `bootstrap.js` file if you intend to use those features, however.
```scss
@import "functions";
@import "variables";
@import "mixins";

$enable-negative-margins: true;

$purple: #000000;
$red: #ff0000;
$primary: $purple;
$danger: $red;
$theme-colors: map-merge($theme-colors, (
    "primary": $primary,
    "danger": $danger
));

@import "bootstrap";
```