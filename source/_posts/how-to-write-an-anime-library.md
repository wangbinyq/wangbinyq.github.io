---
title: how to write an anime library
date: 2018-04-17 13:42:11
tags:
---
## The engine
We need a mechanism to deal with [layout thrashing](https://github.com/wangbinyq/writings/wiki/Html-Reflow-and-Layout-Thrashing). So we have an engine that group all animations into to a `requestAnimationFrame` callback.

```js
const engine = (function () {
  let animations = {}
  let raf = 0
  
  function play () { raf = requestAnimationFrame(step) }
  function step () { animations.forEach((anim) => anim.step()); if (animations.length) play() }
  function add (anim) { animations.push(anim) }
  function remove (anim) { const index = animations.indexOf(index); animations.splice(index, 1) }

  return {
    play, step, add, remove, raf
  } 
})()
```

## The tween and animation
A tween object repersent a property change. It is an object like this:
```
const defaultTweenSetting = {
  target: Element,
  property: string,
  duration: number,
  easing: Function,
  from: number,
  to: number,
  unit: string
}
```
This object can describe what to be change (`target`, `property`), how long it will last (`duration`), how does the chnage looks like (`easing`), and the start and end value (`from`, `to` and `unit`).

An animation object group list of tweens and play the real animation by add it self to the engine and step the tween.

```js
class Animation {
  constructor (tweens) {
    this.tweens
  }

  step (now) {
    let elapsed = 0
    if (!this.startTime) {
       this.startTime = now
    }
    const elapsed = now - this.startTime
    finisheds = this.tweens.map((tween) => {
      const process = Math.min(1, elapsed / tween.duration)
      const eased = tween.easing(process)
      const value = tween.from + (tween.to - tween.from) * eased
      tween.target.style[tween.property] = value + tween.unit
      if (process >= 1) {
        return true
      }
    })
    if (finisheds.every((f) => f)) {
       this.pause()
    }
  }

  play () {
    engine.add(this)
  }
  pause () {
    engine.remove(this)
  }
  restart () {
    this.pause()
    this.startTime = 0
    this.play()
  }
}
```

## The Libraries
Now we have `engine`, `tween` and `Animtaion`. Use those object or class you can build some style animatons you what to. But it's not enough, we need construct the `tweens` ourself and we only support css style property animation.
There are some realy awesome animation libraries exist:
- [anime.js](https://github.com/juliangarnier/anime): This is what inspire our little animation `engine` with   more features like `transform, svg and object property` (we can do it by add a `type` field to our `tween` and add the corresponding changes code to  `Animation.step` function), `timeline`, `can automaticlly transform params to tweens` etc.
- [velocityjs](http://velocityjs.org): It's incredibly fast, and it features color animation, transforms, loops, easings, SVG support, and scrolling.
- [react-spring](https://github.com/drcmda/react-spring): data driven animation library.
- [Web Animations API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API): The W3C

## Others
### Unit transfrom
1. [jquery.adjustCSS](https://github.com/jquery/jquery/blob/master/src/css/adjustCSS.js): calculate the scale between current unit and the excepted unit, then apply the new united value.
2. velocityjs: 
```js
if (fromUnit !== toUnit) {
  currentValue = `calc(${from * (1 - eased) + fromUnit} + ${to * eased + toUnit})`
}
```

### FLIP Technique
For list reordering stuff.
https://aerotwist.com/blog/flip-your-animations/#the-general-approach
> - Calculate the *First* position.
> - Calculate the *Last* position.
> - *Invert* the positions
> - *Play* the animation