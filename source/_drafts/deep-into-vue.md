---
title: Deep Into vue
tags:
---

## src Directory structure

This is come from the `Project Structure` section of the [Vue contributing document](https://github.com/vuejs/vue/blob/dev/.github/CONTRIBUTING.md).

- `compiler`: contains code for the template-to-render-function compiler.
- `core`: contains universal, platform-agnostic runtime code.
  - `observer`: contains code related to the reactivity system.
  - `vdom`: contains code related to vdom element creation and patching.
  - `instance`: contains Vue instance constructor and prototype methods.
  - `global-api`: as the name suggests.
  - `components`: universal abstract components. Currently `keep-alive` is the only one.
- `server`: contains code related to server-side rendering.
- `platforms`: contains platform-specific code.
  - platforms build entry file
  - `compiler`, `runtime`, `server`: corresponding to the three directories above.
- `sfc`: contains single-file component (`*.vue` files) parsing logic.
- `shared`: contains utilities shared across the entire codebase.
- `types`: contains TypeScript type definitions.

## VueRouter

### Link

```js
  render (h: Function) {
    const router = this.$router
    const current = this.$route
    const { location, route, href } = router.resolve(this.to, current, this.append)

    // compute the link active and exact active class
    const classes = {}
    const globalActiveClass = router.options.linkActiveClass
    const globalExactActiveClass = router.options.linkExactActiveClass
    // Support global empty active class
    const activeClassFallback = globalActiveClass == null
      ? 'router-link-active'
      : globalActiveClass
    const exactActiveClassFallback = globalExactActiveClass == null
      ? 'router-link-exact-active'
      : globalExactActiveClass
    const activeClass = this.activeClass == null
      ? activeClassFallback
      : this.activeClass
    const exactActiveClass = this.exactActiveClass == null
      ? exactActiveClassFallback
      : this.exactActiveClass
    const compareTarget = location.path
      ? createRoute(null, location, null, router)
      : route

    classes[exactActiveClass] = isSameRoute(current, compareTarget)
    classes[activeClass] = this.exact
      ? classes[exactActiveClass]
      : isIncludedRoute(current, compareTarget)

    // click callback
    const handler = e => {
      if (guardEvent(e)) {
        if (this.replace) {
          router.replace(location)
        } else {
          router.push(location)
        }
      }
    }

    // more event callback
    const on = { click: guardEvent }
    if (Array.isArray(this.event)) {
      this.event.forEach(e => { on[e] = handler })
    } else {
      on[this.event] = handler
    }

    const data: any = {
      class: classes
    }

    // attach the event callback to the `a` link
    // if it exist, otherwise event callback attaches to it self.
    if (this.tag === 'a') {
      data.on = on
      data.attrs = { href }
    } else {
      // find the first <a> child and apply listener and href
      const a = findAnchor(this.$slots.default)
      if (a) {
        // in case the <a> is a static node
        a.isStatic = false
        const extend = _Vue.util.extend
        const aData = a.data = extend({}, a.data)
        aData.on = on
        const aAttrs = a.data.attrs = extend({}, a.data.attrs)
        aAttrs.href = href
      } else {
        // doesn't have <a> child, apply listener to self
        data.on = on
      }
    }

    return h(this.tag, data, this.$slots.default)
  }
  ```


### View

```js
  render (_, { props, children, parent, data }) {
    data.routerView = true

    // directly use parent context's createElement() function
    // so that components rendered by router-view can resolve named slots
    // usages:
    // <router-view> <div slot="slotA">AAA</div> </router-view>
    // 
    const h = parent.$createElement
    // name property is used for named views
    // <router-view class="view one"></router-view>
    // <router-view class="view two" name="a"></router-view>
    // <router-view class="view three" name="b"></router-view>
    const name = props.name
    const route = parent.$route
    const cache = parent._routerViewCache || (parent._routerViewCache = {})

    // determine current view depth, also check to see if the tree
    // has been toggled inactive but kept-alive.
    let depth = 0
    let inactive = false
    while (parent && parent._routerRoot !== parent) {
      if (parent.$vnode && parent.$vnode.data.routerView) {
        depth++
      }
      if (parent._inactive) {
        inactive = true
      }
      parent = parent.$parent
    }
    data.routerViewDepth = depth

    // render previous view if the tree is inactive and kept-alive
    if (inactive) {
      return h(cache[name], data, children)
    }

    const matched = route.matched[depth]
    // render empty node if no matched route
    if (!matched) {
      cache[name] = null
      return h()
    }

    const component = cache[name] = matched.components[name]

    // attach instance registration hook
    // this will be called in the instance's injected lifecycle hooks
    data.registerRouteInstance = (vm, val) => {
      // val could be undefined for unregistration
      const current = matched.instances[name]
      if (
        (val && current !== vm) ||
        (!val && current === vm)
      ) {
        matched.instances[name] = val
      }
    }

    // also register instance in prepatch hook
    // in case the same component instance is reused across different routes
    ;(data.hook || (data.hook = {})).prepatch = (_, vnode) => {
      matched.instances[name] = vnode.componentInstance
    }

    // resolve props
    let propsToPass = data.props = resolveProps(route, matched.props && matched.props[name])
    if (propsToPass) {
      // clone to prevent mutation
      propsToPass = data.props = extend({}, propsToPass)
      // pass non-declared props as attrs
      const attrs = data.attrs = data.attrs || {}
      for (const key in propsToPass) {
        if (!component.props || !(key in component.props)) {
          attrs[key] = propsToPass[key]
          delete propsToPass[key]
        }
      }
    }

    return h(component, data, children)
  }


  function resolveProps (route, config) {
    switch (typeof config) {
      case 'undefined':
        return
      case 'object':
        return config
      case 'function':
        return config(route)
      case 'boolean':
        return config ? route.params : undefined
      default:
        if (process.env.NODE_ENV !== 'production') {
          warn(
            false,
            `props in "${route.path}" is a ${typeof config}, ` +
            `expecting an object, function or boolean.`
          )
        }
    }
  }
```

## Vuex
### Store

- `dispatch & commit` always bound to store
- `function resetStoreVM (store, state, hot)`: responsible for the reactivity, also registers getters as computed properties
- `module`
- `subscribe`: register mutation callbacks (`store._subscribers`), `subscribeAction`: register action callbacks (`store._actionSubscribers`).
