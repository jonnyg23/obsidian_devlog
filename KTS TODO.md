---

kanban-plugin: basic

---

## TODO 💭

- [ ] Add Hamburger Menu with Reallocated Buttons.
- [ ] 1. Need to Check on how to add the :loading prop to each list item once clicked.
- [ ] Add loading progress bar on top of page when going to different page routes.
- [ ] Question: **Materials & Bolts don't have tower refs even though they should?** ==This is a separate task==


## DOING ⚙️

- [ ] Speed up tower list page task <br>1. Add crud rules for adding projection information to top level of db response. <br>2. metadata: sections (data.sections), hub_height (data.hub_height) , turbine uuid (data.turbine), pure boto3 command-line no python. <br> 3. On terraform end: Global secondary index
- [ ] Add these questions to tower list speedup review:<br>1. Line 402-404 main.py I am not sure if it will export the items correctly as I cannot test it without the GSI table. But I followed the way list_items returns the response items.<br>2. I don't know what the name of the GSI will be so I set the index name on line 399 of main.py to IndexName='turbine-uuid-index'<br>3. I added a route watcher on TowerList.vue and re-added the dispatch(fetchTowersByTurbine) to the mounted section. This way it will rerun a dispatch if there is a route change and if someone navigates directly to a specific tower/turbine_id route.<br>4. I created the dispatch method "fetchTowersByTurbine" imitating the "fetchTowers" dispatch. Because of this, I'm unsure if Line 85 of front-end/src/store/index.js should have context.commit("updateTowers")


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