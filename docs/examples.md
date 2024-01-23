# User Guide

Let's go over the features of `work` with an example use case.

## Time tracking

Assume we start our workday and **enter the office at 10:00**.
We meet a colleague in the hall and **discuss our project until 10:30**.
Then, we sit down at our desk and start our computer.
We open a shell and type:

```
$ work start 10
Started work at 10:00 today

$ work switch 10:30 -c project -m "Discussion with colleague"
Switched work at 10:30 today (30 m recorded)
```

What has now happened is that we...

- ...added our first entry to the log from 10:00–10:30 with category "project" and a message.
- ...started a new work item at 10:30.

Assume that we also discussed with our colleague yesterday, but we forgot to track that in our log.
Furthermore, we worked the rest of the day on our project and developed new features.
To retroactively add the entries, we type:

```
$ work add --date yesterday 10 10:30 -c project -m "Discussion with colleague"
Added a record from 10:00 to 10:30 yesterday

$ work add --date yesterday 10:30 16 -c project -m "Development"
Added a record from 10:30 to 16:00 yesterday
```

Let's inspect what has now been logged:

```
$ work ls --since yesterday
Tue, 21.06.: 2 records
10:00 – 10:30 | 30 m      (project) "Discussion with colleague"
10:30 – 16:00 | 5 h 30 m  (project) "Development"
              = 6 h

Wed, 22.06.: 1 records
10:00 – 10:30 | 30 m  (project) "Discussion with colleague"
              = 30 m

Total: 3 records, 6 h 30 m worked
```

Notice:

- We have added two identical entries to the log, one with `work add`, and one with `work start` and `work switch`.
- We are currently working, but no entry has been logged yet.

We realize that we actually took a break before **starting to work at 10:45**.
To change the start time of the active run, we use `work start --force`.

```
$ work start 1045 --force
Started work at 10:45 today
```

Now, we want to check if this did what we expected and `work` is currently "active" and a run has been started at 10:45.
For that, we can do one of the following:

1. `work status` checks *if a run is currently active* and additionally *informs us about our workday* (expected hours, week balance).

    ```
    $ date                                            # assume it is 11:38
    Wed, 22. Jun 2022, 11:38:20 CEST

    $ work status                                     # check work status (at 11:38)
    Active since 10:45 today (0 h 53 m)
    > 1 h 23 m worked until now – 1 entry on record   # includes log entries + active run
    > Last entry ends at 10:30                        # last entry stored in log

    Workday status:
      6 h 7 m left (until 17:45)                     # time to complete expected hours
      Week balance: On time
    ```

2. `work list --include-active` (alias `ls`) lists logged entries but *includes the active run*.

    ```
    $ work ls --include-active
    Wed, 22.06.: 1 records (+ active run)
    10:00 – 10:30 | 30 m     (project) "Discussion with colleague"
    10:45 ~       | 0 h 53 m
                  = 1 h 23 m
    ```

3. Finally, we can use the special short-hand `work day`, which is equivalent to `work list --include-active --with-breaks --list-empty`.

    ```
    $ work day
    Wed, 22.06.: 1 records (+ active run)
    10:00 – 10:30 | 30 m     (project) "Discussion with colleague"
                  ~ 15 m break
    10:45 ~       | 0 h 53 m
                  = 1 h 23 m (+ 15 m of breaks)
    ```

## Analysis and modification

We want to look back at our work week and understand (1) how much we worked and (2) on which projects.
First, we check *how much we worked* with `work view balance`:

```
$ work view balance --week
Day      Date      Days    Expected    Worked       Balance graph
-----------------------------------------------------------------------
Mon      20.06.    1       8 h         8 h          |        |        |
Tue      21.06.    1       8 h         6 h          |      ==|        |
Wed      22.06.    1       8 h         30 m         |========|        |
Thu      23.06.    1       8 h         -            |========|        |
Fri      24.06.    1       8 h         -            |========|        |
Sat      25.06.    1       -           -            |        |        |
Sun      26.06.    1       -           -            |        |        |
-----------------------------------------------------------------------
Total              7       40 h        14 h 30 m    25 h 30 m remaining
```

As it is currently Wednesday, we have of course not logged any hours for Thursday and Friday yet.
Note that, on Tuesday, we have accumulated 2 hours of undertime.

Next, let's see exactly *what we worked on* this week with `work view by-category`:

```
$ work view by-category -w
category    hours        %      records
---------------------------------------
project     11 h 30 m    79%    4
social      3 h          21%    1
---------------------------------------
Total       14 h 30 m           5
```

This shows us that we have spent over 20 % of our time on social events - that can't be right!
We must have a mistake in our log.
Let's check by *only viewing entries with category "social"*:

