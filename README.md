# seapig 🌊🐷

![Sea Pig](https://media.giphy.com/media/X2C5xcQLdVGda/giphy.gif)

> Utility to restrict the shape of a [React](http://facebook.github.io/react/index.html) component.

## `seapig`. `seapig`. Does whatever a `seapig` does!

Your friendly neighborhood `seapig` is here to help you maintain consistent rendering structure for your React Components. It retrieves designated props from children, ensures all corresponding children are provided, then renders them in a pre-determined order.

Good ol' `seapig` restricts this consistent shape by enforcing [**Rendering Order**](#RenderingOrder) and [**Child Presence**](#ChildPresence).

The main idea is to promote a cleaner rendering structure by requiring components that can (or must) be rendered together to be provided externally as children rather than passing them as props:

```js
/* No more need for this */
<Button label={<span>Click Me</span>} icon={<i className="fa" />} />

/* Now we can have this */
<Button>
  <i className="fa" icon />
  <span>Click Me</span>
</Button>
```

This not only allows a cleaner JSX layout but also decouples child specific properties such as tag name or class name, allowing external users to freely specify them.

### <a name="RenderingOrder">Rendering Order</a>

Children are rendered into special placeholders determined by the provided props. This means that the rendering shape is enforced internally, allowing us to pass the children into a `seapig` component in any order.

```js
/* `MyCoolSidebar` and `Content` are always rendered in the same order no matter what order they are passed in */

/* This one would render the same */
<Main>
  <MyCoolSidebar sidebar />
  <Content content>
    <h2>Hello `seapig`!</h2>
  </Content>
</Main>

/* as this one */
<Main>
  <Content content>
    <h2>Hello `seapig`!</h2>
  </Content>
  <MyCoolSidebar sidebar />
</Main>

/* Corresponding seapig component */
import seapig from 'seapig'

const Main = props => {
  const {
    sidebar,
    content
  } = seapig(props.children, {
    required: ['sidebar', 'content']
  })

  // rendering order is always the same
  return (
    <div>
      <aside>{sidebar}</aside>
      <section>{content}</section>
    </div>
  )
}
```

### <a name="ChildPresence">Child presence</a>

A `seapig` component ensures that all children identified by the required props are passed in.

To reuse `<Main>` from above as an example, if we didn't pass one or more of its children with the required props (`'content'` or `'sidebar'`), `seapig` would throw:

```js
// The code bellow would throw a "Missing required prop `sidebar`" error
<Main>
  <Content content>
    <h2>Hello `seapig`!</h2>
  </Content>
</Main>
```

The `seapig` also ensures that there are no rogue unidentified children, unless we tell it explicitly that we are expecting a single child without an identifying prop.

```js
import seapig, { DEFAULT_CHILD } from 'seapig'

const Main = props => {
  const {
    sidebar,
    content,
    [DEFAULT_CHILD]: thirdChild
  } = seapig(props.children, {
    optional: DEFAULT_CHILD // <-- tell seapig we're expecting an unidentified child
    required: ['sidebar', 'content']
  })

  // rendering order is always the same
  return (
    <div>
      <aside>{sidebar}</aside>
      <section>{content}</section>
      <section>{thirdChild}</section>
    </div>
  )
}

/* Now we don't need to specify an identifying prop on one of the children */
<Main>
  <Content content>
    <h2>Hello `seapig`!</h2>
  </Content>
  <MyCoolSidebar sidebar />
  <IAmTheThirdChildWithoutIdentifyingProp />
</Main>
```

## Why `seapig`?

Sea pig stands for (**C**hildren **P**rops **I**nternal **G**etter), except the `C` is spelled phonetically.

## Install

```bash
npm install seapig --save
```

## Example

### `seapig` button with a required label and an optional icon:

```js
import React, { Component } from 'react'
import seapig from 'seapig'

/* Button with a required label and an optional icon */
class Button extends Component {
  render() {
    const {
      icon,
      label
    } = seapig(this.props.children, {
      optional: ['icon'],
      required: ['label']
    })

    return (
      <button>
        {icon && <span className="pull-left">{icon}</span>}
        {label}
      </button>
    )
  }
}

/* usage of the seapig Button */
import React, { Component } from 'react'
import Button from './Button'

class Form extends Component {
  render() {
    return (
      <div>
        <Button>
          <i className="fa fa-upload" icon />
          <span label>Submit</span>
        </Button>
      </div>
    )
  }
}
```

## API

```js
const {
  icon,
  [DEFAULT_CHILD]: anyChild, // the single allowed child without any specific designate props
  label,
} = seapig(children, {
  // optional children prop names are denoted in the `optional` array
  optional: ['icon', DEFAULT_CHILD],
  // required children prop names are denoted in the `required` array
  required: ['label'] // code will throw if no child with the `label` prop was passed because it's required
})
```

## License

MIT
