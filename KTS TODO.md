---

kanban-plugin: basic

---

## TODO 💭

- [ ] Add loading progress bar on top of page when going to different page routes.
- [ ] Question: **Materials & Bolts don't have tower refs even though they should?** ==This is a separate task==
- [ ] **Bug**: Tower Intent Sketch only shows after pressing the back button and selecting "no" on leaving the tower with unsaved changes. (Unless the sketch takes longer to load? but it shouldn't since it is being read from dynamoDB)


## DOING ⚙️

- [ ] Fix Title so changelog history is on a new line for the History Modal
- [ ] Make a removed piece of data look better rather than being blank
- [ ] Make loadset changes look better and have a name instead of just the loadset number - check for other things that need a name instead of uuids
- [ ] on TowerCrud.create add revision number 1 when tower is made
- [ ] When tower is moved to release increment revision number up by one
- [ ] Add revision number to projections top level for towers from tower intent


## Done



## IN REVIEW 🔍

- [ ] Figure out copy button bug in tower, ensure that you can delete as well
- [ ] add release info task


## REPORT 📎



***

## Archive

- [ ] Fixed Bug where Navigation Side Drawer was not showing on small screens

%% kanban:settings
```
{"kanban-plugin":"basic"}
```
%%