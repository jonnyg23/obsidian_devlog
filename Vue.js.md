---
tags: 📝/🌱
aliases:
  -
cssclass:
---

# [[Vue.js]]

> A [[Javascript]] Framework for User Interfaces

## Layout
---

This is a typical example of a component layout:

```js
<template>
	<h1>{{ fullName }}</h1>
	<component-here :title="title"/>
</template>

<script>
import ComponentHere from '@/components/ComponentHere'

export default {
	name: 'ComponentName',
	components: {
		ComponentHere,
	},
	props: {
		title: {
			type: String,
			default: "Default Title",
		},
	},
	data() {
		return {
			profile: {
				firstName: "Jonny",
				lastName: "G",
			},
		}
	},
	mounted() {
		// On mounted, run startup js code here
		this.fetchDataPromise()
	},
	updated() {
		// `updated` is another vue lifecyle hook
	},
	computed: {
		fullName() {
			return this.profile.firstname + " " + this.profile.lastName
		},
	},
	methods: {
		fetchDataPromise() {
			return this.$store.dispatch("fetchBackendInfo")
				.then((response) => return response)
				.catch((error) => console.log("Error fetching data promise: ", error))
		},
	},
}
</script>

<style> // Can be `style scoped` or even scss
</style>
```

## Q&A

- Difference between `computed` and `methods` sections?
	- Computed is for things that are basically a function of things in the state at the moment. If you have some props and some data that gets mutated, and you want to have a variable that's a combination of those and auto updates with every change to the data, use a computed
	- If you want to send a message, have an effect on the API and hit some http end point, use a method.
	- Methods can also have arguments so that's where you'd put a function that needs to take in arguments from things in the template and give back a result.
	- ✏️ Note: Use computed for small things like sorting data.

- What is a `watch`er? Should I use it?
	- A watcher is 


---

🔗 Links to this page:
- [[Javascript]]
- [[JS Frameworks]]
- [[Vuetify]]

