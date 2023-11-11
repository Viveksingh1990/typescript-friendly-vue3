(Bonus) Vue vs React: Framework Philosophy
==========================================

In this bonus lesson, we’ll explore the difference in design philosophy between the two popular JavaScript frameworks in regard to their TypeScript experiences.

* * *

Vue vs React
------------

Here’s a visual comparison between Vue and React on implementing the same **Counter** component in TypeScript:

![vue_vs_react.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.1640294209936.jpg?alt=media&token=81e66b76-e689-490b-a2cd-a407489dba36)

If you squint your eyes enough, they might look just about the same. However, there are some subtle but profound differences (motivated by philosophical differences between the two frameworks).

**For example:**

![Screen Shot 2021-08-24 at 1.23.17 PM.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.1640294209937.jpg?alt=media&token=d7ecda98-30ff-4c3c-91a3-5bc9e4b0fbec)

In React, props and default prop values are carried out through native ES6 parameter features. In Vue, these things are “managed” by the framework itself.

This same philosophical difference also applies to _emit_. The React equivalent of that is _callback prop_, which is literally just a _callback_ function passed to a component as a _prop_, which is just a _parameter_.

React is all about relying on as many native language features as possible to keep a small API. This minimalistic approach has always been its strength in providing competitive TypeScript support. And with the improvements of the _Composition API_ and the _Script Setup_ syntax, Vue is able to offer its own version of minimalism through framework-level compilation. This new approach provides the same level of simplicity in TypeScript experience, but with a user-friendly undertone.

