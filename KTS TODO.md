---

kanban-plugin: basic

---

## TODO 💭

- [ ] Add loading progress bar on top of page when going to different page routes.
- [ ] Question: **Materials & Bolts don't have tower refs even though they should?** ==This is a separate task==
- [ ] **Bug**: Tower Intent Sketch only shows after pressing the back button and selecting "no" on leaving the tower with unsaved changes. (Unless the sketch takes longer to load? but it shouldn't since it is being read from dynamoDB)


## DOING ⚙️

- [ ] Update **main.py** to use new `TowerCrud.query` method. **Review TODOs**
- [ ] Update backend and frontend to make use of TowerCrud.query


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