---
tags: 📝/🌱
aliases:
  -
cssclass:
---

# [[Vue.js]]

> A [[Javascript]] Framework for User Interfaces
---

## Q&A

- Difference between `computed` and `methods` sections?
	- Computed is for things that are basically a function of things in the state at the moment. If you have some props and some data that gets mutated, and you want to have a variable that's a combination of those and auto updates with every change to the data, use a computed
	- If you want to send a message, have an effect on the API and hit some http end point, use a method.
	- Methods can also have arguments so that's where you'd put a function that needs to take in arguments from things in the template and give back a result.
	- ✏️ Note: Use computed for small things like sorting data.


---

🔗 Links to this page:
[[Javascript]]
[[JS Frameworks]]

