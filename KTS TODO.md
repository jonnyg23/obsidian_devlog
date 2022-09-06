---

kanban-plugin: basic

---

## TODO 💭

- [ ] Add loading progress bar on top of page when going to different page routes.
- [ ] Question: **Materials & Bolts don't have tower refs even though they should?** ==This is a separate task==
- [ ] **Bug**: Tower Intent Sketch only shows after pressing the back button and selecting "no" on leaving the tower with unsaved changes. (Unless the sketch takes longer to load? but it shouldn't since it is being read from dynamoDB)


## DOING ⚙️

- [ ] Question on Store Revision Numbers on Tower Release: We need to add “previous_release” as a field to model_number table. You can read that number, add one, and update it when we’re releasing a new tower. **How should this be implemented?**
- [ ] We will need to add rev# and status_dates to GSI projected attrs & add upgrade rev# when selecting **copyTower** on tower intent & tower list
- [ ] Add error handler in main.py for any operations that need to trigger and error, review past project for this.
- [ ] `saveTower()` works if forcing a "BadRequest" error on the backend>main.py>update_tower(). This is because there is a catch statement on the dispatch. I think we need to add catch statements to the problem dispatches.


## Done

- [ ] Fix Title so changelog history is on a new line for the History Modal
- [ ] Make a removed piece of data look better rather than being blank
- [ ] Make loadset changes look better and have a name instead of just the loadset number - check for other things that need a name instead of uuids
- [ ] When tower is moved to release increment revision number up by one
- [ ] Add dropdown popover menu to status button with ability to change status.


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