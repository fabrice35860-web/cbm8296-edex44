# EDEX 4.4 — Reference Manual

*BASIC extension for the Commodore CBM 8000 / 8296 (French version)*
*(C) 1981 Eristel — manual reconstructed in 2026 by disassembling the ROM*

> No original documentation has been found. This manual is based on an
> analysis of the EDEX 4.4 ROM code and on tests with a French CBM 8296.
> Points that are still uncertain are marked **(to be confirmed)**.

---

## 1. Overview

EDEX is a 4K ROM that adds about twenty commands to BASIC 4.0:
programming aids (auto line numbering, renumbering, search, line deletion,
trace), language extensions (`IF…ELSE`, `PRINTUSING`, `CALL`), and disk
commands (`MERGE`, `FETCH`, `SWAP`).

It was designed for French CBM 8000 machines. Each EDEX version only works
with one specific editor ROM ($E000): version 4.4 requires the French
editor ROM it was paired with.

### Hardware

| Item | Location |
|---|---|
| EDEX 4.4 ROM | Option socket at $A000 (UD11 on 8032/8096, equivalent option socket on the 8296) |
| Matching editor ROM | UE8 on the 8296 ($E000-$E7FF and $E900-$EFFF) |

## 2. Getting started

```
SYS 40960
```

The screen shows:

```
EDEX V4.4 (C)ERISTEL 1981
```

The new commands are now available, both in direct mode and in programs.
To return to plain BASIC: `KILL`.

### Good to know

- EDEX keywords are stored as their own tokens ($E2 to $F5). A program
  using them only lists correctly and only runs while EDEX is active.
- `PRINTUSING` is written **as one word**. With a space (`PRINT USING`),
  BASIC reads `PRINT` followed by a variable named `USING`.
- Extra arguments cause `?SYNTAX ERROR` (example: `AUTO 10,10`).

## 3. Command list

| Command | Purpose |
|---|---|
| `AUTO` | Automatic line numbering |
| `RENU` | Renumbering |
| `DELETE` | Delete a range of lines |
| `LIST` | Listing (replaces the BASIC LIST) |
| `FIND` | Search for text in the program |
| `DUMP` | Display variables |
| `ERROR` | Show the line of the last error |
| `TRACE` / `OFF` | Execution trace |
| `IF … ELSE` | Alternative branch in an IF |
| `PRINTUSING` | Formatted output |
| `CALL` | Call a machine-code routine with parameters |
| `BEEP` | Beep |
| `HARDCOPY` | Screen dump to printer |
| `MERGE` | Merge a program from disk |
| `FETCH` | Load a program from disk |
| `SWAP` | Chain programs, keeping variables |
| `KILL` | Disable EDEX |
| `PLOT`, `RESET` | Reserved, not implemented in 4.4 |

## 4. Command reference

### AUTO — automatic line numbering

```
AUTO step
AUTO
```

Turns on automatic line numbering with the given step (1 to 255). Type the
first line yourself; after each line is entered, EDEX offers the next line
number.

```
AUTO 10
10 PRINT "HELLO"
20 _            <- offered by EDEX
```

`AUTO` alone (or `AUTO 0`) turns it off. A step of 256 or more gives
`?ILLEGAL QUANTITY`.

### RENU — renumber

```
RENU
RENU step
RENU start,step
RENU from,start,step
```

Renumbers the program and updates the line numbers referenced after
`GOTO`, `GOSUB`, `THEN`, `RUN` and `ELSE` (including `ON … GOTO` lists).
Parameters are read **from the right**:

| Form | Effect |
|---|---|
| `RENU` | Start 100, step 10, whole program |
| `RENU 5` | Start 100, step 5 |
| `RENU 1000,10` | Start 1000, step 10 |
| `RENU 500,2000,20` | Lines from 500 onwards become 2000, 2020, 2040… |

The step must be below 256. Line numbers that grow too large cause
`?ILLEGAL QUANTITY`.

### DELETE — delete lines

```
DELETE 100        deletes line 100
DELETE 100-200    deletes lines 100 to 200
DELETE 100-       from 100 to the end
DELETE -200       from the start to 200
```

Without arguments: `?SYNTAX ERROR`. Variables are preserved.

### LIST — list the program

```
LIST
LIST 100-200
```

Same syntax as BASIC 4.0 `LIST`. EDEX replaces the original `LIST` so it
can display its own keywords.

### FIND — search

```
FIND /text/
FIND "text"
FIND /text/ 100-500
```

Lists every line containing the text. The first character after `FIND` is
the delimiter.

- With `/…/` (or any other delimiter), the text is tokenised like BASIC:
  `FIND /PRINT/` finds the PRINT statement.
- With `"…"`, the text is searched literally, including inside strings.

A line range may follow, as with `LIST`.

### DUMP — show variables

```
DUMP
```

Prints all simple variables and their values:

```
A=3.5
N%=12
NAME$="SMITH"
```

