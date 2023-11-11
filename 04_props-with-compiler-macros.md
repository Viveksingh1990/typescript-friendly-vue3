Props with Compiler Macros
==========================

Props
-----

In this lesson, we’ll look at the most important change in the script setup syntax, the new way of defining props and setting their default values.

Before we can demonstrate how to create props, we need to break our **App** component into two so that we can pass props from one to the other.

Let’s extract the counting logic and its related template code to a new component:

📃 **/src/components/Counter.vue**

    <script setup lang="ts">
    import { ref, onMounted } from 'vue'
    import fetchCount from '../services/fetchCount'
    
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
    
    </script>
    
    <template>
      <p>{{ count }}</p>
      <p>
        <button @click="addCount(1)">Add</button>
      </p>
    </template>
    

Here’s the **App** component after importing and using the **Counter** component:

📃 **/src/App.vue**

    <script setup lang="ts">
    import { reactive } from 'vue'
    import Counter from './components/Counter.vue'
    
    interface AppInfo {
      name: string
      slogan: string
    }
    
    const appInfo: AppInfo = reactive({
      name: 'Counter',
      slogan: 'an app you can count on'
    })
    
    </script>
    
    <template>
      <div>
        <h1>{{ appInfo.name }}</h1>
        <h2>{{ appInfo.slogan }}</h2>
      </div>
      <Counter></Counter>
    </template>
    

Now, let’s make the **Counter** component accept two props:

*   `limit` (the highest `number` the counter can go up to)
*   `alertMessageOnLimit` (the `string` message that we’ll show when the count can’t go up anymore)

In the _script setup syntax_, we have to use the `defineProps` function:

📃 **/src/components/Counter.vue**

    const props = defineProps<{
      limit: number,
      alertMessageOnLimit: string
    }>()
    

We’re defining the type of each prop item using pure TypeScript annotations. Note that we’re passing the object type through the angular brackets, not the parentheses. That means we’re passing it into the function as a _generic argument type_.

`defineProps` is a function called a compiler macro. There are a few other compiler macros we’ll talk about soon. These functions are only called during compile-time; they won’t appear in your runtime code. That’s why we didn’t have to import them before using them. These compile-time functions can only be used within `<script setup>`.

A common gotcha here is attempting to destructure `props`, which causes loss of reactivity.

    const { limit, alertMessageOnLimit } = defineProps<{
      limit: number,
      alertMessageOnLimit: string
    }>()
    

However, the Vue team is working on accommodating this. So this may not be an issue by the time you’re reading this tutorial.

But for code clarity, we’ll still be using the `props` variable through this tutorial:

    const props = defineProps<{
      limit: number,
      alertMessageOnLimit: string
    }>()
    

* * *

Function argument
-----------------

As an alternative, we can still pass the traditional `props` config object into the `defineProps` as a _function argument_ (through the parentheses this time, not the angular brackets):

    const props = defineProps({
      limit: { type: Number, required true },
      alertMessageOnLimit: { type: String, required: true }
    })
    

But be warned that these are not TypeScript annotations, the `Number` type and the `String` type are all capitalized. (the TypeScript counterparts are all lowercase)

So to recap, `defineProps` offers two ways to define the types of our props, using a _generic type argument_ and using a _function argument._

_Generic type argument:_

    const props = defineProps<{
      limit: number,
      alertMessageOnLimit: string
    }>()
    

_Function argument_:

    const props = defineProps({
      limit: { type: Number, required: true },
      alertMessageOnLimit: { type: String, required: true }
    })
    

Both of these methods will provide you with static type checking for your props. But the _generic type argument_ is in pure TypeScript, and that’s also the recommended method, so we’ll be sticking with that.

We can also create an interface for the _type argument_:

    interface Props {
      limit: number
      alertMessageOnLimit: string
    }
    
    const props = defineProps<Props>()
    

This makes the code easier to read, and it makes it easier to set up default values (as we’ll see in a bit).

* * *

Next, let’s use the two new props in the logic:

