# Release History

## 1.1: Late night work support

Currently released version: `1.1.0` (2024-06-06)

### Breaking changes and deprecations

- Flag aliases `--date` and `-D` have been removed. Instead, use `--day` or `-d`.
- Migrations from versions < `0.100` have been removed.

### Runs and entries that cover midnight or extend beyond it

The headline feature of this release is that runs and entries can now end on midnight or extend beyond it (multi-day runs).
This concerns most features and leads to various changes, as described below.

#### Migration

Previously, entries could not end at midnight, only at 23:59 or earlier.
If you worked around this bug by saving entries with an end time of, e.g., 23:59 that actually ended at 24:00, you can make use of the automatic migration.

On first start, you will be offered to optionally migrate existing records.
Follow the interactive prompt to move the end time of all entries ending on or after a specified time to 24:00.
E.g., an entry `22:00 – 23:59` can be extended to midnight: `22:00 – 24:00`.

#### The active run can now extend beyond midnight

An active run started on the previous day is now not considered an invalid state any more.
Meaning, `status`, `list`, and `stop` work as expected:

```
$ work status  # assuming the current time is 02:13
Active since 22:00 yesterday (4 h 13 m)
> 4 h 13 m worked until now – nothing on record

Workweek status:
	Done (0 h 11 m over)

$ work list --since yesterday --include-active  # list yesterday and today, including the active run
Tue, 04.06.: 0 records (+ active run)
22:00 – 24:00 | 2 h  # this virtual "entry" will be highlighted in blue, as it's the active run
              = 2 h

Wed, 05.06.: 0 records (+ active run)
00:00 ~       | 02 h 13 m  # second part of the virtual "entry" for the active run
              = 02 h 13 m

Total: 2 records (+ active run), 4 h 13 m worked

$ work stop now-  # stop now, but round down
Stopped work at 02:00 today (4 h recorded)

$ work list -s y  # short-hand for --since yesterday
Tue, 04.06.: 1 records
22:00 – 24:00 | 2 h
              = 2 h

Wed, 05.06.: 1 records
00:00 – 02:00 | 2 h
              = 2 h

Total: 2 records, 4 h worked
```

Note that the *lingering run* heuristic (a check if a run that started the previous day was "forgotten") has been reworked.
It now triggers only if a run started on the previous day gets longer than 8 hours.

#### Log entries ending on midnight or extending beyond it

To allow `stop`ping at midnight or adding an entry that touches midnight, `24:00` is now understood and parsed correctly:

```
$ work status  # assume it is 23:50
Active since 22:00 (1 h 50 m)
> 7 h 50 m worked until now – 2 entries on record
> Last entry ends at 22:00
# [...]

$ work stop 24:00
Stopped work at 00:00 tomorrow (2 h recorded)  # let me know if you want this to read "24:00 today" ;)
```

When `add`ing entries, you can now also enter multi-day entries (extending beyond midnight).
E.g., to add a run from 22:00 yesterday to 02:30 today:

```
$ work add -1 22 26:30
Added a record from 22:00 yesterday to 02:30 today
```

You may enter any "hour" up to `29`. These are considered as relative to the start time (as is common in Japan), so `25:05` represents `01:05` on the following day.

Note that, when `edit`ing, you may only enter `24:00` as an end time – but no later time.

### Even more robust handling of overlapping free days

`free-days` now deals with overlaps in all scenarios when adding a vacation.

