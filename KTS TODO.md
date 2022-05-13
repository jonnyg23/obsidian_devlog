---

kanban-plugin: basic

---

## TODO 💭

- [ ] Add Hamburger Menu with Reallocated Buttons.
- [ ] 1. Need to Check on how to add the :loading prop to each list item once clicked.
- [ ] Question: When refreshing webpage  loading of tower usage takes ~20 sec. Also, saving code change makes the page buggy sometimes.
- [ ] **Add "/intent" to end of path for routeToLibrary if name is "tower".**
- [ ] Add loading progress bar on top of page when going to different page routes.


## DOING ⚙️

- [ ] When switching libraries in Navbar, I this error: `Error in render: "TypeError: Cannot read properties of null (reading 'references')"<br>`
- [ ] Question: **Materials & Bolts don't have tower refs even though they should?**


## IN REVIEW 🔍

- [ ] 2. Add "Show Usages" Button with a badge that shows how many references to towers the current turbine, design basis, inventory, materials, & bolts has with towers.
- [ ] Make buttons disabled in CRUDView.vue until data from tower usage has been loaded.
- [ ] Only show unarchived towers.


## REPORT 📎

- [ ] Working on creating links for each tower attribute for shortcuts to where they are defined.
- [ ] Added Website Shortcut Icon
- [ ] Fixed local docker setup & was able to run it. The fix was **running the makefile commands in a git bash terminal window rather than a powershell window** Also, Gage helped me authenticate into the towerdesign library.
- [ ] Learned git rebasing to keep my local master and feature branches up to date with remote master.
- [ ] Refactor Linking Feature to its own Component - Just need to refactor into separate component. **Reviewed: Update LibrarySelect so we don't need turbine-select. Also add a mixin for the routeTo. Call this mixin "routeToLibrary". Then remove routeTo from methods & chips.**


***

## Archive

- [ ] Fixed Bug where Navigation Side Drawer was not showing on small screens

%% kanban:settings
```
{"kanban-plugin":"basic"}
```
%%