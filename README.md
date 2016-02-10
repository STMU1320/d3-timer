# d3-timer

This module provides an efficient queue capable of managing thousands of concurrent animations, while guaranteeing consistent, synchronized timing with concurrent or staged animations. Internally, it uses [requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) for fluid animation (if available), switching to [setTimeout](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/setTimeout) for delays longer than 24ms.

## Installing

If you use NPM, `npm install d3-timer`. Otherwise, download the [latest release](https://github.com/d3/d3-timer/releases/latest). The released bundle supports AMD, CommonJS, and vanilla environments. Create a custom build using [Rollup](https://github.com/rollup/rollup) or your preferred bundler. You can also load directly from [d3js.org](https://d3js.org):

```html
<script src="https://d3js.org/d3-timer.v0.3.min.js"></script>
```

In a vanilla environment, a `d3_timer` global is exported. [Try d3-timer in your browser.](https://tonicdev.com/npm/d3-timer)

## API Reference

<a name="timer" href="#timer">#</a> d3.<b>timer</b>(<i>callback</i>[, <i>delay</i>[, <i>time</i>]])

Schedules a new timer, invoking the specified *callback* repeatedly until the timer is [stopped](#timer_stop). An optional numeric *delay* in milliseconds may be specified to invoke the given *callback* after a delay; if *delay* is not specified, it defaults to zero. The delay is relative to the specified *time* in milliseconds; if *time* is not specified, it defaults to [now](#now).

The *callback* is passed the *elapsed* time since the timer became active. For example:

```js
var t = d3.timer(function(elapsed) {
  console.log(elapsed);
  if (elapsed > 200) t.stop();
}, 150);
```

This produces roughly the following console output:

```
3
25
48
65
85
106
125
146
167
189
209
```

(The exact values may vary depending on your JavaScript runtime and what else your computer is doing at the moment.) Note that the first *elapsed* time is 3ms: this is the elapsed time since the timer started, not since the timer was scheduled. Here the timer started 150ms after it was scheduled due to the specified delay.

If [timer](#timer) is called within the callback of another timer, the new timer callback (if eligible as determined by the specified *delay* and *time*) will be invoked immediately at the end of the current frame, rather than waiting until the next frame. Within a frame, timer callbacks are guaranteed to be invoked in the order they were scheduled (regardless of their start time).

<a name="timer_restart" href="#timer_restart">#</a> <i>timer</i>.<b>restart</b>(<i>callback</i>[, <i>delay</i>[, <i>time</i>]])

Restart a timer with the specified *callback* and optional *delay* and *time*. This is equivalent to stopping this timer and creating a new timer with the specified arguments, although this timer retains the original invocation priority.

<a name="timer_stop" href="#timer_stop">#</a> <i>timer</i>.<b>stop</b>()

Stops this timer, preventing subsequent callbacks. This method has no effect if the timer has already stopped.

<a name="timerOnce" href="#timerOnce">#</a> d3.<b>timerOnce</b>(<i>callback</i>[, <i>delay</i>[, <i>time</i>]])

Like [timer](#timer), except the timer automatically [stops](#timer_stop) on its first callback.

<a name="timerFlush" href="#timerFlush">#</a> d3.<b>timerFlush</b>()

Immediately execute (invoke once) any eligible timer callbacks. Note that zero-delay timers are normally first executed after one frame (~17ms). This can cause a brief flicker because the browser renders the page twice: once at the end of the first event loop, then again immediately on the first timer callback. By flushing the timer queue at the end of the first event loop, you can run any zero-delay timers immediately and avoid the flicker.

<a name="now" href="#now">#</a> d3.<b>now</b>()

Returns the current time as defined by [performance.now](https://developer.mozilla.org/en-US/docs/Web/API/Performance/now) if available, and [Date.now](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Date/now) if it is now. The current time is updated at the start of a frame; this means it is consistent during the frame, and any timers scheduled during the same frame will have consistent timing. If the current time is queried outside of a frame, such as in response to a user event, the current time is recalculated but then fixed until the next animation frame, again making the current time consistent during event handling.