1. Also checks for configured holidays and offers to remove those dates from the added vacation, just like non-working days (see [release notes of `0.101`](#more-robust-handling-of-overlapping-free-days)).
2. If the vacation overlaps a reduced hour day, a clear error is printed before exiting.

### Other features

- You can now force stopping with `stop --force` and `switch --force` if the added entry would overlap a stored one.
- More human-readable error messages
	+ Errors raised when trying to add overlapping entries will now always be nicely formatted, even in corner cases.
	+ `list` now detects if the active run overlaps with a stored entry and prints an according error message.

### Fixed bugs

- `edit` now behaves as expected in dry run mode: by not performing any modifications to the log.

### Internals

- Debug switches are now set with environment variables.


## 1.0: New name on PyPI and the (symbolic) "version 1"

Final release version: `1.0.3` (2023-08-23)

Finally, you can `pipx install work`!

Since Dan Colish, who owned the name `work` on PyPI, was so kind to transfer the name, the project has moved to this new name.
It is recommended to uninstall `work-time-log` and do a clean install of `work` – your user data survives reinstalls:

- **If you used `pipx`**: `pipx uninstall work-time-log` and `pipx install work`
- **If you used `pip`**: `pip uninstall work-time-log` and `pip install --user work`

### Fixed bugs

- `stop`
	+ If the stopping time equals the active start, the run is now again cancelled.
	+ The run length that *would* be recorded is now correctly printed for dry runs.
- `list --include-active`: Now detects if the listed active run would overlap a stored entry and prints an appropriate error message.

## 0.101: Command-line comforts

Final release version: `0.101.1` (2023-07-21)

### Breaking changes and deprecations

- Filtering is now case insensitive by default (see *Smart case*).
- Date selection flags `--date` and `-D` have been deprecated (see *Day names and dates...*).

### Skip interactive selection (or select quicker)

For `edit` and `remove`, users are presented a list of entries that allows them to interactively select what they want to edit or remove concretely. E.g.:

```
$ work edit -1  # edit yesterday's record
Edit mode – Wednesday, 19.07.2023

[0] 08:00 – 12:00 () "Various tasks"
[1] 12:00 – 14:00 (project-B) "Lunch meeting"

Enter nothing to cancel, or
Enter one or more indices [0..1] separated by a space, or
Enter "all" to edit all entries, or "last" for the last.

Which entries? >
```

One small change can be seen immediately: the new keyword `"last"` allows easy selection of the last entry added that day.

More importantly, though, two new flags were added to both `edit` and `remove`: `--all` and `--last`. They allow skipping the interactive selection completely.

Now, remove all entries recorded on, e.g., Monday with just one command:

```
$ work remove --day mon --all
Remove mode – Monday, 17.07.2023

[0] 12:00 – 13:00 (project-A) ""

Interactive selection skipped – selecting "all".
Removed 1 record
```

Or, perhaps even more useful, directly edit the last record added today:

```
$ work edit --last
Edit mode – Thursday, 20.07.2023

[0] 12:00 – 14:00 () ""

Interactive selection skipped – selecting "last".

 > Selected record: 12:00 – 14:00 () ""

New start time? (12:00)
# ... etc.
```

### Filter with *smart case*

Filtering used to be case sensitive. In most cases, though, we do not need case sensitivity. To preserve the option to use case sensitive search, while keeping the command line interface clean, `work` now employs *smart case* (popularized by [fd](https://github.com/sharkdp/fd)).

In short:

- If the search string is *fully lowercase*, filtering is *case insensitive*.
- If it contains *at least one capital letter*, it will switch to *case sensitive* filtering.

E.g.:

```
$ work ls --since 1.1. --Fm "meet*"
Tue, 21.03.: 1 record
11:30 – 12:30 | 1 h  (project-A) "Meeting"
              = 1 h

Mon, 22.05.: 2 records
10:30 – 10:45 | 15 m  (project-A)       "meeting with josef"
11:30 – 11:45 | 15 m  (project-B/task1) "Meet and Greet"
              = 30 m

Total: 3 records, 1 h 30 m worked

$ work ls --since 1.1. --Fc "*[AC]*" --Fm "Meet*"
Tue, 21.03.: 1 record
11:30 – 12:30 | 1 h  (project-A) "Meeting"
              = 1 h

Total: 1 record, 1 h worked
```

Note that, as seen in the examples above, filtering supports glob patterns such as `[AC]` matching either `"A"` or `"C"`, or `*` that matches everything. [Read more](https://docs.python.org/3/library/fnmatch.html)

### Day names and dates can now be used interchangeably

- The parsing of day names (e.g., "monday") and dates (e.g., "12.") were merged.
- Most visible impact: the flags `-d/--date` and `-D/--day` have been merged into one (`-d/--day`).
	+ The combined flag supports the combination of all valid inputs.
	+ E.g., try `--date yesterday`, `-d tue`, or `-d 15.`.
	+ The short flag `-D` and the long flag `--date` are deprecated and will be removed in a future release.
- Small but impactful effect: Most date-dependent flags (e.g., `--period`, `--since`) that previously understood only dates now also support day names.
	+ E.g., try `list --since sat` or `free-days --add-vacation mon wed`
	+ Day names and dates can be mixed; e.g., `--period mon 5.` works.

### More robust handling of overlapping free days

`free-days` now deals with overlaps more robustly. Two features were added:

1. Adding a vacation now checks for configured non-working days and can remove those dates from the added vacation.
2. Adding any free day now cancels if the chosen date already has a free day stored.

#### Removing non-working days before adding a vacation

In most cases, the configuration of "expected hours" will be set to expect no work on weekends. Therefore, Saturday and Sunday will be considered "non-working days".

When adding a vacation that is longer than a week, though, the vacation period will overlap with a weekend. If `work` is used to track the number of taken vacation days, this is unwanted behavior, as weekends do not count towards the vacation day limit.

Therefore, when adding a vacation and if it overlaps with non-working days, the user will be prompted and can choose to have them removed:

```
$ work free-days --add-vacation 31.12.22 5.1.
The vacation overlaps with configured non-working day(s): 01.01.2023 (Sunday)
Should non-working days be removed from the vacation before it is added?
[Y/n] y
Added vacation from 31.12.2022 to 05.01.2023
```

This is optional, as the behavior may not always be intended.

#### Preventing double entry of free days

The three free day categories "vacation", "holidays", and "reduced hour days" are mutually exclusive by definition. To preempt user errors that require a manual fix, adding any type of free day on a date that already has a free day stored will now be prevented.

### Other features

- New flag `hours --for-balance` calculates the end time for a target balance; e.g. `h -b 2+` for 2 h overtime.

### Fixed bugs

- `view balance` now handles multi-year weeks correctly as single, continuous weeks.
- `edit`
	+ Does not allow entries to have 0 m length after editing any more.
	+ The date is now not printed with an "on" before it any more.
	+ Edited entries can no longer overlap the active run (the reverse was already impossible).
- Time parsing now detects the mode correctly also if an alias is used.


## 0.100: Aliases and macros

Final release version: `0.100.4` (2023-07-05)

### Breaking changes

- Configuration file location changed (see *User configuration*)
- Some command line flags of `config` removed (see *Other changes*)

### Configurable aliases and macros

User-defined *aliases* and *macros* can now be added via the configuration, for example:

```json
{
	"aliases": {
		"start":  ["1"],
		"stop":   ["0"],
		"status": ["s", "st"]
	},
	"macros": {
		"s1": "status --oneline"
	}
}
```

#### Aliases

An *alias* is a *short name for a mode*, e.g. `ls` for `list`. Running `work --help` will show any configured aliases. For the example above, this would look as follows (shortened):

```
$ work --help
...

modes:
  {start, stop, add, status, list, ...}
    start (1)           Start work
    stop (0)            Stop work
    ...
    status (s, st)      Print the current status
    ...

```

They can be used in place of the command they replace:

```
$ work 1
Started work at 23:15 today
```

Note that the hardcoded aliases defined in previous versions were removed and instead added to the default configuration.

By configuring custom aliases, the defaults can be overwritten.

#### Macros

A *macro* is an *arbitrary expansion*, e.g. `vbw` could be configured to mean `view balance --week`. When a macro is used, it is replaced in the command line with the configured expansion before parsing the arguments, e.g.:

```
$ work s1  # "s1" -> "status --oneline" (see above)
Inactive
```

*Note: Macros and the mode argument cannot be combined on the command line. E.g., `status s1` would be invalid. That means a macro must always include a mode in its expansion, as seen above.*

### User configuration

#### New file location

- The location is now determined based on the [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html)
- In short, the file is stored in `$XDG_CONFIG_HOME` if that is defined, otherwise in `$HOME/.config`
- With that change, the file name has now also been changed from `.workrc` to `workrc`

#### Keys may now be left unset

- Configuration keys can now be omitted
- If left unset, the default value is used instead
- To check the default values, use `config --default`

*Note that for mappings (such as `"expected hours"`), only the complete mapping can be overwritten in the user configuration, not just a single key.*

### Magic arguments to coerce rounding

When using the keyword `now`, the current time is rounded either up or down, depending on the command.

For example, assuming it is 12:10:
```
$ work start now
Started work at 12:00 today  # rounded down
```

Now, the rounding can be influenced with the new keywords `now+`, `now-`, and `now!`. By using them, one can force rounding up, rounding down, or disable rounding, respectively.

Compare to above:

```
$ work start now!
Started work at 12:10 today  # not rounded
```

### Improved listing of free days

`free-days --list` has been made more useful.

1. Now also outputs the number of days in each category and in total.
2. Continuous periods in the output of vacation days are merged.

For example:

```
$ work free-days --list
Holidays (2 days):
  01.01.2022
  10.01.2022

Vacations (13 days):
  10.02 – 11.02.2022 (2 days)
  21.05 – 31.05.2022 (11 days)

Total: 15 free days
```

### Other changes

- `config`:
	+ Short flags `-c`, `-e`, and `-s` removed.
	+ Flag `--expected` renamed to `--default`.
	+ Flag `--create` removed (instead, use `work config --default > <file_path>`).
	+ `--see` argument `expected hours` renamed to `expected-hours`.
- `--day` now excludes the current day when resolving the weekday.
- `--period` no longer sorts the provided start and end date.
- Mode `day` removed (supplanted by macro functionality).
- Improved messages in case of verification errors.
- All exceptions, including those raised before argument parsing, are now handled gracefully.
- Migrations:
	+ Moves configuration file to new location.
	+ Converts existing files on Windows to UTF-8 encoding and `\n` line terminator.
- Brief "what's new" message after a version upgrade.

### Fixed bugs

- `list --include-active` now layouts correctly if the log is empty, but a run is active.
- `hours` now says `no active run` instead of `0 m active run` if no run is active.
- The plural form is now only used when a message refers to zero or multiple elements.
- `edit` now always sorts selected indices in ascending order before starting the edit.
- Fixes alias handling triggering a key error if no aliases have been configured.
- Fixes that if the log consisted of just one entry, and that entry was deleted, the integrity check would fail.

### Internals

- Files created on Windows are now encoded in UTF-8 and use the `\n` line terminator, as on Linux.
- In debug mode, a separate flags state is now used. The configuration file is intentionally shared, though.


## 0.99: Getting a better view

Final release version: `0.99.2.post1` (2022-06-28)

### New mode `view`

- Until now, the only way to view entries was grouped by date (with `list`).
- The new mode `view` allows other views on entries.
- This version adds the `by-category` and `balance` view.

#### `view by-category`

- View entries grouped by their category.
- Use cases:
	+ Quickly view categories used in the past in a compact form.
	+ Check how much work was done for a specific project (category), e.g. in the current week.

For example:

```
$ work view by-category --since 1.2.
category            hours        %      records
-----------------------------------------------
project-A/coding    16 h 30 m    40%    17
meetings            9 h          21%    5
∅                   7 h          17%    19
project-B/coding    4 h 30 m     10%    7
project-B/slides    3 h 30 m     8%     4
help                30 m         1%     1
-----------------------------------------------
Total               41 h                76
```

Above, `∅` is used to represent entries without a category.

We could combine this with a filter (more on the new filters below) to check how much we worked for `project-B`:

```
$ work view by-category --since 1.2. --Fc "project-B*"
category            hours        %      records
-----------------------------------------------
project-B/coding    4 h 30 m     56%    7
project-B/slides    3 h 30 m     44%    4
-----------------------------------------------
Total               8 h                 11
```

#### `view balance`

- View the balance, and how it developed, over multiple weeks or longer periods.
- Until now, the balance was only tracked over the current week. This view allows arbitrary periods for balance calculation.
- Use case: Check for accumulated overtime over a longer period of time, e.g. the past year or even since you started tracking your work.

For example, assuming it is Tuesday, 2022-02-22:

```
$ work view balance --week
Day      Date      Days    Expected    Worked       Balance graph
-----------------------------------------------------------------------
Mon      21.02.    1       8 h         8 h          |        |        |
Tue      22.02.    1       8 h         5 h 45 m     |      ==|        |
Wed      23.02.    1       8 h         -            |========|        |
Thu      24.02.    1       8 h         -            |========|        |
Fri      25.02.    1       8 h         -            |========|        |
Sat      26.02.    1       -           -            |        |        |
Sun      27.02.    1       -           -            |        |        |
-----------------------------------------------------------------------
Total              7       40 h        13 h 45 m    26 h 15 m remaining
```

Alternatively, we can take a look back to previous weeks, e.g.:

```
$ work view balance --period 3.1. 20.2.
Week     Date range         Days    Expected    Worked       Balance graph
--------------------------------------------------------------------------------
w. 01    03.01. – 09.01.    7       32 h        32 h         |        |        |
w. 02    10.01. – 16.01.    7       40 h        40 h         |        |        |
w. 03    17.01. – 23.01.    7       40 h        40 h         |        |        |
w. 04    24.01. – 30.01.    7       40 h        40 h         |        |        |
w. 05    31.01. – 06.02.    7       40 h        37 h 30 m    |       =|        |
w. 06    07.02. – 13.02.    7       40 h        42 h 30 m    |        |=       |
w. 07    14.02. – 20.02.    7       40 h        45 h         |        |=       |
--------------------------------------------------------------------------------
Total                       49      272 h       277 h        5 h overtime
```

Longer periods are grouped by month:

```
$ work view balance --period 1.9.21 31.12.2021
Month    Date range             Days    Expected    Worked       Balance graph
-------------------------------------------------------------------------------------
Sep      01.09.21 – 30.09.21    30      176 h       177 h        |        |        |
Oct      01.10.21 – 31.10.21    31      168 h       168 h        |        |        |
Nov      01.11.21 – 30.11.21    30      176 h       180 h        |        |        |
Dec      01.12.21 – 31.12.21    31      144 h       100 h        |      ==|        |
-------------------------------------------------------------------------------------
Total                           122     664 h       625 h        39 h remaining
```

### Filtering for `list` (and `view`)

- Beyond selecting the date (range), records can now optionally be filtered based on both category or message.
- New flags: `--filter-category` / `--Fc` and `--filter-message` / `--Fm`
- By default, the complete category or message must match, but glob patterns (with `*`) are supported.

For example:

```
$ work ls
Tue, 22.02.: 3 records
10:00 – 12:00 | 2 h   (project-B/slides)   "Slides for customer meeting on 2022-03-01"
12:00 – 12:30 | 3 h   (project-A/coding)
14:00 – 15:00 | 45 m                       "Cleaned office"
              = 5 h 45 m

$ work ls --Fc "project*"
Tue, 22.02.: 2 records
10:00 – 12:00 | 2 h   (project-B/slides)   "Slides for customer meeting on 2022-03-01"
12:00 – 12:30 | 3 h   (project-A/coding)
              = 5 h

$ work ls --Fc "project*" --Fm "*customer*"
Tue, 22.02.: 1 record
10:00 – 12:00 | 2 h   (project-B/slides)   "Slides for customer meeting on 2022-03-01"
              = 2 h
```

### Expanded date selection

- `add`, `edit`, and `remove` now also support `--yesterday` / `-1` and `--day`
- For `list`, `view`, and `export`:
	+ New `--since` selector equivalent to `--period X today`
	+ The `--month` selector now allows (optionally) passing a DATE, to select a month by one of its dates. E.g., `--month 1.1.` selects January of the current year.

### More robust `edit`ing

- The improved handling for multi-record editing introduced in v0.97 has been finalized.
- Most noticeable will be that now multiple entries can be edited in such a way that each individual edit would be invalid, but the final state is valid again. E.g., two records (`10–12` and `12–13`) can be shifted at once (to `10–12:30` and `12:30–13`).
- Also, if an individual edit operation encounters an error, the complete edit will be rolled back safely.

### Changes

- `--day` no longer accepts ambiguous prefixes such as "s".
- `--help`:
	+ Main program now has a "teaser message".
	+ Does not mention "protocol" any more.
- If an invalid state is detected but fixed by the user, the originally requested command is now retried automatically.
- Output:
	+ `hours`: Both `--until` and `--target` / `-8` now print the projected balance at the assumed end time.
	+ When stopping or switching, the recorded run length will be printed out, e.g.: `Stopped work at 17:30 today (30 m recorded)`
	+ When an invalid state is detected, the user will no longer be offered to "abort"; instead use Ctrl-C.
	+ `config --path` now only prints the file path, enabling `subl (work config --path)`.
- CLI:
	+ `switch`: The alias `pause` will be deprecated in an upcoming version; a warning is printed if it's used.
	+ `recess`: The primary name has been changed to `free-days`. The alias `recess` will remain as an option for now.
	+ The short flag for `--version` is now `-V`.
- This version removes the migration code for the info file introduced in v0.94.

### Fixed bugs

- `status`:
	+ Note "reduced hour day" no longer printed just because expected hours are less than 8.
	+ Note "work until" now incorporates start time of active run in calculation of end time.
- `switch`: Now checks if the switch time equals the start time of the active run (meaning the run would simply cancelled and immediately restarted at the same time) and prints a warning instead.
- `edit`: Invalid changes no longer lead to deletion of the last edited record.


## 0.98: Improved selection interface and fine-grained `hours` calculation

Final release version: `0.98.1` (2021-12-18)

### New `edit` and `remove` selection interface

- A new selection interface to improve readability (inspired by `fish`'s `history` tool).
- Now allows selection of multiple indices at once.
- By entering nothing, the selection can be immediately cancelled.

For example:

```
$ work add 10 14 -c supre
$ work add 15 17:5 -m "ji"
$ work edit
Edit mode – today

[0] 10:00 – 14:00 (supre) ""
[1] 15:00 – 17:05 () "ji"

Enter nothing to cancel, or
Enter one or more indices [0..1] separated by a space, or
Enter "all" to edit all entries.

Which entries? > 
```

### New flags for `hours` calculation

- New flag `--start` to override the assumed start time.
- New flag `--pause` to include planned breaks in the calculation.
- Assumptions for the calculations are only printed once if multiple modes are selected.
- Flag `--balance` removed – the balance is now always printed.

For example (assume it is 14:17):

```
$ work start 12
Started work at 12:00 today

$ work hours --until 17
0 m on record, 2 h 17 m active run | Balance: 5 h 43 m remaining (8 h to work today)
> You will have worked 5 h at target time 17:00.

$ work cancel
Run cancelled

$ work hours --until 17
0 m on record, 0 m active run | Balance: 8 h remaining (8 h to work today)

Assuming you start now (14:15):
> You will have worked 2 h 45 m at target time 17:00.

$ work add 10 12
Added a record from 10:00 to 12:00 today

$ work hours --until 17 --target 6 --start 14 --pause 0:30
2 h on record, 0 m active run | Balance: 6 h remaining (8 h to work today)

Assuming you start at 14:00 and take 30 m of breaks:
> You will have worked 4 h 30 m at target time 17:00.
> Work until 18:30 for a 6:00 hour day.
```

### Changes

- Corrupted files will now be detected more robustly while parsing. Specifically this covers the case that a file is formatted validly, but the contents violate assumed invariants.

### Fixed bugs

- `recess`: If a vacation was added over multiple years, the days were added to both years.
- `hours`: If a day started with overtime, an exception was raised.


## 0.97: Force actions and edit better

Final release version: `0.97.4` (released 2021-08-17)

### Force `start`, `resume`, `add`

A new `--force` flag allows intentionally overriding some warnings:

- `start --force`: Start even if a run is already active.
- `resume --force`: Resume even if a new run was already started (undo `switch`).
- `add --force`: Insert entries forcefully.
	+ If an entry is *subsumed*, remove and replace it.
	+ If an entry is *overlapped*, shorten it to make space.
	+ If an entry *contains* the forced entry, split it up and shorten the splits to make space in-between.

For example:

```
$ work ls
Fri, 28.05.: 1 records
10:00 – 18:00 | 8 h  (original)
              = 8 h

$ work add 14 15 -c split --force
Existing record split up: 10:00 – 18:00 (original) ""
  ➜  10:00 – 14:00  &  15:00 – 18:00
Added a record from 14:00 to 15:00 today

$ work ls
Fri, 28.05.: 3 records
10:00 – 14:00 | 4 h  (original)
14:00 – 15:00 | 1 h  (split)
15:00 – 18:00 | 3 h  (original)
              = 8 h
```

### New `edit` handling

- When editing `"all"`, edits are only applied after all entries were iterated to allow cancelling any time and prevent some merge conflicts.
- Category and message can now be removed when editing an entry by entering `"-"` as their new value.
- Entering nothing and just pressing Enter now accepts a change, as is suggested by `[Y/n]` in the message.

### `list` output improvements

- `--include-active` now counts active run in total
- `--only-time` now merges touching entries for output
- `--with-breaks`:
	+ now shows breaks in separate lines
	+ now omits 0 minute breaks from the output.

### Changes

- `switch` now only allows one positional argument. To add a break before restarting, use the optional argument `--start H:M`.
- The `--week` selector now allows an optional argument for the week number. If none is given, the current week is assumed.
- `hours --target` now expects a string formatted as "H:M" instead of a float.
- Entries with empty category and message are now printed as `<start> – <end> () ""`.
- `recess`: Clearer error message when removing nonexistent day
- completions: Global flags no longer suggested after sub-commands
- packaging: Test modules now omitted from package

### Known bugs

- When editing `"all"` and changing an entry so that it would be merged with its *next* neighbor, an exception occurs.


## 0.96: Expected hour management

Final release version: `0.96.3`

### Expected hour management

- New module `recess`
	+ Configure holidays, vacations, and days with reduced expected hours.
	+ The expected hours for the week and day will update accordingly.

For example:

```shell
$ work recess --add-vacation 1.2. 3.2.
$ work recess --add-reduced-day 4.2. 2.0
$ work recess --list
Vacation:
  01.02.2021
  02.02.2021
  03.02.2021
Reduced hour days:
  04.02.2021 (2.0 hours)
```

- Expected hours per weekday can now be configured in the RC file.

For example:

```json
{
  "expected_hours": {
    "Monday":    8.0,
    "Tuesday":   8.0,
    "Wednesday": 4.0,
    "Thursday":  8.0,
    "Friday":    8.0,
    "Saturday":  2.0,
    "Sunday":    0.0
  }
}
```

Check your configuration with `work config --see "expected hours"`.

### Calculation of presumed work time with `hours`

- `--until` now handles differently depending on if a run is active.
- Fixes hour calculation for runs starting in the future (`--target` and `--workday`).
- Fixes presumed start time in `--until` if no run is active.

### Changes

- Output messages for `switch` are now more abstract and intuitive.
-  `work config --expected` now prints the default configuration that is used by `work config --create`.
- **Breaking**
	+ Old configuration files are now invalid, due to added (see above) and removed (see below) keys. Create a valid configuration file with `work config --create`.
	+ Autopause functionality removed, including the configuration option


## 0.95: export and CLI convenience

- New mode `export` that enables exporting entries of any date or date range as CSV.
- `start` now understands the keyword `again`, enabling easy back-to-back entries after a run was stopped.
- Time parsing now understands military time, e.g. `1400` = 14:00.
- If a corrupted record file is detected on disk, its path is now printed out.

### Fixes

- Handles Ctrl-C more cleanly.
- Fixes bug where entering nothing in `rehash` lead to a rehash.
- Fixes bug where `list --include-active` would crash if an active run starts in the future.


## 0.94: New verification algorithm

- Verification algorithm changed: Replaced MD5 hash with a much faster Adler-32 checksum.
	+ As the hash was only meant to protect against accidental edits, priorities were shifted to increase the speed.
	+ Relative speed improvement: About 20–50 % faster, more with growing size.
	+ Absolute improvement: About 1 ms for large sets of records (5 years * 200 days * 10 records).
- Empty folders for years and months in the records directory will be removed now.
- This version removes the migration code introduced in v0.93. Make sure to migrate your records before updating.

With this update, your info file needs to be updated once. Please run `work rehash` to do that.


## 0.93: Performance improvements

- Alias "pause" for `switch`
- Internal structural changes that make listing a huge number of days quicker.
- Start-up checks slimmed down to reduce start-up time.
- Adds verification functionality for traversed record directories; if an unexpected file name or type is encountered, an error is raised.

Due to a bug in previous versions, not all entries were migrated from protocol v2 to protocol v3. It is recommended to run `work ls --period 1.1.1900 31.12.2100` once (this traverses all entries in the protocol). This will fix any leftover entries automatically.


## 0.92: switch

- New `switch` function
- The `pause` function was deprecated in favor of the new `switch` function.
- The old functionality "stop at time A, restart at time B" has been preserved.
- New: only one time argument can be passed.
	+ This enables a "switch in place" functionality.
	+ When a new task is started, this command suffices to save the completed work and immediately start a new task.


## 0.9: Category and message

Final release version: `0.91`

- Entries can now have an optional category and message.
- Both can be added when stopping a run or adding an entry.
- When listing entries, these fields can be displayed.
