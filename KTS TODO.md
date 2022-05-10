---

kanban-plugin: basic

---

## TODO 💭

- [ ] Add Hamburger Menu with Reallocated Buttons.
- [ ] 1. Need to Check on how to add the :loading prop to each list item once clicked.
- [ ] 2. Add "Show Usages" Button with a badge that shows how many references to towers the current turbine, design basis, inventory, materials, & bolts has with towers.
- [ ] 3. Add the meta-data feature that is within the "Delete" request in main.py to the "Get" request section. This will be used to perform Step 2 above by getting references for each tower using the object.


## DOING ⚙️

- [ ] Refactor Linking Feature to its own Component - Just need to refactor into separate component. **Reviewed: Update LibrarySelect so we don't need turbine-select. Also add a mixin for the routeTo. Call this mixin "routeToLibrary". Then remove routeTo from methods & chips.**


## IN REVIEW 🔍



## REPORT 📎

- [ ] Working on creating links for each tower attribute for shortcuts to where they are defined.
- [ ] Added Website Shortcut Icon
- [ ] Fixed local docker setup & was able to run it. The fix was **running the makefile commands in a git bash terminal window rather than a powershell window** Also, Gage helped me authenticate into the towerdesign library.
- [ ] Learned git rebasing to keep my local master and feature branches up to date with remote master.


***

## Archive

- [ ] Fixed Bug where Navigation Side Drawer was not showing on small screens

%% kanban:settings
```
{"kanban-plugin":"basic"}
```
%%