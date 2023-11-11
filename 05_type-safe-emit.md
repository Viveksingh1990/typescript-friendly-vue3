Type-Safe Emit
==============

Emit
----

Just like props, emit has a total makeover in the script setup syntax. In this lesson, we’ll pick up another compiler macro for working emit.

To demonstrate the new annotation syntax for emits, let’s first extract the button into a new component that can emit a custom event:

📃 **/src/components/ControlBar.vue**

    <script setup lang="ts">
    </script>
    
    <template>
      <p>
        <button @click="addCount(1)">Add</button>
      </p>
    </template>
    

To define the events that we’re allowed to emit from this component, we have to use the `defineEmits` function:

📃 **/src/components/ControlBar.vue**

    <script setup lang="ts">
    
    const emit = defineEmits<{ 
      (event: 'add-count', num: number): void 
    }>()
    
    </script>
    

Technically, this is an object type with call signature.

We can add additional call signature inside the object type:

📃 **/src/components/ControlBar.vue**

    <script setup lang="ts">
    
    const emit = defineEmits<{ 
      (event: 'add-count', num: number): void 
      (event: 'reset-count'): void 
    }>()
    
    </script>
    

What this means is that `emit` will be a function that can act as either of those two call signatures.

The `add-count` event will be emitted with a payload called `num`, and the `reset-count` event has no payload.

Next, we have to modify the template to emit these events:

📃 **/src/components/ControlBar.vue**

    <template>
      <p>
        <button @click="emit('add-count', 1)">Add</button>
        <button @click="emit('reset-count')">Reset</button>
      </p>
    </template>
    

Notice that when we emit, we are doing so with the event name as the first parameter, just like the call signatures we specified. And we’re passing in any additional parameters as required by each event.

If we didn’t specify the `reset-count` event with `defineEmits`, we wouldn’t be able to emit it in our component:

📃 **/src/components/ControlBar.vue**

![Screen Shot 2021-08-23 at 1.49.12 AM.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.1639180301388.jpg?alt=media&token=f064ad2c-6b83-4bec-b30a-2fbd1328bb35)

And if we were to emit the wrong event name, it would scream at us, too:

📃 **/src/components/ControlBar.vue**

![Screen Shot 2021-08-23 at 2.39.41 AM.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.1639180307876.jpg?alt=media&token=4f822d2f-bde9-4e68-9b0c-f0b03a733529)

Once again, that’s the power of static type checking.

To complete the event handling cycle, we have to go back to the parent component **Counter.vue** and register a callback function for the `add-count` event:

📃 **/src/components/Counter.vue**

    <template>
      <p>{{ count }}</p>
      <AddButton @add-count="foo"></AddButton>
    </template>
    

If you put your cursor on `@add-count`, it will show you the type of function that can be used as a callback for the `add-count` event:

📃 **/src/components/Counter.vue**

![Screen Shot 2021-09-22 at 3.43.18 PM.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.1639180311761.jpg?alt=media&token=624530ac-c8f2-4ba7-96b3-f992ad582828)

Notice that the bottom box (the _Volar_ box) says `(num: number) => void`, and our `addCount` function fits this description.

Now you might be wondering… _Where does this_ `(num: number) => void` _thing come from?_

It’s actually derived from the `defineEmits` type argument that we set up earlier.

This one:

📃 **/src/components/ControlBar.vue**

    <script setup lang="ts">
    
    const emit = defineEmits<{ 
      (event: 'add-count', num: number): void 
      (event: 'reset-count'): void 
    }>()
    
    </script>
    

So Vue took the `(event: 'add-count', num: number): void` part and stripped away the `event` parameter. That gives us the `(num: number) => void` type for our `add-count` event.

![emit_type.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F4.opt.1639180315842.jpg?alt=media&token=3f59495a-525e-4b2e-8a4f-9aff176a6667)

* * *

Inline event handling
---------------------

Let’s also do something when `reset-count` is emitted:

📃 **/src/components/Counter.vue**

    <ControlBar 
      @add-count="addCount"
      @reset-count="count = 0"
    ></ControlBar>
    

(because we are still inside the template, we can just use `count` instead of `count.value`)

Put your cursor on `@reset-count`. You’ll see the type of callback it accepts:

📃 **/src/components/Counter.vue**

![Screen Shot 2021-09-22 at 3.47.20 PM.png](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F5.1639180320913.jpg?alt=media&token=9ed54664-af85-4c29-91b6-799f1bce6883)

So it says `($event? undefined) => void`. You might be wondering… _Why is_ `count = 0` _an acceptable value?_ _It’s not even a function, it’s a statement!_

The `count = 0` bit will actually get compiled into a function, something like `() => { count = 0 }`. It’s technically still a function, and an acceptable callback for the `reset-count` event.

* * *

Here’s our _final_ code:

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
      ></Counter>
    </template>
    

📃 **/src/components/Counter.vue**

    <script setup lang="ts">
    import { ref, onMounted } from 'vue'
    import fetchCount from '../services/fetchCount'
    import AddButton from './AddButton.vue'
    
    interface Props {
      limit: number
      alertMessageOnLimit?: string
    }
    
    const props = withDefaults(defineProps<Props>(), {
      alertMessageOnLimit: 'can not go any higher'
    })
    
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
      <AddButton 
        @add-count="addCount"
        @reset-count="count = 0"
      ></AddButton>
    </template>
    

📃 **/src/components/AddButton.vue**

    <script setup lang="ts">
    
    const emit = defineEmits<{ 
      (event: 'add-count', num: number): void  
      (event: 'reset-count'): void 
    }>()
    
    </script>
    
    <template>
      <p>
        <button @click="emit('add-count', 1)">Add</button>
        <button @click="emit('reset-count')">Reset</button>
      </p>
    </template>
    

Next, we’ll do a cross-framework comparison to see how this new TypeScript experience fares alongside another very popular framework.

* * *

Course Conclusion
-----------------

We’ve gone through all the essential aspects of using TypeScript with Vue in this new _Script Setup_ syntax. As you can hopefully see, it is a significant improvement from the TypeScript experience of the original Composition API.

If you’re starting a new Vue app project in TypeScript, it’s recommended to create your new components in this new _Script Setup_ syntax.

### Lesson Resources

##### Source Code:

*   [Starting Code](https://github.com/Code-Pop/vue-3-typescript/tree/L5_start)
    
*   [Ending Code](https://github.com/Code-Pop/vue-3-typescript/tree/L5_end)
