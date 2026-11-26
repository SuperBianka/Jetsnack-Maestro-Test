# Jetsnack: Automated test with Maestro (Part 5)

This folder contains one automated UI test for the Jetsnack Android app, written with [Maestro](https://maestro.mobile.dev).

| Item | Value |
|---|---|
| Scenario | TC-01 (High): Verify that the "Filters" modal opens from the "Filters" icon on the "Search" screen |
| Flow file | `flows/tc01_open_filters_from_search.yaml` |
| App under test | Jetsnack, commit `09aae2013b3e5897f9679e2b7539f505f4f6f7f6`, package `com.example.jetsnack` |
| Tested with | Maestro 2.12.1, Android emulator `Medium_Phone_API_37.0` (Android 17) |

## What the test does?

The flow follows test case TC-01 from `QA_Mobile_Take-Home_Assignment.md` word for word.

- **Preconditions:** App is launched. The "Home" screen is shown.
- **Steps:**
  1. Tap the "Search" icon in the bottom navigation bar.
  2. Tap the "Filters" icon on the "Search" screen.
- **Expected result:** The "Filters" modal opens. It shows the title "Filters", the "Close" button and the sections "Sort", "Price", "Category" and "LifeStyle".

The flow starts with cleared app data, so the app always starts on the "Home" screen.

## Expected result of the run on this build

**The test fails at step 2.** This is on purpose and it is the correct result: on this build the "Search" screen has no "Filters" icon (BUG-02 in my bug reports). The icon exists only on the "Home" screen. The saved output is in `results/tc01_run_output.txt`:

```
Tap on "Filters"... FAILED
Element not found: Text matching regex: Filters
```

When BUG-02 is fixed, the same file should pass with no changes. To check that all the other steps and checks are correct, I ran a copy of the flow that opens "Filters" from the "Home" screen instead. Every command in that copy passed.

## How to run?

Requirements: Java 17 or newer, the [Maestro CLI](https://docs.maestro.dev/getting-started/installing-maestro), Android SDK platform tools (`adb`), and an Android emulator or a USB device with USB debugging.

1. Start an emulator or connect a device. `adb devices` must list it.
2. Build and install the app from the `Jetsnack` folder:
   ```
   ./gradlew installDebug
   ```
3. Run the test from the `Jetsnack` folder:
   ```
   maestro test automation/flows/tc01_open_filters_from_search.yaml
   ```
   On Windows in Git Bash, use `maestro.bat` instead of `maestro`.

Optional: set the environment variable `MAESTRO_CLI_NO_ANALYTICS=1` before running if you do not want Maestro to send usage data.

## Why I chose this scenario?

- **It is the entry point of the feature.** Every other Filters test starts by opening the modal, so if this one fails, the rest cannot run.
- **It is a smoke test.** It is quick, has no random data, and needs no test account, so it is safe to run after every build.
- **It is repetitive manual work.** It checks the same screen and the same sections each time.
- **It has already found a real problem.** It fails on this build for a real reason (BUG-02), so it shows that the automation catches regressions, not only that it runs.
- **It uses stable selectors.** Maestro finds elements by visible text and accessibility labels ("Search", "Filters", "Close"), and these do not change with layout or screen size.

## What I learned?

- Maestro finds elements by visible text and by content descriptions, so the Compose app could be tested without any code change or test IDs. The "Search" tab and the "Filters" icon are content descriptions (the tab is "SEARCH" in the app, and Maestro matched "Search" regardless of capital letters), and they worked as selectors.
- A failing step gives a clear message ("Element not found: Text matching regex: Filters"), which makes it easy to see whether it is a real bug or a wrong selector.
- `launchApp` with `clearState: true` gives the same starting point every time, which removes problems with leftover filter state.
- The setup took longer than the test: Maestro is a large download, it needs Java, and on Windows it must be started as `maestro.bat` from Git Bash.
- It is useful to prove the later checks on a version that passes (opening "Filters" from "Home"), because a test that stops at its first failure does not show whether the remaining lines are correct.

## What I would automate next?

1. **Filtering by one option** (TC-02, TC-04, TC-05): Select one chip and check that only matching products remain. It would fail today because of BUG-01, so it would protect the fix.
2. **Closing without applying** (TC-10 and TC-11): Select chips, use the "Close" button or the Maestro `back` command, and check that nothing changed. This would catch BUG-03 as well.
3. **Empty state** (TC-12): Choose a combination with no products and check the message.
4. **One price range at a time** (TC-14): Tap two price chips and check that only one is selected.
5. **Run in CI** on every pull request with an emulator, and on a small-screen and a large-screen profile.
6. **Ask developers for test tags** (`testTag`) on the chips, so tests do not depend on visible text if the wording or the language changes.
