![Version](https://img.shields.io/static/v1?label=org-weekly-schedule&message=0.1.0&color=brightcolor)
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![Emacs](https://img.shields.io/badge/Emacs-27.1+-purple.svg)](https://www.gnu.org/software/emacs/)



# org-weekly-schedule

Build weekly time-block schedule tables inside Emacs Org mode.

## Overview

`org-weekly-schedule` generates Org mode tables that map activity codes
(A, B, C, ...) to 90-minute time blocks across the days of the week.
You define reusable **day templates** — named patterns like
"research-heavy" or "teaching-day" — then stamp them into a schedule
grid one column at a time.

The package operates in three layers:

| Layer          | Function                                  | Purpose                                     |
|----------------|-------------------------------------------|---------------------------------------------|
| Scaffold       | `org-weekly-schedule-insert`              | Insert the base table (Time column only)    |
| Column stamp   | `org-weekly-schedule-append-day`          | Append one day column from a template       |
| Week builder   | `org-weekly-schedule-build-week`          | Insert scaffold + stamp all days at once    |

## Requirements

- Emacs 25.1 or later
- Org mode 9.0 or later (ships with Emacs 26+)

## Installation

### Manual

Clone or download this repository, then add the directory to your
`load-path`:

```elisp
(add-to-list 'load-path "/path/to/org-weekly-schedule")
(require 'org-weekly-schedule)

(with-eval-after-load 'org
  (define-key org-mode-map (kbd "C-c t i") #'org-weekly-schedule-insert)
  (define-key org-mode-map (kbd "C-c t w") #'org-weekly-schedule-build-week)
  (define-key org-mode-map (kbd "C-c t a") #'org-weekly-schedule-append-column)
  (define-key org-mode-map (kbd "C-c t d") #'org-weekly-schedule-append-day)
  (define-key org-mode-map (kbd "C-c t h") #'org-weekly-schedule-add-date-header))
```

### use-package

```elisp
(use-package org-weekly-schedule
  :load-path "/path/to/org-weekly-schedule"
  :bind (:map org-mode-map
         ("C-c t i" . org-weekly-schedule-insert)
         ("C-c t w" . org-weekly-schedule-build-week)
         ("C-c t a" . org-weekly-schedule-append-column)
         ("C-c t d" . org-weekly-schedule-append-day)
         ("C-c t h" . org-weekly-schedule-add-date-header)))
```

## Quick start

1. Open an Org buffer.
2. Run `M-x org-weekly-schedule-build-week`.
3. Enter a start date when prompted (e.g. `2026-03-16` for Monday
   March 16) or press RET to skip the date header.
4. At each day prompt, type a template name (`research-heavy`,
   `balanced`, etc.) or press RET to skip that day.
5. A complete schedule table appears at point.

The result looks like this (with a date header):

```
| Day           | M  | Tu | W  |
| Date          | 16 | 17 | 18 |
|---------------+----+----+----|
| Time          | M  | Tu | W  |
|---------------+----+----+----|
| Generative:   |    |    |    |
| 04:00-05:30   | A | B  | A  |
| 05:45-07:15   | A | B  | B  |
| 07:30-09:00   | A | A  | A  |
|---------------+---+----+----|
| Rewriting:    |   |    |    |
| 09:15-10:45   | A | A  | B  |
| 11:30-13:00   | A | B  | A  |
|---------------+---+----+----|
| Supporting    |   |    |    |
| 13:15-14:45   | B | A  | A  |
| 15:00-16:30   | B | A  | B  |
| 16:45-18:15   | B | B  | B  |
| 20:30-22:00   | B | A  | A  |
|---------------+---+----+----|
|               |   |    |    |
| A:            |   |    |    |
|               |   |    |    |
| B:            |   |    |    |
|               |   |    |    |
| C:            |   |    |    |
|               |   |    |    |
|---------------+---+----+----|
```

## Defining templates

A template maps 9 activity-letter strings to the 9 time slots:

```elisp
(org-weekly-schedule-define-template "my-day" "MD"
  "A" "A" "B"    ; Generative  (04:00, 05:45, 07:30)
  "B" "A"        ; Rewriting   (09:15, 11:30)
  "A" "B"        ; Supporting  (13:15, 15:00)
  "B" "A")       ; Evening     (16:45, 20:30)
```

The first argument is the template name (used at the prompt).  The
second is a short default column header.  The remaining 9 strings are
the slot values from earliest to latest.  Use `""` for any slot you
want to leave blank.

### Built-in templates

| Name             | Header | Description                              |
|------------------|--------|------------------------------------------|
| `research-heavy` | RH     | Deep work all morning on project A       |
| `teaching-day`   | Tch    | Lighter generative, heavier supporting   |
| `balanced`       | Bal    | Alternating A/B throughout the day       |
| `light`          | Lt     | Skips earliest slot, evenings off        |
| `off`            | Off    | Completely blank (rest day placeholder)  |

## Customization

Three `defcustom` variables control the table structure.  Change them
via `M-x customize-group RET org-weekly-schedule RET` or set them in
your init file.

### `org-weekly-schedule-time-slots`

A list of `(TIME-LABEL CATEGORY-OR-NIL)` pairs defining every row in
the Time column.  When `CATEGORY` is non-nil, a separator line and a
category-label row are inserted above that slot.

```elisp
(setq org-weekly-schedule-time-slots
      '(("06:00-07:30" "Morning:")
        ("07:45-09:15" nil)
        ("09:30-11:00" "Midday:")
        ("11:15-12:45" nil)
        ("14:00-15:30" "Afternoon:")
        ("15:45-17:15" nil)))
```

> **Important:** If you change the number of slots, every
> `org-weekly-schedule-define-template` call must supply the matching
> number of values.

### `org-weekly-schedule-legend-keys`

The labels in the legend section at the bottom of the table.

```elisp
(setq org-weekly-schedule-legend-keys '("A:" "B:" "C:" "D:"))
```

### `org-weekly-schedule-day-names`

The day labels used by the interactive week builder.

```elisp
(setq org-weekly-schedule-day-names '("M" "Tu" "W" "Th" "F" "Sa" "Su"))
```

### `org-weekly-schedule-dow-abbreviations`

A 7-element vector of day-of-week abbreviations used by the date
header, indexed 0=Sunday through 6=Saturday.

```elisp
;; Use "Wd" for Wednesday instead of "W":
(setq org-weekly-schedule-dow-abbreviations
      ["Su" "M" "Tu" "Wd" "Th" "F" "Sa"])
```

## Adding a date header

You can add Day and Date rows to the top of any schedule table so that
each column shows its calendar date:

```
M-x org-weekly-schedule-add-date-header
Start date for first day column (YYYY-MM-DD): 2026-03-16
```

This produces:

```
|-------------+----+-----+-----+----+----+----+----|
| Day         | M  | Tu  | W   | Th | F  | Sa | Su |
| Date        | 16 | 17  | 18  | 19 | 20 | 21 | 22 |
|-------------+----+-----+-----+----+----+----+----|
| Time        | M  | Tu  | W   | Th | F  | Sa | Su |
|  ...        |    |     |     |    |    |    |    |
```

The week builder also accepts an optional start date.  When called
interactively it prompts for one before the day-template prompts; when
called from Lisp, pass it as the second argument:

```elisp
(org-weekly-schedule-build-week
 '(("M" . "research-heavy") ("Tu" . "balanced"))
 "2026-03-16")
```

## Commands

| Command                                   | Keybinding   | Description                                |
|-------------------------------------------|--------------|--------------------------------------------|
| `org-weekly-schedule-insert`              | `C-c t i`    | Insert bare scaffold at point              |
| `org-weekly-schedule-build-week`          | `C-c t w`    | Insert scaffold + prompt for day templates |
| `org-weekly-schedule-append-column`       | `C-c t a`    | Append an empty column                     |
| `org-weekly-schedule-append-day`          | `C-c t d`    | Append a column from a named template      |
| `org-weekly-schedule-add-date-header`     | `C-c t h`    | Add Day/Date rows to existing table        |
| `org-weekly-schedule-delete-last-column`  | —            | Remove the rightmost column                |

## Running tests

The test suite uses ERT (Emacs Lisp Regression Testing).

```bash
# Run all tests
make test

# Byte-compile
make compile

# Compile with strict warnings (lint)
make lint

# Clean compiled files
make clean

# Compile then test
make all
```

Or run interactively inside Emacs:

```
M-x ert RET t RET
```

## Project structure

```
org-weekly-schedule/
├── Makefile                      # Build and test automation
├── README.md                     # This file
├── org-weekly-schedule.el        # Package source
└── org-weekly-schedule-test.el   # ERT test suite
```

## License

GNU General Public License v3.0 or later.  See the file headers for
the full license text.

## Update table


| Version | Changes                                                                                                                | Date              |
|:---------|:------------------------------------------------------------------------------------------------------------------------|:-------------------|
|   0.1.0   | Initiate project. Added badges, funding, and this update table.                                                        | 2026 March 15 |

## Sources of funding

- NIH: R01 CA242845
- NIH: R01 AI088011
- NIH: P30 CA225520 (PI: R. Mannel)
- NIH: P20 GM103640 and P30 GM145423 (PI: A. West)
