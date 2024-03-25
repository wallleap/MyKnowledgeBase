# How to Access DOM Element Using Vue 3 & Composition API

In javascript, we can easily target a dom using getElementById, getElementByClassName, getElementByTagname, or querySelector. But dom manipulation is costly. It’s not a vue way to target a dom element. So, how can we achieve this in a vue way?

***Template Ref:\*** *In vue,* ***t\***emplate ref allows to target a dom elements or instance of child component after their initial rendering.

Suppose, you want to target **h1** elemenet from your component .

```
<template>
 <h1> hello world </h1>
</template>
```

To achive this with composition api you need to follow **three** steps:

**step 1:** Add **ref** attribute with your target element.

```
<template>
  <h1 ref="headline"> hello world </h1>
</template>
```

**step 2**: Declare a **reactive state** for that element with the **same template ref name.**It will hold the reference of the element.

```
<script setup>
import { ref } from "vue";
const headline=ref(null);
</script>
```

**step 3**: In Vue 3, the script setup runs before anything.So,you can obtain the element instance in that reactive state when the component will render.

As,onBeforeMount hook runs before dom creation.So, reactive state will be null.But onMounted/onUpdated hook is called after the dom creation. So you can obtain the element instance here.

```
<script setup>
 import { onBeforeMount,onMounted, onUpdated, ref } from "vue";
 const headline=ref(null);

 onBeforeMount(()=>{
 console.log(headline.value);
 //this hook runs before component is rendered && print null

 }
 onMounted(()=>{
 console.log(headline.value)
//print the element
 }

onUpdated(()=>{
 console.log(headline.value)
//print the element
 }
</script>
```

# Complete Code

```
<script setup>
 import { onMounted, ref } from "vue";
 const headline=ref(null);
 onMounted(()=>{
 console.log(headline.value) 
 }
</script>

<template>
 <h1 ref="headline"> hello world </h1>
</template>
```

*Thank you for reading until the end. Please consider following the writer and this publication. Visit* [***Stackademic\***](https://stackademic.com/) *to find out more about how we are democratizing free programming education around the world.*