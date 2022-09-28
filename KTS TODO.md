---

kanban-plugin: basic

---

## TODO 💭

- [ ] Add loading progress bar on top of page when going to different page routes.
- [ ] **Bug**: Tower Intent Sketch only shows after pressing the back button and selecting "no" on leaving the tower with unsaved changes. (Unless the sketch takes longer to load? but it shouldn't since it is being read from dynamoDB)


## DOING ⚙️

- [ ] Fix display Errors and add special .catch statements checking MainAlert for anywhere we can add more error details for easier bug identification.
- [ ] Questions: 1. Do we use the `constraints` in `SectionIntent` for the min_thickness? Or should I add a separate function for this? **Note:** I cannot add them where they are specified in `compute.py` because the each section is an immutable tuple.


## Done

- [ ] Fix pdf reports bug - blank reports
- [ ] Change Last Modified on released towers to Released Date
- [ ] Remove revision numbers from released table expansion, as well as status.
- [ ] Add Revision number span next to Tower Drafts
- [ ] Remove expanded item subtitle
- [ ] Remove Edit icons and make tower name a blue hyperlink.


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