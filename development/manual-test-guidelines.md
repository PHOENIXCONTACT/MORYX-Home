# Manual Test Guidelines

## Testing Strategy

### What should you test manually?

The term *Full User Interaction* is referred to several times in this document. This term refers to a series of actions a user takes with the intend to execute a certain functionality on the UI.

- [x] The user navigates to a page and presses, selects an item, changes a value and saves it. (Intent: Editing the Item)
- [ ] The user navigates to a page (No intend)
**That is, you should not make tests for things which are not in itself a feature or for different cases of the same feature**

### When should you write a Test?

1. After each Story/Task that creates a new feature for the user to use in the UI
2. After each Story/Task/Bug that changes/fixes the UI, if there is no existing test by which the change **might** be covered. (No new tests if e.g. a bug only appears with a certain screen size, but is otherwise already covered in any full user interaction)

### When should you update Tests?

1. If a change you made makes a test incorrect or the actions specified for it unexecutable.
2. If a certain response of the UI is expected but not mentioned in the expected behaviour.

### When should you execute Tests?

1. After each MR (or a set of MRs in the same iteration as you see fit), you execute the module specific tests on a local instance of the demo. (Afterwards push the demo with the new package version)
2. (Try once:) On Friday after the daily every dev gets a module to test (which they haven't worked on in the iteration) with the intend to break it

**Additional Remark:** Document time it took you to test a module (to have indications where to start with automated testing) at the bottom of the 0_index.md of your module.

## Style

For the style look at the [template](./tests/manual_test_template.md).

## Folder and File structure

Files for manual tests are saved in the project repository in the folder `docs\manualTests\<ModuleName>`. Each TestCase must have its own file. To give a better overview, you should numerate the files.

This is how the folder structure of the media module look

```folder structure
communication
repository-folder
└───docs
    └───manualTests
        └───module1
            └─── 0_index.md
            └─── 1_test1.md
            └─── 2_test2.md
        └───module2
            └─── 0_index.md
            └─── 1_test1.md
```

Each module should have one overview over all test cases called `0_index.md`. This file also contains the common preconditions for all test cases. It could look like this

| No. | Description                   | Automated | Comment   |
|-----|-------------------------------|-----------|-----------|
| 1   | Upload a media file           |           |           |
| 2   | Add a variant to a media file |           |           |
| 3   | Delete and save a Variant     |           |           |
| 4   | Delete a media file           |           |           |

There are no preconditions.