Arrays are not shown. The STOP key interrupts the listing.

### ERROR — locate an error

```
ERROR
```

After a program has stopped on an error, shows the faulty line and marks
the exact place where the error occurred.

### TRACE / OFF — execution trace

```
TRACE
RUN
...
OFF
```

While the program runs, the current line is displayed on the top screen
line and a marker appears in the top right corner. The SHIFT key pauses
and single-steps the program **(exact behaviour to be confirmed)**. `OFF`
stops the trace.

### IF … ELSE

```
IF condition THEN statements ELSE statements
IF condition GOTO line ELSE statements
```

Adds `ELSE` to BASIC `IF`. Up to 16 nested `IF`s are supported. An `ELSE`
without a matching `IF` gives `?ELSE WITHOUT IF`.

```
10 INPUT A
20 IF A>0 THEN PRINT "POSITIVE" ELSE PRINT "ZERO OR NEGATIVE"
```

### PRINTUSING — formatted output

```
PRINTUSING "format",value[,value...][;]
PRINTUSING#n,"format",value...
```

Prints one or more values according to a format. A trailing `;` suppresses
the newline. With `#n`, output goes to open file `n` (for example a
printer). An invalid format gives `?BAD FORMAT`.

Format characters found in the code **(exact meaning to be confirmed by testing)**:

| Character | Probable meaning |
|---|---|
| `9` | Digit position |
| `Z` | Digit, zero-filled |
| `.` | Decimal point |
| `,` | Separator / end of field |
| `S` | Print sign |
| `+` `-` | Forced sign |
| `$` | Currency symbol (character stored at $03DE = 990, `$` by default) |
| `E` | Exponential notation |
| `A` | Alphanumeric field (strings) |
| `\text\` | Literal text, copied as is |
| `*` | Printed when the value does not fit the format |

### CALL — call machine code

```
CALL address[,param1,param2...]
```

Calls a machine-code routine. Parameters (numbers, strings or variables)
are evaluated and stored in a table starting at $0210, 4 bytes per
parameter (type, length, address), up to 15 parameters. The routine
returns to BASIC with `RTS`.

### BEEP

```
BEEP
```

Sounds the bell, using the editor ROM routine.

### HARDCOPY — screen dump

```
HARDCOPY
```

Prints the screen contents on printer device 4, converting graphic
characters.

### MERGE — merge programs

```
MERGE "name"[,Ddrive][,Uunit]
```

Reads a program from disk and inserts its lines into the program in
memory; a line with the same number replaces the existing one. Parameter
syntax is the same as `DLOAD`.

### FETCH — load

```
FETCH "name"[,Ddrive][,Uunit]
```

Loads a program from disk, in direct mode or from a running program
**(exact differences from DLOAD to be confirmed)**.

### SWAP — chain programs

```
SWAP "name"[,Ddrive][,Uunit]
```

Loads another program and runs it **keeping the current variables**.
Useful to split a large application into modules.

> `MERGE`, `FETCH` and `SWAP` use the BASIC 4.0 disk routines: they need a
> disk drive and do not work with tape.

### KILL — disable EDEX

```
KILL
```

Restores plain BASIC (CHRGET routine and interrupt vectors). `SYS 40960`
re-enables EDEX.

### PLOT, RESET

Recognised keywords, but **not implemented** in version 4.4: they jump
straight back to `READY.`, which stops a running program.

## 5. Additional error messages

| Message | Cause |
|---|---|
| `?BAD FORMAT` | Invalid `PRINTUSING` format |
| `?ELSE WITHOUT IF` | `ELSE` without a matching `IF` |

## 6. Technical information

| Item | Value |
|---|---|
| Address | $A000-$AFFF (4K) |
| Activation | `SYS 40960` |
| Method | Wedges CHRGET ($0070), the IRQ vector ($0090) and the BRK vector ($0092) |
| Work area | $03C3-$03E1 |
| Tokens | $E2-$F5 (IF uses the standard token $8B) |
| Dependency | French editor ROM routines ($E000), including the bell at $E02A |

| Token | Keyword | Token | Keyword |
|---|---|---|---|
| $8B | IF | $EC | BEEP |
| $E2 | FETCH | $ED | MERGE |
| $E3 | LIST | $EE | PRINTUSING |
| $E4 | AUTO | $EF | ELSE |
| $E5 | FIND | $F0 | CALL |
| $E6 | DUMP | $F1 | TRACE |
| $E7 | ERROR | $F2 | OFF |
| $E8 | RENU | $F3 | HARDCOPY |
| $E9 | DELETE | $F4 | SWAP |
| $EA | PLOT | $F5 | KILL |
| $EB | RESET | | |

The $03C3-$03E1 work area is inside the second cassette buffer: avoid
using cassette #2 while EDEX is active.

---

*Manual by Fafa, October 2026, based on the ROM of a French CBM 8296.
Corrections and additions welcome.*
