# lrud

[![Travis build status](http://img.shields.io/travis/jamesplease/lrud.svg?style=flat)](https://travis-ci.com/github/jamesplease/lrud)
[![npm version](https://img.shields.io/npm/v/@please/lrud.svg)](https://www.npmjs.com/package/@please/lrud)

A React library for managing focus in LRUD applications.

## Motivation

The native focus system that ships with browsers is one-dimensional: users may only move forward and back. Some applications
(typically those that use remote controls or video game controllers) require a two dimensional focus system.

Because of this, it is up to the application to manage its own focus state. That's where this library comes in: it makes working
with two dimensional focus seamless.

## Installation

Install using [npm](https://www.npmjs.com):

```
npm install @please/lrud
```

or [yarn](https://yarnpkg.com/):

```
yarn add @please/lrud
```

This library has the following peer dependencies:

- [`react@^16.8.0`](https://www.npmjs.com/package/react)

## Table of Contents

- [**Guides**](#guides)
  - [Getting started](#getting-started)
  - [FAQ](#faq)
- [**API Reference**](#api-reference)
  - [\<FocusRoot/\>](#focusroot-)
  - [\<FocusNode/\>](#FocusNode-)
  - [useFocusNode()](#usefocusnode-focusid-)
  - [useSetFocus()](#usesetfocus)
  - [useNodeEvents()](#usenodeevents-focusid-events-)
  - [useFocusHierarchy()](#usefocushierarchy)
  - [useFocusStoreDangerously()](#usefocusstoredangerously)
- [**Interfaces**](#interfaces)
  - [FocusNode](#focusnode)
  - [LRUDEvent](#lrudevent)
  - [FocusStore](#focusstore)
  - [FocusState](#focusstate)
- [**Prior Art**](#prior-art)
- [**Limitations**](#limitations)

## Guides

### Getting Started

Render the `FocusRoot` high up in your application's component tree.

```jsx
import { FocusRoot } from '@please/lrud';

export default function App() {
  return (
    <FocusRoot>
      <AppContents />
    </FocusRoot>
  );
}
```

You may then use FocusNode components to create a focusable elements on the page.

```jsx
import { FocusNode } from '@please/lrud';

export default function Profile() {
  return <FocusNode className="profile">Profile</FocusNode>;
}
```

This library automatically moves the focus between the FocusNodes as the user inputs
LRUD commands on their keyboard or remote control.

This behavior can be configured through the props of the FocusNode component. To
learn more about those props, refer to the API documentation below.

### FAQ

#### What is LRUD?

LRUD is an acronym that stands for left-right-up-down, and it refers to the directional buttons typically found on remotes. In LRUD systems,
input devices usually also have some kind of "submit" button, and, less commonly, a back button.

#### Is this library right for me?

The [limitations](#limitations) described below may help you to determine that.

## API Reference

This section of the documentation describes the library's named exports.

### `<FocusRoot />`

Serves as the root node of a new focus hierarchy. There should only ever be one `FocusRoot` in each application.

All props are optional.

| Prop          | Type    | Default value  | Description                                                                                             |
| ------------- | ------- | -------------- | ------------------------------------------------------------------------------------------------------- |
| `orientation` | string  | `'horizontal'` | Whether the children of the root node are arranged horizontally or vertically.                          |
| `wrapping`    | boolean | `false`        | Set to `true` for the navigation to wrap when the user reaches the start or end of the root's children. |

```jsx
import { FocusRoot } from '@please/lrud';

export default function App() {
  return (
    <FocusRoot orientation="vertical">
      <AppContents />
    </FocusRoot>
  );
}
```

### `<FocusNode />`

A [Component](https://reactjs.org/docs/react-component.html) that represents a focusable node in your application.

All props are optional. Example usage appears beneath the props table.

| Prop                        | Type                | Default value    | Description                                                                                                                                            |
| --------------------------- | ------------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `propsFromNode`             | function            |                  | A function you can supply to compute additional props to apply to the element. The function is passed one argument, the [focus node](#focusnode).      |
| `className`                 | string              |                  | A class name to apply to this element.                                                                                                                 |
| `focusedClass`              | string              | "isFocused"      | A class name that is applied when this element is focused.                                                                                             |
| `focusedLeafClass`          | string              | "isFocusedExact" | A class name that is applied this element is exactly focused.                                                                                          |
| `activeClass`               | string              | "isActive"       | A class name that is applied this element is active.                                                                                                   |
| `disabledClass`             | string              | "focusDisabled"  | A class name that is applied this element is disabled.                                                                                                 |
| `elementType`               | string\|elementType | 'div'            | The React element type to render. For instance, `"img"` or [`motion.div`](https://www.framer.com/api/motion/component/).                               |
| `focusId`                   | string              | `{unique_id}`    | A unique identifier for this node. Specify this yourself for debugging purposes, or when you will need to manually set focus to the node.              |
| `orientation`               | string              | 'horizontal'     | Whether the children of this node are arranged horizontally or vertically. Pass `"vertical"` for vertical lists.                                       |
| `wrapping`                  | boolean             | 'false'          | Set to `true` for the navigation to wrap when the user reaches the start or end of the children list. For grids this sets wrapping in both directions. |
| `wrapGridColumns`           | boolean             | 'false'          | Set to `true` for horizontal navigation in grids to wrap.                                                                                              |
| `wrapGridRows`              | boolean             | 'false'          | Set to `true` for vertical navigation in grids to wrap.                                                                                                |
| `disabled`                  | boolean             | 'false'          | This node will not receive focus when `true`.                                                                                                          |
| `isGrid`                    | boolean             | 'false'          | Pass `true` to make this a grid.                                                                                                                       |
| `isTrap`                    | boolean             | 'false'          | Pass `true` to make this a focus trap.                                                                                                                 |
| `canReceiveFocusFromArrows` | boolean             | 'true'           | Pass `false` and this node will not receive focus from arrows. Commonly paired with `isTrap`.                                                          |
| `restoreTrapFocusHierarchy` | boolean             | 'true'           | Pass `false` and, if this node is a trap, it will not restore their previous focus hierarchy when becoming focused again.                              |
| `onMountAssignFocusTo`      | string              |                  | A focus ID of a nested child to default focus to when this node mounts.                                                                                |
| `isExiting`                 | boolean             |                  | Pass `true` to signal that this node is animating out. Useful for certain kinds of exit transitions.                                                   |
| `onFocused`                 | function            |                  | A function that is called when the node receives focus. Passed one argument, an [LRUDEvent](#lrudevent).                                               |
| `onBlurred`                 | function            |                  | A function that is called when the node loses focus. Passed one argument, an [LRUDEvent](#lrudevent).                                                  |
| `onKey`                     | function            |                  | A function that is called when the user presses any TV remote key while this element has focus. Passed one argument, an [LRUDEvent](#lrudevent).       |
| `onArrow`                   | function            |                  | A function that is called when the user presses a directional button. Passed one argument, an [LRUDEvent](#lrudevent).                                 |
| `onLeft`                    | function            |                  | A function that is called when the user presses the left button. Passed one argument, an [LRUDEvent](#lrudevent).                                      |
| `onUp`                      | function            |                  | A function that is called when the user presses the up button. Passed one argument, an [LRUDEvent](#lrudevent).                                        |
| `onDown`                    | function            |                  | A function that is called when the user presses the down button. Passed one argument, an [LRUDEvent](#lrudevent).                                      |
| `onRight`                   | function            |                  | A function that is called when the user presses the right button. Passed one argument, an [LRUDEvent](#lrudevent).                                     |
| `onSelected`                | function            |                  | A function that is called when the user pressed the select button. Passed one argument, an [LRUDEvent](#lrudevent).                                    |
| `onBack`                    | function            |                  | A function that is called when the user presses the back button. Passed one argument, an [LRUDEvent](#lrudevent).                                      |
| `onMove`                    | function            |                  | A function that is called when the focused child index of this node changes. Only called for nodes with children that are _not_ grids.                 |
| `onGridMove`                | function            |                  | A function that is called when the focused child index of this node changes. Only called for grids.                                                    |
| `children`                  | React Node(s)       |                  | Children of the Focus Node.                                                                                                                            |
| `...rest`                   | any                 |                  | All other props are applied to the underlying DOM node.                                                                                                |

```jsx
import { FocusNode } from '@please/lrud';

export default function Profile() {
  return (
    <FocusNode
      elementType="button"
      className="profileBtn"
      onSelected={({ node }) => {
        console.log('The user just selected this profile', node);
      }}>
      Profile
    </FocusNode>
  );
}
```

### `useFocusNode( focusId )`

A [Hook](https://reactjs.org/docs/hooks-intro.html) that returns the focus node with ID `focusId`. If the node does not exist,
then `null` will be returned instead.

```js
import { useFocusNode } from '@please/lrud';

export default function MyComponent() {
  const navFocusNode = useFocusNode('nav');

  console.log('Is the nav focused?', navFocusNode?.isFocused);
}
```

### `useSetFocus()`

A [Hook](https://reactjs.org/docs/hooks-intro.html) that returns the `setFocus` function, which allows you to imperatively set
the focus.

This can be used to:

- override the default navigation behavior of the library
- focus modals or traps
- exit traps

```js
import { useSetFocus } from '@please/lrud';

export default function MyComponent() {
  const setFocus = useSetFocus();

  useEffect(() => {
    setFocus('nav');
  }, []);
}
```

### `useNodeEvents( focusId, events )`

A [Hook](https://reactjs.org/docs/hooks-intro.html) that allows you to tap into a focus nodes' focus lifecycle events. Use this hook when
you need to respond to the focus lifecycle for a node that is not in your current component.

```js
import { useNodeEvents } from '@please/lrud';

export default function MyComponent() {
  useNodeEvents('nav', {
    focus(navNode) {
      console.log('The nav node is focused', navNode);
    }

    blur(navNode) {
      console.log('The nav node is no longer focused', navNode);
    }
  });
}
```

Each callback receives a single argument, the [focus node](#focusnode).

The available event keys are:

| Event key  | Called when                         |
| ---------- | ----------------------------------- |
| `focus`    | the focus node receives focus.      |
| `blur`     | the focus node loses focus.         |
| `active`   | the focus node becomes active.      |
| `inactive` | the focus node is no longer active. |
| `disabled` | the focus node is set as disabled.  |
| `enabled`  | the focus node is enabled.          |

### `useFocusHierarchy()`

A [Hook](https://reactjs.org/docs/hooks-intro.html) that returns an array representing the focus hierarchy, which are the nodes
that are currently focused. Each entry in the array is a focus node.

```js
import { useFocusHierarchy } from '@please/lrud';

export default function MyComponent() {
  const focusHierarchy = useFocusHierarchy();

  console.log(focusHierarchy);
  // => [
  //   { focusId: 'root', ... },
  //   { focusId: 'homePage', ... },
  //   { focusId: 'mainNav', ... },
  // ]
}
```

### `useFocusStoreDangerously()`

> ⚠️ Heads up! The FocusStore is an internal API. We strongly discourage you from accessing properties or calling
> methods on the FocusStore directly!

A [Hook](https://reactjs.org/docs/hooks-intro.html) that returns the
[FocusStore](#focusstore). Typically, you should not need to use this hook.

```js
import { useFocusStoreDangerously } from '@please/lrud';

export default function MyComponent() {
  const focusStore = useFocusStoreDangerously();

  useEffect(() => {
    console.log('the current focus state:', focusStore.getState());
  }, []);
}
```

## Interfaces

These are the objects you will encounter when using this library.

### `FocusNode`

A focus node. Each `<FocusNode/>` React component creates one of these.

| Property                      | Type             | Description                                                                                                                       |
| ----------------------------- | ---------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `focusId`                     | string           | A unique identifier for this node.                                                                                                |
| `children`                    | Array\<string\>  | An array of focus IDs representing the children of this node.                                                                     |
| `focusedChildIndex`           | number\|null     | The index of the focused child of this node, if there is one.                                                                     |
| `prevFocusedChildIndex`       | number\|null     | The index of the previously-focused child of this node, if there was one.                                                         |
| `isFocused`                   | boolean          | `true` when this node is focused.                                                                                                 |
| `isFocusedLeaf`               | boolean          | Whether or not this node is the leaf of the focus hierarchy.                                                                      |
| `active`                      | boolean          | `true` this node is active.                                                                                                       |
| `disabled`                    | boolean          | `true` when this node is disabled.                                                                                                |
| `isExiting`                   | boolean          | Set to `true` to indicate that the node will be animating out. Useful for certain exit animations.                                |
| `wrapping`                    | boolean          | `true` when the navigation at the end of the node will wrap around to the other side.                                             |
| `wrapGridRows`                | boolean          | `true` when grid rows will wrap.                                                                                                  |
| `wrapGridColumns`             | boolean          | `true` when grid columns will wrap.                                                                                               |
| `isRoot`                      | boolean          | `true` this is the root node.                                                                                                     |
| `trap`                        | boolean          | `true` when this node is a focus trap.                                                                                            |
| `restoreTrapFocusHierarchy`   | boolean          | Set to `true` and a focus trap will restore its previous hierarchy upon becoming re-focused.                                      |
| `parentId`                    | string \| `null` | The focus ID of the parent node. `null` for the root node.                                                                        |
| `orientation`                 | string           | A string representing the orientation of the node (either `"horizontal"` or `"vertical"`)                                         |
| `navigationStyle`             | string           | One of `'first-child'` or `'grid'`                                                                                                |
| `nodeNavigationItem`          | string           | How this node is used in the navigation algorithm. Possible values are 'default'`, 'grid-container'`, `'grid-row'`, `'grid-item'` |
| `canReceiveFocusFromArrows`   | boolean          | Set to `false` and this node will not receive focus due to arrow navigation.                                                      |
| `_gridColumnIndex`            | number \| `null` | The focused column index of a grid.                                                                                               |
| `_gridRowIndex`               | number \| `null` | The focused row index of a grid.                                                                                                  |
| `_focusTrapPreviousHierarchy` | Array\<string\>  | The previous focus hierarchy of a trap.                                                                                           |

### `LRUDEvent`

An object that is passed to you in the event callbacks of a `FocusNode`, such as `onFocus`.

| Property          | Type                    | Description                                                                                                            |
| ----------------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `key`             | string                  | A string representing the key that was pressed. One of `"left"`, `"right"`, `"up"`, `"down"`, `"select"`, or `"back"`. |
| `isArrow`         | boolean                 | Whether or not this key is an arrow.                                                                                   |
| `node`            | [FocusNode](#focusnode) | The [`FocusNode`](#focusnode) that received this event.                                                                |
| `preventDefault`  | function                | Call this to stop the default behavior of the event. Commonly used to override the navigation behavior                 |
| `stopPropagation` | function                | Call this to stop the propagation of the event.                                                                        |

### `FocusStore`

> ⚠️ Heads up! This is an internal API. We strongly discourage you from accessing properties or calling
> methods on the FocusStore directly!

An object that represents the store that contains all of the state related to what is in focus.
Typically, you should not need to interact with this object directly, but it is made available to you
for advanced use cases that you may have.

| Property       | Type     | Description                                                            |
| -------------- | -------- | ---------------------------------------------------------------------- |
| `getState`     | function | Returns the current [FocusState](#focusstate)                          |
| `createNodes`  | function | Creates one or more focus nodes in the tree.                           |
| `deleteNode`   | function | Deletes a focus node from the tree.                                    |
| `setFocus`     | function | Imperatively assign focus to a particular focus node.                  |
| `updateNode`   | function | Update an existing node. Used to, for example, set a node as disabled. |
| `handleArrow`  | function | Call this to navigate based on an arrow key press.                     |
| `handleSelect` | function | Call this to cause the focus tree to respond to a "select" key press.  |

### `FocusState`

> ⚠️ Heads up! This is an internal API. We strongly discourage you from accessing this object directly in your application.

An object representing the state of the focus in the app.

| Property                 | Type             | Description                                                          |
| ------------------------ | ---------------- | -------------------------------------------------------------------- |
| `focusedNodeId`          | string           | The ID of the leaf node in the focus hierarchy.                      |
| `focusHierarchy`         | Array            | An array of node IDs representing the focus hierarchy.               |
| `activeNodeId`           | string \| `null` | The ID of the active node, if there is one.                          |
| `nodes`                  | Object           | A mapping of all of the focus nodes that exist.                      |
| `_updatingFocusIsLocked` | boolean          | A boolean used internally for managing the creation of nested nodes. |

## Limitations

- No support for pointer (mouse) inputs
- Navigation is determined based on a grid-like system, rather than nodes' spatial position on the page

## Prior Art

- ["Pass the Remote" on the Netflix Tech Blog](https://medium.com/netflix-techblog/pass-the-remote-user-input-on-tv-devices-923f6920c9a8)
- [bbc/lrud](https://github.com/bbc/lrud)
- [react-tv](https://github.com/raphamorim/react-tv)
- [@xdproto/focus](https://github.com/jamesplease/focus) _(the predecessor of this library)_
