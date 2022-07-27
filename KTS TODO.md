---

kanban-plugin: basic

---

## TODO 💭

- [ ] Add loading progress bar on top of page when going to different page routes.
- [ ] Question: **Materials & Bolts don't have tower refs even though they should?** ==This is a separate task==
- [ ] **Bug**: Tower Intent Sketch only shows after pressing the back button and selecting "no" on leaving the tower with unsaved changes. (Unless the sketch takes longer to load? but it shouldn't since it is being read from dynamoDB)


## DOING ⚙️

- [ ] Check on NavList to make sure it renders the data table correctly with both refresh and clicking through from the main screen!
- [ ] Fix "Other" so it shows all towers that don't have a top level model number.


## Done

- [ ] Add some `oneoff` scripts that will<br>1. Read and Update and tower with the projected attributes. (Use the pytest you already created for most of this code.) <br> 2. Another script that will list all towers that don't have a model number in the top level or data.model_number level.
- [ ] Change to sorting in alphabetical order in model number combobox select
- [ ] Put in new **Jira Task** for automatically navigating and opening the NavList to the appropriate turbine & model number that is in the params/query of the URL.
- [ ] Create a **check_schema** method in the super class `DynamoCrud` that will perform the validation schema operations and error return as shown in main.py>create_item. Then, call this super class method in TowerCrud.create&.update passing `data` into this method before any db operations are performed.
- [ ] Add a new add_crud_rules2() in `main.py` that will use the new TowerCrud operations. You might be able to get rid of the `/tower` endpoint in favor of `/towers`. In this way, you will have 2 GET requests. One to `/tower` that returns a specific tower and uses `TowerCrud.read` and another that is `/towers` which lists all towers will certain params using `TowerCrud.list`


## IN REVIEW 🔍



## REPORT 📎



***

## Archive

- [ ] Fixed Bug where Navigation Side Drawer was not showing on small screens

%% kanban:settings
```
{"kanban-plugin":"basic"}
```
%%