Reactive Variables with Type Inference
======================================

In this lesson, we’ll start using TypeScript with ref() and reactive(). Type inference is a main theme when working with reactive variables. We’ll go through some general patterns and caveats on creating reactivie variables with/without type inference.

* * *

Reactive variables (with type inference)
----------------------------------------

Inside the `<script>` tag, we can now use `ref()` directly as if we’re still inside the `setup()` function:

📃 **/src/App.vue**

    <script setup lang="ts">
    import { ref } from 'vue'
    
    const count = ref(0)
    
    </script>
    

Everything about `ref` and `reactive` is still the same, including _type inference_.

The code that we have here looks just like plain old JavaScript because the type of this variable is _inferred_.

![Screen Shot 2021-08-24 at 1.12.22 AM.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.1638811810769.jpg?alt=media&token=e711b492-1d0e-437d-a966-c1948ef177f0)

_Type inference_ is a huge part of using TypeScript. It will guess the type of the variable by just looking at the value you pass into the `ref` function. So even if you don’t specify the variable’s type, your IDE will still be able to show you the type info when you put your cursor over the variable.

Just like `ref()`, we can still use `reactive()` the same way as in the original Composition API syntax:

📃 **/src/App.vue**

    const appInfo = reactive({
      name: 'Counter',
      slogan: 'an app you can count on'
    })
    

The _inferred_ type for `appInfo` will be `{ name: string, slogan: string }`.

![Screen Shot 2021-08-24 at 1.16.10 AM.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.1638811810770.jpg?alt=media&token=ed29887b-309b-410b-9fa9-1bb5742837cd)

* * *

Template
--------

Let’s render all of our variables in the template:

📃 **/src/App.vue**

    <template>
      <div>
        <h1>{{ appInfo.name }}</h1>
        <h2>{{ appInfo.slogan }}</h2>
      </div>
      <p>{{ count }}</p>
    </template>
    

Even with the _script setup syntax_, we’re still writing template code the same way. Again, this is because the _script setup syntax_ is still the Composition API, just under a different skin.

One subtle difference is that we didn’t have to `return` the `appInfo` and `count` from the `setup()` function because our code is not inside a function anymore.

Put your cursor on the `count` variable in the template, Volar will show you that its type is `number` not `Ref<number>`.

![Screen Shot 2021-08-24 at 7.20.31 PM.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.1638811816069.jpg?alt=media&token=4a8af044-7eec-4fbf-bd87-9c755e4eb47b)

That’s because the template automatically “unpacks” the `value` of the `Ref` object.

* * *

Here’s our code so far:

📃 **/src/App.vue**

    <script setup lang="ts">
    import { ref, reactive } from 'vue'
    
    const appInfo = reactive({
      name: 'Counter',
      slogan: 'an app you can count on'
    })
    
    const count = ref(0)
    
    </script>
    
    <template>
      <div>
        <h1>{{ appInfo.name }}</h1>
        <h2>{{ appInfo.slogan }}</h2>
      </div>
      <p>{{ count }}</p>
    </template>
    

Because of type inference, the code in this section looks just like JavaScript. (although it’s really TypeScript)

Next, we’ll write some TypeScript code that actually looks like TypeScript.

* * *

Beyond type inference
---------------------

Even with type inference “autopiloting” for you, you can still annotate your variables whenever you feel like the extra annotations could enhance the readability of your code.

For example, the type of the `appInfo` variable will be inferred as `{ name: string; slogan: string; }`.

Putting your cursor on the variable will show you its type:

![Screen Shot 2021-08-22 at 4.24.01 PM.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F4.1638811818590.jpg?alt=media&token=0898e637-772a-4d10-b889-ddf3984fd14b)

If `{ name: string; slogan: string; }` is too verbose for you, you might want to create a custom type for this:

📃 **/src/App.vue**

    interface AppInfo {
      name: string
      slogan: string
    }
    
    const appInfo: AppInfo = reactive({
      name: 'Counter',
      slogan: 'an app you can count on'
    })
    

All this extra setup makes our code more readable and maintainable. When you look at this code, you know immediately that the variable is meant to be an `AppInfo` type object.

Additionally, when you put your cursor on the variable, it will tell you that the variable `appInfo` is of type `AppInfo`.

![Screen Shot 2021-08-22 at 4.30.38 PM.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F5.1638811822984.jpg?alt=media&token=23438e06-45ce-4287-ae3f-b99261d21855)

Although they mean the same thing, `AppInfo` is so much cleaner than `{ name: string; slogan: string; }`.

* * *

Ref annotation
--------------

But manually specifying the type of a variable is not always so helpful.

For example, we clearly see that the following variable is meant to be a `Ref` of `number`:

    const count = ref(0)
    

Adding the extra annotation would actually make the code look bloated:

    const count: Ref<number> = ref(0)
    

However, there is a common case where type inference is not able to figure out the type we intended.

For example, it’s common in a real app to fetch the initial value from an HTTP service via `onMounted`. So when we first declare the ref, we have to set its value to `null`:

📃 **/src/App.vue**

    // CHANGE
    const count = ref(null)
    
    // NEW
    onMounted(() => {
      fetchCount((initialCount) => {
        count.value = initialCount
      })
    })
    

This code wouldn’t work in TypeScript because our `count` variable has the type `Ref<null>` (guessed by TypeScript’s type inference). That means this `Ref` object can only be set with `null` as its value. Setting it to `initialCount`, which is a `number`, will make TypeScript scream at you.

![Screen Shot 2021-08-24 at 1.07.00 AM.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F6.1638811822985.jpg?alt=media&token=384cc883-fccd-4fb1-9bde-402cd7f1ad58)

To solve this problem, we have to specify our intended type (`number | null`) through the generic argument of `ref()`:

📃 **/src/App.vue**

    // CHANGE
    const count = ref<number | null>(null)
    
    onMounted(() => {
      fetchCount((initialCount) => {
        count.value = initialCount
      })
    })
    

We are basically telling TypeScript that this `Ref` object can be set with either a `number` or `null` as its `value`.

_Now TypeScript is happy again._

As a less common alternative, we can also specify the type info of this ref on the left side of the assignment:

    const count: Ref<number | null> = ref(null)
    

But we would have to import the `Ref` type, which is more typing to get the same result. I’m showing this option here so you know that it’s possible to do it this way, but it’s just less intuitive.

* * *

Here’s our code so far:

📃 **/src/App.vue**

    <script setup lang="ts">
    import { ref, reactive, onMounted } from 'vue'
    import fetchCount from './fetchCount' // a mock fetch function
    
    interface AppInfo {
      name: string
      slogan: string
    }
    
    const appInfo: AppInfo = reactive({
      name: 'Counter',
      slogan: 'an app you can count on'
    })
    
    const count = ref<number | null>(null)
    
    onMounted(() => {
      fetchCount((initialCount) => {
        count.value = initialCount
      })
    })
    
    <script>
    
    <template>
      <div>
        <h1>{{ appInfo.name }}</h1>
        <h2>{{ appInfo.slogan }}</h2>
      </div>
      <p>{{ count }}</p>
    </template>
    
    

The general principle when working with TypeScript is to let type inference do its work, and only step in when type inference is not enough.

Aside from the use cases that we’ve gone through, there are more places where we would need to manually specify the type info, such as in functions.

### Lesson Resources

##### Source Code:

*   [Starting Code](https://github.com/Code-Pop/vue-3-typescript/tree/L2_start)
    
*   [Ending Code](https://github.com/Code-Pop/vue-3-typescript/tree/L2_end)
