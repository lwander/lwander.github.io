---
layout: post
title: "Guitar Voicings"
date: 2020-05-10 00:00:00 -0400
categories: music
---

As I was working my way through [Melbay's Complete Book of Harmony, Theory &
Voicing](https://www.melbay.com/Products/95112/complete-book-of-harmony-theory--voicing.aspx),
I realized that I had no way to verify solutions to the exercises the book
prescribes. So I built a tool to help.

Usage:

```
$ ./gchrd --help

usage: gchrd.py [-h] [--chord_name CHORD_NAME]
                [--chord_voicing CHORD_VOICING [CHORD_VOICING ...]] --root
                ROOT [--drop DROP] [--invert INVERT]

Guitar voicing finder

optional arguments:
  -h, --help            show this help message and exit
  --chord_name CHORD_NAME
                        Chord name (no root), e.g. maj7
  --chord_voicing CHORD_VOICING [CHORD_VOICING ...]
                        Chord voicing, e.g. 1 3 5 7
  --root ROOT           Chord root
  --drop DROP           Drop n voicing
  --invert INVERT       Invert n voicing
```

You can use it find notes on the fretboard:

```
$ ./gchrd --root C4

VOICING: 1
CHORD  : C4
E4  | -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -
B3  | C4 -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -
G3  | -  -  -  -  C4 -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -
D3  | -  -  -  -  -  -  -  -  -  C4 -  -  -  -  -  -  -  -  -  -  -
A2  | -  -  -  -  -  -  -  -  -  -  -  -  -  -  C4 -  -  -  -  -  -
E2  | -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  C4 -
            .     .     .     .        :        .     .     .     .
```

Or to help you find an interval:

```
$ ./gchrd --root C4 --chord_voicing 1 7

VOICING: 1 7
CHORD  : C4 B4
E4  | -  -  -  -  -  -  B4 -  -  -  -  -  -  -  -  -  -  -  -  -  -
B3  | C4 -  -  -  -  -  -  -  -  -  -  B4 -  -  -  -  -  -  -  -  -
G3  | -  -  -  -  C4 -  -  -  -  -  -  -  -  -  -  B4 -  -  -  -  -
D3  | -  -  -  -  -  -  -  -  -  C4 -  -  -  -  -  -  -  -  -  -  B4
A2  | -  -  -  -  -  -  -  -  -  -  -  -  -  -  C4 -  -  -  -  -  -
E2  | -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  C4 -
            .     .     .     .        :        .     .     .     .

$ ./gchrd --root C4 --chord_voicing 7 1

VOICING: 7 1
CHORD  : B3 C4
E4  | -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -
B3  O C4 -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -
G3  | -  -  -  B3 C4 -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -
D3  | -  -  -  -  -  -  -  -  B3 C4 -  -  -  -  -  -  -  -  -  -  -
A2  | -  -  -  -  -  -  -  -  -  -  -  -  -  B3 C4 -  -  -  -  -  -
E2  | -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  B3 C4 -
            .     .     .     .        :        .     .     .     .
```

Or to show all notes in a given a chord:

```
$ ./gchrd.py --root C4 --chord_name maj7b5

VOICING: 1 3 b5 7
CHORD  : C4 E4 F#4 B4
E4  O -  F#4-  -  -  -  B4 -  -  -  -  -  -  -  -  -  -  -  -  -  -
B3  | C4 -  -  -  E4 -  F#4-  -  -  -  B4 -  -  -  -  -  -  -  -  -
G3  | -  -  -  -  C4 -  -  -  E4 -  F#4-  -  -  -  B4 -  -  -  -  -
D3  | -  -  -  -  -  -  -  -  -  C4 -  -  -  E4 -  F#4-  -  -  -  B4
A2  | -  -  -  -  -  -  -  -  -  -  -  -  -  -  C4 -  -  -  E4 -  F#4
E2  | -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  C4 -
            .     .     .     .        :        .     .     .     .
```

Clearly a maj7b5 chord is difficult to play with that voicing, so you can cycle
through inversions and drop voicings to find one that's a little easier on the
fingers:

```
$ ./gchrd.py --root C4 --chord_name maj7b5 --drop 2 --invert 2

VOICING: 1 b5 7 3
CHORD  : C4 F#4 B4 E5
E4  | -  F#4-  -  -  -  B4 -  -  -  -  E5 -  -  -  -  -  -  -  -  -
B3  | C4 -  -  -  -  -  F#4-  -  -  -  B4 -  -  -  -  E5 -  -  -  -
G3  | -  -  -  -  C4 -  -  -  -  -  F#4-  -  -  -  B4 -  -  -  -  E5
D3  | -  -  -  -  -  -  -  -  -  C4 -  -  -  -  -  F#4-  -  -  -  B4
A2  | -  -  -  -  -  -  -  -  -  -  -  -  -  -  C4 -  -  -  -  -  F#4
E2  | -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  -  C4 -
            .     .     .     .        :        .     .     .     .
```

Enjoy!