📃 **/src/components/Counter.vue**

    function addCount(num: number) {
      if (count.value !== null) {
        if (count.value >= props.limit) {
          alert(props.alertMessageOnLimit)
        }
        else {
          count.value += num
        }
      }
    }
    

This will cap the `count` at a certain number, and it will alert the user when they try to add beyond that number.

Now back in the parent component, you should see that TypeScript is currently screaming at you for not passing in the required props:

📃 **/src/App.vue**

![Screen Shot 2021-08-24 at 1.01.14 PM.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.1639178968727.jpg?alt=media&token=b51f3357-d934-4533-9422-e7f2d3eb803c)

This is a good thing, and that’s the power of static type checking.

To fix this, we just have to set the props on the element:

📃 **/src/App.vue**

    <template>
      ...
      <Counter 
        :limit="10" 
        :alert-message-on-limit="'can not go any higher'"
      ></Counter>
    </template>
    

Now TypeScript is happy again.

* * *

Here’s our code so far:

📃 **/src/App.vue**

    <script setup lang="ts">
    import { reactive } from 'vue'
    import Counter from './components/Counter.vue'
    
    interface AppInfo {
      name: string
      slogan: string
    }
    
    const appInfo: AppInfo = reactive({
      name: 'Counter',
      slogan: 'an app you can count on'
    })
    
    </script>
    
    <template>
      <div>
        <h1>{{ appInfo.name }}</h1>
        <h2>{{ appInfo.slogan }}</h2>
      </div>
      <Counter 
        :limit="10" 
        :alert-message-on-limit="'can not go any higher'"
      ></Counter>
    </template>
    

📃 **/src/components/Counter.vue**

    <script setup lang="ts">
    import { ref, onMounted } from 'vue'
    import fetchCount from '../services/fetchCount'
    
    interface Props {
      limit: number
      alertMessageOnLimit?: string
    }
    
    const props = defineProps<Props>()
    
    const count = ref<number | null>(null)
    
    onMounted(() => {
      fetchCount((initialCount) => {
        count.value = initialCount
      })
    })
    
    function addCount(num: number) {
      if (count.value !== null) {
        if (count.value >= props.limit) {
          alert(props.alertMessageOnLimit)
        }
        else {
          count.value += num
        }
      }
    }
    
    </script>
    
    <template>
      <p>{{ count }}</p>
      <p>
        <button @click="addCount(1)">Add</button>
      </p>
    </template>
    

Another benefit of using `defineProps` with _generic type argument_ is that it offers a cleaner way to define default values. We’ll look at how to do that next.

* * *

Default Prop Value
------------------

To define a default value for a prop, we have to first make that prop optional in the `interface`:

📃 **/src/components/Counter.vue**

    interface Props {
      limit: number
      alertMessageOnLimit?: string
    }
    

Here, we’re making `alertMessageOnLimit` optional.

Next, we have to use another function called `withDefaults` to define the actual default value for `alertMessageOnLimit`:

📃 **/src/components/Counter.vue**

    const props = withDefaults(defineProps<Props>(), {
      alertMessageOnLimit: 'can not go any higher'
    })
    

We’re using `withDefaults` and `defineProps` in a function composition pattern, that is, putting the `defineProps` function call inside `withDefaults` as an argument. And setting up the default value of `alertMessageOnLimit` in an object literal as the second argument.

* * *

Now back in the parent component, we have the choice to not pass in `alertMessageOnLimit`:

📃 **/src/App.vue**

    <template>
      <div>
        <h1>{{ appInfo.name }}</h1>
        <h2>{{ appInfo.slogan }}</h2>
      </div>
      <Counter 
        :limit="10"
      ></Counter>
    </template>
    

And `Counter` will just opt for the default message that we’ve specified.

### Lesson Resources

##### Source Code:

*   [Starting Code](https://github.com/Code-Pop/vue-3-typescript/tree/L4_start)
    
*   [Ending Code](https://github.com/Code-Pop/vue-3-typescript/tree/L4_end)
