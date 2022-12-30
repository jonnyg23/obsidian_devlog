---

kanban-plugin: basic

---

## TODO 💭

- [ ] 1. Revamp UI to use report notes on tower intent, design basis, and loadset pages. Use test_report.py::test_report_notes as an example for how to format each variable.
- [ ] Getting cross weld lengths in bounds
- [ ] Add tower status to LibraryCrud.check_references


## DOING ⚙️

- [ ] 1. Work on certification report & add report_notes to LoadSet<br/> <br/> TODO: Fix `LoadSet.Merge` in towerdesign to allow for merging `report_notes`


## Done

- [ ] Add ReportNotes schema & class to towerdesign to be used for Design Basis. Includes vars such as `blueprint_notes` and `certification_notes`
- [ ] After section notes PR gets merged, update **bluprints.py** to pull section_notes not from kwargs but from `tower_intent.report_notes.blueprint_notes.section_notes` list
- [ ] Tox fails because of Phash comparisons & schema assertions after adding `report_notes`. Test failures shown below:
- [ ] FAILED tests/test_structural_phash.py::test_phash_design_basis - AssertionError: assert '49031fa9b5c4a2b8' == 'f6c58972f78380ec'
- [ ] FAILED tests/test_schema_structural.py::test_pampa_design_basis - AssertionError: assert {'notes': ['IEC 61400-1 3rd Edition', 'Buckling design according...
- [ ] FAILED tests/test_schema_structural.py::test_design_basis - AssertionError
- [ ] FAILED tests/test_loads_phash.py::test_hash_load_set - AssertionError: assert '7c34c88b71853d87' == '7076f678411576b5'


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