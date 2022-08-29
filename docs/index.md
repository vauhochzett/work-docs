# work

`work` allows manual time tracking with an interaction model inspired by `git`.

---

**Documentation**: this page

**PyPI package**: [https://pypi.org/project/work-time-log/](https://pypi.org/project/work-time-log/)

---

## Key concepts

1. Human-readable **text files for storage**. This makes it easy to track the log with `git` or perform manual edits.
2. The tool **does not run continously** and instead only track the current status on disk.
3. **Global work status**, meaning any terminal can be used to check or update it.

## Features

- **Time tracking**
    + Time track while working and (optionally) add a category and message.
    + Retroactively add and modify any entry.
- **Analyses**
    + Calculate and check the hours worked over arbitrary periods.
    + List tracked entries by date or category with optional filters.
- **Overtime and undertime**
    + Configure "expected hours" and view the accumulated over-/undertime.
    + (Optionally) store vacations or holidays.
- **Export entries as CSV**

## Installation

The installation with [pipx](https://pypa.github.io/pipx/) is recommended. For that, simply run:

```
$ pipx install work-time-log
```

Of course, installation with `pip` works just as well.

## Getting Started

Find a list of example use cases on the page [Examples](examples.md).

Furthermore, check out the help pages. `work --help` shows an overview, and `work <cmd> help` shows help for any sub-command.
