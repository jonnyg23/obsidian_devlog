---

kanban-plugin: basic

---

## TODO 💭

- [ ] Add Hamburger Menu with Reallocated Buttons.
- [ ] 1. Need to Check on how to add the :loading prop to each list item once clicked.
- [ ] Add loading progress bar on top of page when going to different page routes.
- [ ] Question: **Materials & Bolts don't have tower refs even though they should?** ==This is a separate task==


## DOING ⚙️

- [ ] Speed up Tower List page by changing the list_items def on the main.py backend


## IN REVIEW 🔍

- [ ] Remove white space in Design Basis Analyzers; Add elevation to each card to distinguish them from one another


## REPORT 📎

- [ ] Working on creating links for each tower attribute for shortcuts to where they are defined.
- [ ] Added Website Shortcut Icon
- [ ] Fixed local docker setup & was able to run it. The fix was **running the makefile commands in a git bash terminal window rather than a powershell window** Also, Gage helped me authenticate into the towerdesign library.
- [ ] Refactor Linking Feature to its own Component - Just need to refactor into separate component. **Reviewed: Update LibrarySelect so we don't need turbine-select. Also add a mixin for the routeTo. Call this mixin "routeToLibrary". Then remove routeTo from methods & chips.**
- [ ] Change color of Usage button
- [ ] 2. Add "Show Usages" Button with a badge that shows how many references to towers the current turbine, design basis, inventory, materials, & bolts has with towers.
- [ ] Only show unarchived towers.
- [ ] Make buttons disabled in CRUDView.vue until data from tower usage has been loaded.


***

## Archive

- [ ] Fixed Bug where Navigation Side Drawer was not showing on small screens

%% kanban:settings
```
{"kanban-plugin":"basic"}
```
%%