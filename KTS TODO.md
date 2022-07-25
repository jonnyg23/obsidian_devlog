---

kanban-plugin: basic

---

## TODO 💭

- [ ] Add loading progress bar on top of page when going to different page routes.
- [ ] Question: **Materials & Bolts don't have tower refs even though they should?** ==This is a separate task==
- [ ] **Bug**: Tower Intent Sketch only shows after pressing the back button and selecting "no" on leaving the tower with unsaved changes. (Unless the sketch takes longer to load? but it shouldn't since it is being read from dynamoDB)


## DOING ⚙️

- [ ] Change to sorting in alphabetical order in model number combobox select
- [ ] Put in new **Jira Task** for automatically navigating and opening the NavList to the appropriate turbine & model number that is in the params/query of the URL.
- [ ] Add some `oneoff` scripts that will<br>1. Read and Update and tower with the projected attributes. (Use the pytest you already created for most of this code.) <br> 2. Another script that will list all towers that don't have a model number in the top level or data.model_number level.
- [ ] Add a new add_crud_rules2() in `main.py` that will use the new TowerCrud operations. You might be able to get rid of the `/tower` endpoint in favor of `/towers`. In this way, you will have 2 GET requests. One to `/tower` that returns a specific tower and uses `TowerCrud.read` and another that is `/towers` which lists all towers will certain params using `TowerCrud.list`
- [ ] Create a **check_schema** method in the super class `DynamoCrud` that will perform the validation schema operations and error return as shown in main.py>create_item. Then, call this super class method in TowerCrud.create&.update passing `data` into this method before any db operations are performed.


## Done

- [ ] Update Model Number combobox to have ***visual feedback** when adding a new model number. Make it blue or something when its a new number and doesn't match any of the ones in Vuex store and change it back to black once it is added.
- [ ] ***Trimmed white space from the outsides of any combobox input**
- [ ] Tested NewTowerDialog's use of model number.
- [ ] Use TowerCrud.update method as a quick way to add the top level "turbine" "hub_height""etc" attributes to **only my personal Tower** that I'm testing on. In this way, I can test the NavList on my specific tower
- [ ] Test TowerCrud.list when tower has all the needed top level attributes
- [ ] Use the new model number once created by calling addModelNumber or something which will append the new model number result to the Vuex store modelNumbers


## IN REVIEW 🔍

- [ ] Adding highlights to srf<1 summary report
- [ ] Add links to certification report
- [ ] Add backend api to main.py of tdweb using model_number
- [ ] 1. Add spacing between model number combobox and tower name.
- [ ] 2. When implementing the new TowerCrud operation in tdstore, use turbine id and model number numeric NOT model number name when querying for tower list.
- [ ] 3. Move turbine v-select above model number combobox in the NewTowerDialog
- [ ] 4. When implementing the Step 2, Make sure to also make a change to tdstore's ModelNumberCrud for create feature. Return the object which was added to the db as a dictionary object.
- [ ] 5. Rename "All" in NavList to "Other"
- [ ] Ensure model number selections are filtered by turbine before showing in combobox
- [ ] Also update **main.py** to return from `ModelNumberCrud.create` call


## REPORT 📎

- [ ] PR in tdstore for TowerCrud.list operation
- [ ] ALso added comments for an implementation for using the query on GSI.
- [ ] Today Working on make use of this new operation in the backend and frontend NavList


***

## Archive

- [ ] Fixed Bug where Navigation Side Drawer was not showing on small screens

%% kanban:settings
```
{"kanban-plugin":"basic"}
```
%%