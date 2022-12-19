---
sidebar_position: 3
---

# Preact

[Preact](https://preactjs.com/) is an alternative to React with the same modern
API.

Preact components are packaged using [Webpacker](webpacker.md) and the
Preact code is located in `app/javascript`.

Preact components get loaded via webpacker's helper function
`javascript_packs_with_chunks_tag`.

## How we use Preact

Our approach to Preact follows the [Islands Architecture](https://jasonformat.com/islands-architecture/) pattern: we render as much as we can server-side, and then use Preact to enhance the parts of that server-rendered page that require richer interactivity. Preact is used to either render new content into a placeholder container in the HTML, or (preferably) to replace a server-rendered piece of UI with one that offers a richer experience.

### Example: Preact `<ColorPicker />`

We have a Preact `<ColorPicker />` component which offers both a popover color picker, and a plain text input for a color. When we use this component, we first render a plain text input server-side. This text input is fully functional, and if no JavaScript is loaded, a user can still complete the hex code of their choice. When the page loads, we asynchronously find these plain text inputs and dynamically replace them with the richer Preact experience.

## PropTypes

Preact supports
[PropTypes](https://reactjs.org/docs/typechecking-with-proptypes.html). When
creating Preact components, please ensure that you have defined your PropTypes.

### Common PropTypes

Using PropTypes can be repetitive. Some duplication is normal, like when a
PropType is a string or a number. But for commonly-used PropTypes, like the user
entity, you can use the provided common PropTypes, located in the
`/app/javascript/common-prop-types`, as shown below.

```javascript
import PropTypes from "prop-types";

export const userPropTypes = PropTypes.shape({
  id: PropTypes.string.isRequired,
  name: PropTypes.string.isRequired,
  profile_image_url: PropTypes.string.isRequired,
  summary: PropTypes.string.isRequired,
});
```

#### Using Common PropTypes

Common PropTypes are imported just like any other
[JavaScript Module](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules).
For example, here are two scenarios where a component needs to use the
`tagPropTypes`.

In the example below, our component `SomeComponentUsingTags` has a `tags` prop,
which is an array of the tag entity. PropTypes have a built-in method called
`arrayOf` that allows you to define a prop as an array of something. In our
case, this is the tag entity, so we can use the `tagPropTypes` PropType.

```jsx
import { h } from "preact";
import PropTypes from "prop-types";
import { tagPropTypes } from "../../../components/common-prop-types";

const SomeComponentUsingTags = ({ tags = [] }) => (
  <ul>
    {tags.map((tag) => (
      <li key={tag.id}>{tag.name}</li>
    ))}
  </ul>
);

SomeComponentUsingTags.displayName = "SomeComponentUsingTags";
SomeComponentUsingTags.propTypes = {
  tags: PropTypes.arrayOf(tagPropTypes).isRequired,
};
```

In the following example, the `SomeComponentUsingOneTag` component has a `tag`
prop representing a single tag. In this case, we can just the `tagPropTypes` on
their own to represent the shape of the `tag` prop.

```jsx
import { h } from "preact";
import { tagPropTypes } from "../../../components/common-prop-types";

const SomeComponentUsingOneTag = ({ tag }) => <li key={tag.id}>{tag.name}</li>;

SomeComponentUsingOneTag.displayName = "SomeComponentUsingTags";
SomeComponentUsingOneTag.propTypes = {
  tag: tagPropTypes.isRequired,
};
```

## Managing State

One thing to note about our usage of Preact is that there are no state management libraries in play, only [component state](https://reactjs.org/docs/faq-state.html). The lack of a state management library is intentional and enforced by the nature of Islands architecture. There is no main root component where global state management can live.

Although there is no state management library in play, data attributes on DOM elements sometimes simulate it, e.g. user data.

For a more complex component state, use the [useReducer hook](https://reactjs.org/docs/hooks-reference.html#usereducer). The [multiselect autocomplete component](https://storybook.forem.com/?path=/story/app-components-multiselectautocomplete--default) [(source code)](https://github.com/forem/forem/blob/0024fe40d6ade998a216216b00f157fa7f49e1c0/app/javascript/crayons/MultiSelectAutocomplete/MultiSelectAutocomplete.jsx) is a great example of this.

 It's a judgment call to use useReducer. If you notice you're juggling many useState hooks, it's time to consider useReducer.

## Cummulative Layout Shift (CLS)
Render the client-side component's initial markup server-side to avoid cumulative layout shift (CLS) entirely or as best as possible. Preact's virtual dom (VDOM) usage is smart enough to determine if the component needs to rerender markup, even from the initial server-side rendered markup.

## Benefiting from the React Ecosystem

Preact ships with a thin compatibility layer if you're consuming 3rd party components from the React ecosystem. The react and react-dom packages are [already aliased for you in our webpack configuration for the application](https://github.com/forem/forem/blob/b81d8c947972761a8f3c8f82bb594ffd2f701b25/config/webpack/environment.js#L31-L32[) and the [Webpack configuration for Storybook](https://github.com/forem/forem/blob/b81d8c947972761a8f3c8f82bb594ffd2f701b25/app/javascript/.storybook/main.js#L80-L81).

It also allows you to use [functionality that is not in the core of Preact](https://preactjs.com/guide/v10/whats-new/#compat-lives-in-core), e.g. [forwardRef](https://reactjs.org/docs/forwarding-refs.html), [createPortal](https://reactjs.org/docs/portals.html), [memo](https://reactjs.org/docs/react-api.html#reactmemo) etc. To use those though, you need to [explicitly import them](https://github.com/forem/forem/blob/b81d8c947972761a8f3c8f82bb594ffd2f701b25/app/javascript/crayons/MentionAutocompleteTextArea/MentionAutocompleteTextArea.jsx#L3) from 'preact/compat'

Here are some examples of the compatibility layer in use:

- [MentionAutocompleteTextArea](https://github.com/forem/forem/blob/0024fe40d6ade998a216216b00f157fa7f49e1c0/app/javascript/crayons/MentionAutocompleteTextArea/MentionAutocompleteTextArea.jsx#L3) component
- [Loading the moderation center tooling](https://github.com/forem/forem/blob/0024fe40d6ade998a216216b00f157fa7f49e1c0/app/javascript/modCenter/singleArticle/index.jsx#L3)