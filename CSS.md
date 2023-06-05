---
title: CSS
lastmod: 2023-06-05T14:38:49-05:00
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