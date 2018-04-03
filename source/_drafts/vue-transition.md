---
title: vue-transition
tags:
---

- automatically apply classes for CSS transitions and animations
- integrate 3rd-party CSS animation libraries, such as Animate.css
- use JavaScript to directly manipulate the DOM during transition hooks
- integrate 3rd-party JavaScript animation libraries, such as Velocity.js


## single elements
- conditional rendering

```js
export default {
  render () {
    const children = this.$slots.default
    if (!children) {
      return
    }

    console.assert(children.length === 1, 'only one children can be in transition')
    const child = children[0]
    
    return child
  }
}
```