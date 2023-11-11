Typing Your Callback Functions
==============================

Just like reactive variables, callback functions can also be easily inferred. In this lesson, we’ll explore the various ways to use functions in TypeScript with the help of type inference.

Functions
---------

Let’s add a button that triggers a function on `click`:

📃 **/src/App.vue**

    function addCount() {
      if (count.value !== null) {
        count.value += 1
      }
    }
    
    <template>
      ...
      <p>
        <button @click="addCount">Add</button>
      </p>
    <template>
    

TypeScript won’t complain about this `addCount` function.

But what if we want to pass a number to the function as a parameter?

📃 **/src/App.vue**

    function addCount(num) {
      if (count.value !== null) {
        count.value += num
      }
    }
    
    <template>
      ...
      <p>
        <button @click="addCount(1)">Add</button>
      </p>
    <template>
    

This situation is too ambiguous for _type inference_ to guess what the type of this parameter should be, so TypeScript will use `any` as the type for `num`. Depending on your **tsconfig.json**, TypeScript might scream at you.

![Screen Shot 2021-08-24 at 1.21.09 AM.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.1639001598645.jpg?alt=media&token=b2957817-e1a3-4caf-ad51-93fb77cec877)

To improve the code, we simply need to annotate the parameter with its intended type:

📃 **/src/App.vue**

    function addCount(num: number) {
      if (count.value !== null) {
        count.value += num
      }
    }
    

Now TypeScript is happy again.

* * *

Type inference for function return
----------------------------------

Notice that we’ve only been talking about the parameter type of a function. What about the return type of a function?

Function return is easily _inferred_ by TypeScript.

For example, when we’re using `computed`:

    const nextCount = computed(() => {
      if (count.value !== null) {
        return count.value + 1
      }
      return null
    })
    

Since TypeScript knows that `count.value` is a number, a number + 1 is still just a number. Type inference is smart enough to figure that out by itself.

![Screen Shot 2021-09-20 at 2.18.44 PM.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.1639001598646.jpg?alt=media&token=56796f80-6ea9-4cab-a27c-b2118a958139)

So there’s no need to manually specify the return type of the function.

* * *

Type inference for inline function
----------------------------------

You might have noticed that we didn’t specify the type of the parameter in our `fetchCount` callback earlier:

📃 **/src/App.vue**

    fetchCount((initialCount) => {
      count.value = initialCount
    })
    

How is this callback function different from our `addCount` function?

In short, type inference is able to infer the parameter type if the function is used as an inline callback function.

For a more elaborate explanation, let’s do a little experiment.

Extract the callback function to its own variable:

📃 **/src/App.vue**

    const cb = (initialCount) => {
      count.value = initialCount
    }
    
    fetchCount(cb)
    

Depending on your **tsconfig**, the compiler might start screaming about `initialCount` needing an explicit type.

![Screen Shot 2021-08-24 at 1.26.17 AM.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.1639001605668.jpg?alt=media&token=f877a563-91fc-497d-b515-890bfd33385e)

Basically, type inference can only make its best guess when the callback function is passed inline to `fetchCount`. TypeScript knows what `fetchCount` accepts as a parameter, so it just “connects the dots.” But if the callback function is defined beforehand, TypeScript wouldn’t be able to connect those dots because it has no idea where you would use this function and you could have used it in multiple places as a callback function.

Once again, type inference will work effectively if the callback is passed inline like this:

📃 **/src/App.vue**

    fetchCount((initialCount) => {
      count.value = initialCount
    })
    

* * *

Here’s our code so far:

📃 **/src/App.vue**

    <script setup lang="ts">
    import { ref, reactive, onMounted } form 'vue'
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
    
    function addCount(num: number) {
      if (count.value !== null) {
        count.value += num
      }
    }
    
    <script>
    
    <template>
      <div>
        <h1>{{ appInfo.name }}</h1>
        <h2>{{ appInfo.slogan }}</h2>
      </div>
      <p>{{ count }}</p>
      <p>
        <button @click="addCount(1)">Add</button>
      </p>
    </template>
    

Until now, the way we’ve been writing TypeScript in this new _script setup syntax_ is still the same as how we would do it in the `setup()` function.

Next, we’ll move into the territory of the exclusive features in _script setup_. The TypeScript experience is vastly different there.

### Lesson Resources

##### Source Code:

*   [Starting Code](https://github.com/Code-Pop/vue-3-typescript/tree/L3_start)
    
*   [Ending Code](https://github.com/Code-Pop/vue-3-typescript/tree/L3_end)
