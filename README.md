# pending-state-protocol
An open protocol for interoperable asynchronous Web Components.

Document status: Draft

# Background

There are a number of scenarios where a Web Component might depend on some asynchronous work. Components may lazy-load their implementation or content, perform I/O in response to user input, asynchronously render, etc. It's often desirable to communicate the state of async work up the component tree to parent components so that they can display user affordances indicating whether the UI or content is pending some work.

Frameworks have the ability to invent proprietary APIs and patterns to handle this kind of cross-component communication. To enable the same with Web Components in a way that's interoperable across components from from diffrent sources, we need a protocol that components can implement without reliance on a common implementation.

# Goals

* Allow components to communicate that they have pending asynchonous work to ancestors in the DOM tree.
* Allow components to know if there is pending asynchronous work in descendants in the DOM tree.
* Allow components to intercept, modify, and block pending state notifications.
* Give guidance on the type of asynchronous work that should be notified.
* Allow for some differentiation between types of asynchronous work.

# Details

## Asynchronous States

There are four main states that an asynchronous operation can be in:

* Not-started
* Started
* Sucessfully completed
* Failed

## The `pending-state` event

Components with pending work indicate so by firing a composed, bubbling, `pending-state` CustomEvent, with a `promise` property on its `detail` object.

Example:

```js
this.dispatchEvent(new CustomEvent('pending-state', {
  composed: true,
  bubbles: true,
  detail: {promise}
});
```

The Promise must be resolved when the work is complete and rejected if the work fails. The value the Promise resolves to is unspecified.

Using an event with a Promise allows us to represent three of the asynchronous states:

* Not-started: not represented
* Started: `pending-state` fired
* Sucessfully completed: Promise resolved
* Failed: Promise rejected

# Open Questions

## Should the event carry a Promise or should there be two events?

Animations have multiple events: `animationstart` and `animationend`. Promises make it very easy to correlate the start of an operation with the end of that operation.

## Event Name

Is `pending-state` good? Should it be `pending-work`?

## Types of Async Work

There are some types of async work that we may not want to display UI affordances (like progress indicators) for. Async rendering used in order to yield to the browser's task queue for input hanlding and layout/paint for instance, should probably not trigger a progress indicator. Yet, code that measures style or layout may need to wait for rendering to complete.

We could add a field to the event that indicates the type of work and standardize a small number of types, such as `loading` and `rendering`. An event to indicate async rendering starting and stopping has existing analogies with the `animationstart` and `animationend` events.

## Work Estimation

Some use cases, like a progress bar that shows how much work is remaining, could benefit from estimating how much work is pending. The `pending-state` event could carry a numeric work estimate property so that containers can estimate the total amount of pending work and incremental progress.
