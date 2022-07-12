---

kanban-plugin: basic

---

## TODO 💭

- [ ] Add loading progress bar on top of page when going to different page routes.
- [ ] Question: **Materials & Bolts don't have tower refs even though they should?** ==This is a separate task==
- [ ] **Bug**: Tower Intent Sketch only shows after pressing the back button and selecting "no" on leaving the tower with unsaved changes. (Unless the sketch takes longer to load? but it shouldn't since it is being read from dynamoDB)


## DOING ⚙️

- [ ] Speed up tower list page task <br>1. Add crud rules for adding projection information to top level of db response. <br>2. metadata: sections (data.sections), hub_height (data.hub_height) , turbine uuid (data.turbine), pure boto3 command-line no python. <br> 3. On terraform end: Global secondary index
- [ ] Add backend api to main.py of tdweb using model_number
- [ ] ⚠️ Look into the new Turbine and new Design Basis buttons since they don't pop up the create new component modal.
- [ ] Task TODOs:
- [ ] 1. Add spacing between model number combobox and tower name.
- [ ] 2. When implementing the new TowerCrud operation in tdstore, use turbine id and model number numeric NOT model number name when querying for tower list.
- [ ] 3. Move turbine v-select above model number combobox in the NewTowerDialog
- [ ] 4. When implementing the Step 2, Make sure to also make a change to tdstore's ModelNumberCrud for create feature. Return the object which was added to the db as a dictionary object.


## IN REVIEW 🔍

- [ ] Adding highlights to srf<1 summary report
- [ ] Add links to certification report


## REPORT 📎

- [ ] PR design basis one analyzer task


***

## Archive

- [ ] Fixed Bug where Navigation Side Drawer was not showing on small screens

%% kanban:settings
```
{"kanban-plugin":"basic"}
```
%%