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
- [ ] Questions: <br>1. Will model_number use a validation schema?<br>2. How do you want me to test model_number? I don't have the ability to delete a model_number in the back end so should I add one and delete it manually from dynamodb even though it doesn't have a validation schema?
- [ ] ⚠️ Look into the new Turbine and new Design Basis buttons since they don't pop up the create new component modal.


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