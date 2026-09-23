# Building NC

NC is built from two sources: NC.PAS, the main program, written in
Oregon Software Pascal-2, and NCIO.MAC, a MACRO-11 module holding the
terminal, file system and spawn routines that Pascal-2 does not provide.
The procedure below was used on RSX-11M-PLUS V4.6 BL87 with Pascal-2
V2.1E.


## Prerequisites

Pascal-2 must be installed so that the MCR command PAS runs
LB:[3,54]PAS.TSK. The build also uses the Pascal-2 object library
LB:[1,1]PASLIB.OLB and the Pascal-2 macro file SY:[PAS]PASMAC.MAC,
which defines the proc, func and param macros used by NCIO.MAC.

MACRO-11, the Task Builder and the standard system libraries
(LB:[1,1]SYSLIB.OLB and LB:[1,1]RSXMAC.SML) are also required. These
are part of every RSX-11M-PLUS system.

No privileges are needed to build NC. Creating a new UFD and
installing the task do require a privileged account.


## Files

    NC.PAS       Main program
    NCIO.MAC     MACRO-11 support routines
    NCBLD.CMD    Task Builder command file
    NCMAKE.CMD   Build procedure


## Setting up

Create a directory for NC and make it the default:

    >UFD DB0:[NC]
    >SET /DEF=DB0:[NC]

Copy the four files into it by the usual means (PIP, FLX, NFT or FTP).
If no transfer method is available the files can be entered through
the terminal with PIP:

    >SET /BUF=TI:132.
    >SET /LOWER=TI:
    >PIP NC.PAS=TI:

and ended with CTRL/Z. Widen the terminal buffer first, as shown.
Otherwise the terminal driver splits long lines. When pasting from a
terminal emulator, send one line at a time; RSX will drop characters
that overrun the type-ahead buffer.


## Building

With the default directory set to DB0:[NC]:

    >@NCMAKE

NCMAKE.CMD runs the three steps of the build:

    PAS NC=NC
    MAC NCIO=NCIO
    TKB @NCBLD

A clean build produces no messages. The compile takes about a minute
under SIMH. The result is NC.TSK, a contiguous file of about 116
blocks, and the map NC.MAP.

The steps may also be run by hand. To get listings, use

    >PAS NC,NC=NC
    >MAC NCIO,NCIO/-SP=NCIO

NCIO.MAC reads the Pascal-2 macros with

            .include /SY:[PAS]PASMAC.MAC/

If PASMAC.MAC is kept elsewhere, change this line. Do not name
PASMAC.MAC on the MAC command line instead. The directory in
[PAS]PASMAC carries over to the next file on the line, and MAC then
looks for [PAS]NCIO.MAC.

NCBLD.CMD contains

    NC/CP,NC/-SP=NC,NCIO,LB:[1,1]PASLIB/LB
    /
    UNITS=16
    //

The UNITS=16 option must not be omitted. NCIO uses logical units 13,
14 and 15. In a task built with the default number of units, every
QIO on those units fails with IE.ILU (-96) and NC starts with empty
panels. NCIO also uses event flags 22, 23 and 24. These are not used
by the Pascal-2 run-time library.


## Running

    >RUN DB0:[NC]NC

The terminal must accept VT100 escape sequences and the DEC special
graphics character set. NC takes the screen width and length from the
terminal driver, so these should be set correctly beforehand:

    >SET /VT100=TI:
    >SET /BUF=TI:80.
    >SET /LINES=TI:24.

Widths from 80 to 132 columns and lengths from 16 to 66 lines are
supported. At 132 columns a time column is added to each panel.

To run NC by name, install it:

    >INS DB0:[NC]NC/TASK=...NC

and add the same line to the system startup procedure if it is to be
permanent.


## Checking the build

Run NC. The left panel should show the default directory and the
right panel the master file directory, [0,0], with sizes and dates
filled in. Tab to the right panel, move to a directory and press
RETURN. Press RETURN on ".." to go back; the cursor should return to
the directory just left. Press F3 on NC.PAS and page through the
file. Type TIM on the command line and press RETURN; the time should
be displayed. Press F10 and answer Y to leave NC.


## Problems

"Out of memory in procedure ..." from PAS means that one procedure is
too large for the compiler. Break it into smaller procedures.

Error 154 from PAS ("Actual parameter cannot be used with this
conformant array parameter") occurs when a value conformant array is
passed on to another conformant array parameter. Pass the elements
individually instead.

"Open failure on input file" from MAC means PASMAC.MAC was not found.
Check the .include line in NCIO.MAC.

"File NCIO.OBJ;n has illegal format" from TKB means the assembly
failed and left an empty object file. Delete it, correct the
assembly, and build again.

If NC starts with empty panels, check that the task was built with
UNITS=16.

If the line drawing appears as the letters l, q, k and x, the terminal
does not support the DEC special graphics set.


## Memory

NC.TSK is about 28,600 words, leaving some 4,000 words of the 32K word
address space for the Pascal stack and heap. Most of the data area is
taken by the two directory panels, which hold MAXENT entries of 22
bytes each. MAXENT is 250. It can be raised in NC.PAS if larger
directories must be shown, but check the task image size in NC.MAP
afterwards and keep it under about 30,000 words.