```
$ work ls -w --Fc social
Mon, 20.06.: 1 records
10:00 – 13:00 | 3 h  (social)
              = 3 h

Total: 1 records, 3 h worked
```

Assume we made a mistake – we actually did not socialize from 10-13, but instead we were in a meeting from 11-13.
Let's fix our mistake and *force-add an entry to override the wrong log*:

```
$ work ls --day mon
Mon, 20.06.: 2 records
10:00 – 13:00 | 3 h  (social)
13:00 – 18:00 | 5 h  (project)
              = 8 h

$ work add --force --day mon 11 13 -c project -m "Meeting"
Existing record shortened (was 10:00 – 13:00): 10:00 – 11:00 (social) ""
Added a record from 11:00 to 13:00 on Monday, 20.06.2022

$ work ls --day mon
Mon, 20.06.: 3 records
10:00 – 11:00 | 1 h  (social)
11:00 – 13:00 | 2 h  (project) "Meeting"
13:00 – 18:00 | 5 h  (project)
              = 8 h
```

Now let's say this was not correct either.
There was no meeting, instead we worked the whole day on coding in our project.
Furthermore, we forgot to add a message denoting that.

Let's reword the message of both entries with the *interactive edit mode*:

```
$ work edit --day mon
Edit mode – on Monday, 20.06.2022

[0] 10:00 – 11:00 (social) ""
[1] 11:00 – 13:00 (project) "Meeting"
[2] 13:00 – 18:00 (project) ""

Enter nothing to cancel, or
Enter one or more indices [0..2] separated by a space, or
Enter "all" to edit all entries.

Which entries? > 1 2

 > Selected record: 11:00 – 13:00 (project) "Meeting"

# [...]
New message or remove ["-"]? (Meeting) Development
Change
  11:00 – 13:00 (project) "Meeting"
to
  11:00 – 13:00 (project) "Development"
? [Y/n]

Done

 > Selected record: 13:00 – 18:00 (project) ""

# [...]
New message or remove ["-"]? () Development
Change
  13:00 – 18:00 (project) ""
to
  13:00 – 18:00 (project) "Development"
? [Y/n]

Done
Info: Detected mergeable entries in protocol. Merging...
    11:00 – 13:00 (project) "Development"
  + 13:00 – 18:00 (project) "Development"
  = 11:00 – 18:00 (project) "Development"
Edited 2 records

$ work ls --day mon
Mon, 20.06.: 2 records
10:00 – 11:00 | 1 h  (social)
11:00 – 18:00 | 7 h  (project) "Development"
              = 8 h
```

Thereby, we could also see a second hidden feature: entries with the *same category and message* that are adjacent in the protocol are *automatically merged*.

## Overtime and undertime

In the previous examples, we can already see the *overtime calculation* at work.
Based on the configured *expected hours* to work, it calculates over- and undertime.
To configure the expected hours, simply edit the configuration file:

```fish
$ vim (work config --path)
```

One other case may arise – atypical free days such as holidays or vacations.
On such days, we do not expect to work, even if it is a regular day of the week.
To make sure that overtime and undertime are correctly calculated, we can add these as follows:

```
$ work free-days --add-holiday 21.
Added holiday on 21.06.2022

$ work free-days --add-vacation 24.
Added vacation on 24.06.2022
```

This also shows us that date parsing is *relative to the current day*, allowing us to just state "24." for "24.06.2022".

Now, let's verify this did what we expect it to. The easiest way to see expected hours is to use `work view balance`:

```
$ work view balance --week
Day      Date      Days    Expected    Worked       Balance graph
-----------------------------------------------------------------------
Mon      20.06.    1       8 h         8 h          |        |        |
Tue      21.06.    1       -           6 h          |        |======  |
Wed      22.06.    1       8 h         30 m         |========|        |
Thu      23.06.    1       8 h         -            |========|        |
Fri      24.06.    1       -           -            |        |        |
Sat      25.06.    1       -           -            |        |        |
Sun      26.06.    1       -           -            |        |        |
-----------------------------------------------------------------------
Total              7       24 h        14 h 30 m    9 h 30 m remaining
```

Looks good. As Tuesday was added as a holiday, our work then counts as overtime.
Friday is our vacation, meaning we only have 9 h 30 m remaining for this week.

Finally, let's *predict expected work end* with the `work hours` command.
We only have 9 h 30 m remaining this week, meaning if we work a total of 10 hours today, we are done for the week.
Let's not forget that we will take a 1 hour lunch break, though:

```
$ work hours --target 10 --pause 1
30 m on record, 53 m active run | Current balance: 37 m remaining (2 h to work today)

Assuming you take 1 h of breaks:
> Work until 21:15 for a 10 hour day. | Balance then: 8 h overtime
```

Seems reasonable – let's go full steam and start our vacation one day early this week!

