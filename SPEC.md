# The BWW file format (Bagpipe Music Writer)

**Status:** community reference, version 1.0 (2026-11-25). Not affiliated with or endorsed by the makers of Bagpipe
Music Writer.

`.bww` is the plain-text file format of Bagpipe Music Writer (BMW) and its successors (Bagpipe Music Writer Gold,
Bagpipe MusicWorks Gold). Thousands of Great Highland Bagpipe tunes circulate in it. BMW has no published
specification, so this document describes the format as it is actually written and read:

- as found in a corpus of **4,194 real-world `.bww` files**;
- checked against **Bagpipe Music Writer's own symbol map** (`bmw32.ini`), which lists each symbol's category and the
  notes BMW plays for it;
- cross-checked against several independent open-source readers, and verified against the displayed output of one
  closed-source engraver, WebPiper, used as an ordinary user (see [README.md](README.md#how-it-was-made) and
  [Appendix C](#appendix-c-cross-check-against-other-implementations)).

The exact token tables are in [`bww-tokens.json`](bww-tokens.json). Where prose and JSON disagree, the JSON is the more
precise statement; please report the difference.

**For implementers:** read `bww-tokens.json` for the token lists, then sections 3 (tokens), 4 (bar lengths) and 7
(canonical form). A file is well formed when every token is recognised (section 3) and every bar has the right length
(section 4).

Conventions: "must" and "should" are used in their ordinary sense. Durations are measured in quarter-note beats.

## 1. A whole file at a glance

```
Bagpipe Reader:1.0
MIDINoteMappings,(54,56,58,59,61,63,64,66,68,56,58,60,61,63,65,66,68,70,55,57,59,60,62,64,65,67,69)
FrequencyMappings,(370,415,466,494,554,622,659,740,831,415,466,523,554,622,699,740,831,932,392,440,494,523,587,659,699,784,880)
InstrumentMappings,(71,71,45,33,1000,60,70)
GracenoteDurations,(20,40,30,50,100,200,800,1200,250,250,250,500,200)
FontSizes,(90,100,88,80,250)
TuneFormat,(1,0,M,L,500,500,500,500,P,0,0)

TuneTempo,80

"Example March",(T,C,0,0,Times New Roman,18,700,0,0,18,0,0,0)
"March",(Y,L,0,0,Times New Roman,12,700,255,0,18,0,0,0)
"Trad.",(M,R,0,0,Times New Roman,12,700,255,0,18,0,0,0)
"",(F,R,0,0,Times New Roman,12,400,0,0,18,0,0,0)

& sharpf sharpc 2_4 I!'' gg LAr_8 'la Bl_16 ! dbc Cr_8 eg LAl_8 thrd D_4 !t
& sharpf sharpc gg Er_8 'e Fl_16 dbhg HGr_8 'hg El_16 ! tg HA_4 ''!I
```

- The first seven lines are program settings. A reader may skip them, but BMW expects them, so a writer should keep them.
- `TuneTempo` is the playing speed in beats per minute.
- Quoted lines are text: title (`T`), tune type (`Y`), composer (`M`) and footer (`F`).
- Each line starting with `&` is one printed staff. The words on it are **tokens**, separated by spaces or tabs.

## 2. File layout

### 2.1 Encoding
- UTF-8 or Windows-1252 (BMW Gold usually saves Windows-1252). A reader should try UTF-8 first and fall back to
  Windows-1252.
- A byte-order mark is ignored, as is a trailing DOS end-of-file mark (Ctrl+Z).
- A NUL byte or other control characters (apart from tab, CR and LF) mean the file isn't text.
- Line endings may be CRLF or LF.

### 2.2 Header lines
Lines starting with `Bagpipe ` (`Bagpipe Reader:1.0`, `Bagpipe Music Writer Gold:1.0`, `Bagpipe Musicworks Gold:1.0`),
`MIDINoteMappings`, `FrequencyMappings`, `InstrumentMappings`, `GracenoteDurations`, `FontSizes` and `TuneFormat` are
program settings and carry no music. Their lists can wrap onto the next line. A few files have stray characters before
the first header line, and some put a whole tune on a single line; a tolerant reader handles both.

`TuneTempo,<n>`: `n` is a positive whole number (beats per minute).

### 2.3 Text lines
`"<text>",(<tag>,<align>,<x>,<y>,<font>,<size>,<weight>,…)`. The attribute list sets alignment, position and font;
for the music only the tag matters:

| Tag | Meaning | Usually printed |
|---|---|---|
| `T` | Title | page header, centre |
| `Y` | Tune type (March, Strathspey, …) | page header, left |
| `M` | Composer or arranger | page header, right |
| `F` | Footer (band, copyright) | page footer |
| `I` | Inline text above a staff (part or variation names) | bold, above the staff, at the next bar |
| `X` | Free text at page coordinates | where the coordinates say |

- Quoted text can appear in the middle of a music line, and it can wrap over up to three more lines.
- A reader should blank out all quoted text before reading tokens, so a quoted word is never mistaken for music.

### 2.4 Music lines
Every other line is music. Tokens are read in order across lines, so a line break means nothing by itself: the staff
structure comes from `&` and the `…t` barlines (section 3.5).

## 3. Tokens

### 3.1 Pitches
The nine notes of the chanter. A **note token** uses the upper-case form; dots, fermatas, accidentals, old-style ties
and triplets use the lower-case form.

| Token | Lower | Note | Written pitch |
|---|---|---|---|
| `LG` | `lg` | Low G | G4 |
| `LA` | `la` | Low A | A4 |
| `B` | `b` | B | B4 |
| `C` | `c` | C | C5 |
| `D` | `d` | D | D5 |
| `E` | `e` | E | E5 |
| `F` | `f` | F | F5 |
| `HG` | `hg` | High G | G5 |
| `HA` | `ha` | High A | A5 |

C and F are played sharp; the key signature (`sharpf sharpc`) says so.

### 3.2 Notes, beams and dots
- **Note:** `<PITCH>[r|l]_<value>`, for example `LA_4`, `Dr_8`, `El_16`. Values: `1` whole, `2` half, `4` quarter,
  `8` eighth, `16` sixteenth, `32` thirty-second.
- **Beams:** `r` marks a note that starts or continues a beam, `l` the note that ends it (`Er_8 'e Fl_16`,
  `Dr_16 Er_16 Er_16 Dl_16`). An unbeamed note has no mark. Beam marks affect drawing only, never length.
- **Dot:** `'<pitch>` after the note (`''<pitch>` for a double dot). The pitch is where BMW draws the dot, which is
  often not the note's own pitch, so a reader ignores it: the dot belongs to the note before it. A dot with no note
  before it is an error.
- **Length** in quarter-note beats is `4 / value`, × 1.5 with a dot or × 1.75 with a double dot, × the tuplet ratio
  inside a tuplet.

### 3.3 Rests
`REST_<value>`, for example `REST_4`. Some files use the aliases `wrst` (= `REST_1`) and `hrst` (= `REST_2`). A rest
counts in the bar and is silence.

### 3.4 Staves, key and time signatures
| Token | Meaning |
|---|---|
| `&` | Starts a staff. After a closed bar it marks a line break there. |
| `sharpf` `sharpc` | Key signature, written after `&` on every staff. Pipe tunes always have both. Only the first staff's key signature counts; elsewhere they are repeated for display. |
| `<n>_<unit>` | Time signature: `2_4`, `3_4`, `4_4`, `6_8`, `9_8`, `12_8`, `2_2`, `5_4`, `6_4` … (BMW defines 35, up to `21_8` and `12_16`.) |
| `C` | Common time (4/4) |
| `C_` | Cut time (2/2) |

A time signature belongs to the bar it is written in. The meter in force is the last one written; before any is
written it is the tune's first meter (4/4 if the file has none). Writing the same meter again on a later staff changes
nothing.

Note that `C` is both the common-time symbol and the pitch C; a note token always has `_<value>` (`C_4`), so the two
never collide. `C_` alone (no number) is cut time.

### 3.5 Barlines and repeats
| Token | Meaning | Ends the staff line |
|---|---|---|
| `!` | Barline | |
| `!t` | Barline | yes |
| `!!` | Double barline | |
| `!!t` | Double barline | yes |
| `I!''` or `I!` | Repeat start (the next bar starts with repeat dots) | |
| `''!I` | Repeat end | |
| `''!It` | Repeat end | yes |
| `!I` | End of a part or of the tune (thin-thick barline) | |
| `!It` | End barline | yes |

- A barline straight after a closed bar belongs to that bar and can only make it stronger: `! ''!I` is one repeat end.
- The end of the file closes the last bar as an end barline.
- Two typing quirks are common enough to accept: `I!''` glued to the next token (`I!''gg`) and a barline glued to a
  gracenote (`!gg`, read as `!` then `gg`).

### 3.6 Endings and brackets
- `'1` starts a first-time ending and `'2` a second-time ending. `'22` … `'28` mean "second of n", and `'224` means
  "second and fourth". The first digit says which time is played. BMW prints the labels as "1.", "2.", "2 of 3" and
  "2 of 2 & 4".
- `'si`, `'do`, `'bis` and `'intro` start a **singling, doubling, bis or introduction bracket**. BMW files them with
  the endings (they are "time lines"), and they are drawn the same way: a bracket over the bars, labelled "Singling",
  "Doubling", "bis" or "Introduction". They do not change playback.
- An ending or bracket written as the first token of a bar starts with that bar; written after a note, it starts with
  the next bar.
- `_'` closes the open ending, or else the open bracket, at the end of its bar; it is written before the barline of
  the last bar. Where an ending and a bracket are both open, `_'` closes the ending.
- `bis_'` closes a `'bis` bracket (`_'` is also seen). Some files write `bis_'` inside a first-time ending, where it
  changes nothing.
- Two neighbouring endings with different labels (`'23` then `'224`) are two separate brackets.

### 3.7 Ties, slurs and tuplets
- **Tie mark (new style):** `^ts` before the first note and `^te` after the last (`^ts E_4 E_4 ^te`).
- **Tie mark (old style):** `^t<pitch>` written *after* the first note (`D_4 ^td D_4`). It joins that note to the
  next. A `^te` with no open `^ts` is read the same way.
- **Tie or slur.** BWW writes ties and slurs with the same marks:
  - Over notes that all have the same pitch, with no gracenotes after the first, it is a **tie**. Each note is tied to
    the next (a chain, for example `^ts D_4 D_4 D_4 ^te`); only the first note sounds, held for them all.
  - Otherwise it is a **slur** from the first note to the last, and every note under it sounds. This covers different
    pitches, and a later note with a gracenote (on the pipes a gracenote re-articulates the note).
  - A mark around a single note (`^ts C_2 'c ^te ! C_2`) joins it to the next note in the same way.
  - A mark that is never closed, or one on the tune's last note, has nothing to tie to. It occurs in real files (about
    100 in the corpus) and is best ignored.
- **Tuplet:** `^<n>s` … `^<n>e`, or `^<n><m>s` … `^<n><m>e` for n notes in the time of m. Without m, `^3` is 3 in the
  time of 2 and `^2` is 2 in the time of 3; any other n needs m (`^53s` = 5 in the time of 3). BMW defines `^2s` up to
  `^76s`. A tuplet that is never closed is an error.
- **Old-style triplet:** `^3<pitch>` after three notes makes those three a triplet. It needs three notes before it in
  the same bar.

### 3.8 Gracenotes and embellishments
- An embellishment token adds its gracenotes, in playing order, to the **next melody note**.
- Several embellishment tokens before one note join up in order (`gg dbe` gives HG HG E F).
- Gracenotes are never melody notes: they take no time in the bar and are drawn small with stems up (conventionally
  beamed thirty-seconds).
- Gracenotes left over at the end of the tune, with no note to attach to, are an error.

The table has **363 names** in 32 families. "Used" counts the names that occur at least once in the 4,194-file corpus.
In the table, **(= x)** means the name has exactly the same gracenotes as `x`, the canonical name (section 7).

The last four families are the piobaireachd movements, cadences and echo beats. Their notes are the ones BMW's symbol
map plays for them (pitches only; BMW plays some of them long). Movements BMW places over or under the next note are
best drawn as gracenotes before it.

| Family | Used | Name, gracenotes |
|---|---|---|
| Single gracenotes | 8/8 | `ag` LA; `bg` B; `cg` C; `dg` D; `eg` E; `fg` F; `gg` HG; `tg` HA |
| Doublings | 9/9 | `dblg` HG LG D; `dbla` HG LA D; `dbb` HG B D; `dbc` HG C D; `dbd` HG D E; `dbe` HG E F; `dbf` HG F HG; `dbhg` HG F; `dbha` HA HG |
| Thumb doublings | 7/7 | `tdblg` HA LG D; `tdbla` HA LA D; `tdbb` HA B D; `tdbc` HA C D; `tdbd` HA D E; `tdbe` HA E F; `tdbf` HA F HG |
| Half doublings | 7/7 | `hdblg` LG D; `hdbla` LA D; `hdbb` B D; `hdbc` C D; `hdbd` D E; `hdbe` E F; `hdbf` F HG |
| Strikes (single gracenote) | 8/8 | `strlg` LG; `strla` LA (= `ag`); `strb` B (= `bg`); `strc` C (= `cg`); `strd` D (= `dg`); `stre` E (= `eg`); `strf` F (= `fg`); `strhg` HG (= `gg`) |
| G-gracenote strikes | 7/7 | `gstla` HG LA LG; `gstb` HG B LG; `gstc` HG C LG; `gstd` HG D LG; `lgstd` HG D C; `gste` HG E LA; `gstf` HG F E |
| Thumb strikes | 5/8 | `tstla` HA LA LG; `tstb` HA B LG; `tstc` HA C LG; `tstd` HA D LG; `ltstd` HA D C; `tste` HA E LA; `tstf` HA F E; `tsthg` HA HG F |
| Half strikes | 8/8 | `hstla` LA LG; `hstb` B LG; `hstc` C LG; `hstd` D LG; `lhstd` D C; `hste` E LA; `hstf` F E; `hsthg` HG F (= `dbhg`) |
| Grips | 2/3 | `grp` LG D LG; `hgrp` D LG (= `hstd`); `grpb` LG B LG |
| G-gracenote grips | 2/7 | `ggrpla` HG LA LG D LG; `ggrpb` HG B LG D LG; `ggrpc` HG C LG D LG; `ggrpd` HG D LG D LG; `ggrpdb` HG D LG B LG; `ggrpe` HG E LG D LG; `ggrpf` HG F LG F LG |
| Thumb grips | 2/8 | `tgrpla` HA LA LG D LG; `tgrpb` HA B LG D LG; `tgrpc` HA C LG D LG; `tgrpd` HA D LG D LG; `tgrpdb` HA D LG B LG; `tgrpe` HA E LG D LG; `tgrpf` HA F LG F LG; `tgrphg` HA HG LG F LG |
| Half grips | 5/9 | `hgrpla` LA LG D LG; `hgrpb` B LG D LG; `hgrpc` C LG D LG; `hgrpd` D LG D LG; `hgrpdb` D LG B LG; `hgrpe` E LG D LG; `hgrpf` F LG F LG; `hgrphg` HG LG D LG; `hgrpha` HA LG D LG |
| Taorluaths and bubblys | 5/5 | `tar` LG D LG E; `tarb` LG B LG E; `htar` D LG E; `bubly` LG D LG C LG; `hbubly` D LG C LG |
| Birls | 4/4 | `brl` LG LA LG; `abr` LA LG LA LG; `gbr` HG LA LG LA LG; `tbr` HA LA LG LA LG |
| D throws | 3/4 | `thrd` LG D C; `hvthrd` LG D LG C; `hthrd` D C (= `lhstd`); `hhvthrd` D LG C |
| Peles | 7/7 | `pella` HG LA E LA LG; `pelb` HG B E B LG; `pelc` HG C E C LG; `peld` HG D E D LG; `lpeld` HG D E D C; `pele` HG E F E LA; `pelf` HG F HG F E |
| Thumb peles | 2/8 | `tpella` HA LA E LA LG; `tpelb` HA B E B LG; `tpelc` HA C E C LG; `tpeld` HA D E D LG; `ltpeld` HA D E D C; `tpele` HA E F E LA; `tpelf` HA F HG F E; `tpelhg` HA HG HA HG F |
| Half peles | 4/8 | `hpella` LA E LA LG; `hpelb` B E B LG; `hpelc` C E C LG; `hpeld` D E D LG; `lhpeld` D E D C; `hpele` E F E LA; `hpelf` F HG F E; `hpelhg` HG HA HG F |
| Double strikes | 4/9 | `st2la` LG LA LG (= `brl`); `st2b` LG B LG (= `grpb`); `st2c` LG C LG; `st2d` LG D LG (= `grp`); `lst2d` C D C; `st2e` LA E LA; `st2f` E F E; `st2hg` F HG F; `st2ha` HG HA HG |
| G-gracenote double strikes | 1/7 | `gst2la` HG LA LG LA LG (= `gbr`); `gst2b` HG B LG B LG; `gst2c` HG C LG C LG; `gst2d` HG D LG D LG (= `ggrpd`); `lgst2d` HG D C D C; `gst2e` HG E LA E LA; `gst2f` HG F E F E |
| Thumb double strikes | 1/8 | `tst2la` HA LA LG LA LG (= `tbr`); `tst2b` HA B LG B LG; `tst2c` HA C LG C LG; `tst2d` HA D LG D LG (= `tgrpd`); `ltst2d` HA D C D C; `tst2e` HA E LA E LA; `tst2f` HA F E F E; `tst2hg` HA HG F HG F |
| Half double strikes | 4/9 | `hst2la` LA LG LA LG (= `abr`); `hst2b` B LG B LG; `hst2c` C LG C LG; `hst2d` D LG D LG (= `hgrpd`); `lhst2d` D C D C; `hst2e` E LA E LA; `hst2f` F E F E; `hst2hg` HG F HG F; `hst2ha` HA HG HA HG |
| Triple strikes | 3/9 | `st3la` LG LA LG LA LG; `st3b` LG B LG B LG; `st3c` LG C LG C LG; `st3d` LG D LG D LG; `lst3d` C D C D C; `st3e` LA E LA E LA; `st3f` E F E F E; `st3hg` F HG F HG F; `st3ha` HG HA HG HA HG |
| G-gracenote triple strikes | 0/7 | `gst3la` HG LA LG LA LG LA LG; `gst3b` HG B LG B LG B LG; `gst3c` HG C LG C LG C LG; `gst3d` HG D LG D LG D LG; `lgst3d` HG D C D C D C; `gst3e` HG E LA E LA E LA; `gst3f` HG F E F E F E |
| Thumb triple strikes | 0/8 | `tst3la` HA LA LG LA LG LA LG; `tst3b` HA B LG B LG B LG; `tst3c` HA C LG C LG C LG; `tst3d` HA D LG D LG D LG; `ltst3d` HA D C D C D C; `tst3e` HA E LA E LA E LA; `tst3f` HA F E F E F E; `tst3hg` HA HG F HG F HG F |
| Half triple strikes | 1/9 | `hst3la` LA LG LA LG LA LG; `hst3b` B LG B LG B LG; `hst3c` C LG C LG C LG; `hst3d` D LG D LG D LG; `lhst3d` D C D C D C; `hst3e` E LA E LA E LA; `hst3f` F E F E F E; `hst3hg` HG F HG F HG F; `hst3ha` HA HG HA HG HA HG |
| Two gracenotes (D, E or F first) | 4/15 | `dlg` D LG (= `hstd`); `dla` D LA; `db` D B; `dc` D C (= `lhstd`); `elg` E LG; `ela` E LA (= `hste`); `eb` E B; `ec` E C; `ed` E D; `flg` F LG; `fla` F LA; `fb` F B; `fc` F C; `fd` F D; `fe` F E (= `hstf`) |
| Two gracenotes (High G or High A first) | 9/15 | `glg` HG LG; `gla` HG LA; `gb` HG B; `gc` HG C; `gd` HG D; `ge` HG E; `gf` HG F (= `dbhg`); `tlg` HA LG; `tla` HA LA; `tb` HA B; `tc` HA C; `td` HA D; `te` HA E; `tf` HA F; `thg` HA HG (= `dbha`) |
| Piobaireachd movements | 32/73 | `adeda` D LG E LG; `chechere` HG E HA E; `chedare` F E HG E; `chedari` F E HG E F E; `chelalho` F D E; `crunl` LG D LG E LA F LA; `crunlb` LG B LG E LA F LA; `crunlbbrea` LG B LG E LA F LA (= `crunlb`); `crunlblg` LG B LG E LG F LG; `crunlbrea` LG D LG E LA F LA (= `crunl`); `crunllg` LG D LG E LG F LG; `dare` F E HG E (= `chedare`); `darodo` LG D LG C LG (= `bubly`); `darodo16` LG D LG C LG (= `bubly`); `dbstf` F HG F (= `st2hg`); `dbsthg` HG HA HG (= `st2ha`); `deda` LG E LG; `dili` HA HG (= `dbha`); `dre` LA F LA; `edre` E LA F LA; `edreb` E B F B; `edrec` E C F C; `edred` E D F D; `edrela` E LA F LA (= `edre`); `edrelg` E LG F LG; `embari` E LG F LG (= `edrelg`); `enbain` LA LG D LG (= `hgrpla`); `endari` E LA F LA (= `edre`); `endarig` F E HG E (= `chedare`); `gadeda` HG D LG E LG; `gdare` HG F E HG E; `gedre` HG E LA F LA; `genbain` HG LA LG D LG (= `ggrpla`); `gendari` HG E F E; `godro` HG C LG D LG (= `ggrpc`); `gotro` HG B LG D LG (= `ggrpb`); `hchechere` E HA E; `hcrunl` D LG E LA F LA; `hcrunlla` D LA E LA F LA; `hcrunllabrea` D LA E LA F LA (= `hcrunlla`); `hcrunllg` D LG E LG F LG; `hcrunllgla` D LG E LA F LA (= `hcrunl`); `hdarodo` D LG C LG (= `hbubly`); `hedale` E HG E; `hedari` E HG E F E; `hiharin` D LA LG LA LG; `hlemla` D LA (= `dla`); `hlemlabrea` D LA (= `dla`); `hlemlg` D LG (= `hstd`); `htarla` D LA E; `htarlabrea` D LA E (= `htarla`); `htarlg` D LG E (= `htar`); `htra` D C (= `lhstd`); `lem` LG D LG (= `grp`); `lemb` LG B LG (= `grpb`); `lembbrea` LG B LG (= `grpb`); `lembrea` LG D LG (= `grp`); `odro` C LG D LG (= `hgrpc`); `otro` B LG D LG (= `hgrpb`); `rodin` LG B LG (= `grpb`); `tadeda` HA D LG E LG; `tarbbrea` LG B LG E (= `tarb`); `tarblg` LG B LG E (= `tarb`); `tarbrea` LG D LG E (= `tar`); `tarlg` LG D LG E (= `tar`); `tchechere` HA HG E HA E; `tdare` HA F E HG E; `tedre` HA E LA F LA; `tenbain` HA LA LG D LG (= `tgrpla`); `todro` HA C LG D LG (= `tgrpc`); `totro` HA B LG D LG (= `tgrpb`); `tra` LG D C (= `thrd`); `tra8` LG D C (= `thrd`) |
| Cadences | 10/16 | `cadae` HA E (= `te`); `cadaed` HA E D; `cadaf` HA F (= `tf`); `cade` E (= `eg`); `caded` E D (= `ed`); `cadge` HG E (= `ge`); `cadged` HG E D; `cadgf` HG F (= `dbhg`); `fcadae` HA E (= `te`); `fcadaed` HA E D (= `cadaed`); `fcadaf` HA F (= `tf`); `fcade` E (= `eg`); `fcaded` E D (= `ed`); `fcadge` HG E (= `ge`); `fcadged` HG E D (= `cadged`); `fcadgf` HG F (= `dbhg`) |
| Echo beats | 9/10 | `din` LG (= `strlg`); `echob` B (= `bg`); `echoc` C (= `cg`); `echod` D (= `dg`); `echoe` E (= `eg`); `echof` F (= `fg`); `echoha` HA (= `tg`); `echohg` HG (= `gg`); `echola` LA (= `ag`); `echolg` LG (= `strlg`) |
| Piobaireachd movements over or under the next note | 23/43 | `padeda` D LG E LG (= `adeda`); `pchechere` HG E HA E (= `chechere`); `pchedari` F E HG E F E (= `chedari`); `pcmb` HG B LG D LG B E B F B; `pcmc` HG C LG D LG C E C F C; `pcmd` HG B LG D C D E D F D; `pdare` F E HG E (= `chedare`); `pdarodo` LG D LG C LG (= `bubly`); `pdarodo16` LG D LG C LG (= `bubly`); `pdili` HA HG (= `dbha`); `pedre` E LA F LA (= `edre`); `pedreb` E B F B (= `edreb`); `pedrec` E C F C (= `edrec`); `pedred` E D F D (= `edred`); `pedrela` E LA F LA (= `edre`); `pedrelg` E LG F LG (= `edrelg`); `pembari` E LG F LG (= `edrelg`); `penbain` LA LG D LG (= `hgrpla`); `pendari` E LA F LA (= `edre`); `pgrp` LG D LG (= `grp`); `phdarodo` D LG C LG (= `hbubly`); `phedari` E HG E F E (= `hedari`); `phiharin` D LA LG LA LG (= `hiharin`); `phtra` D C (= `lhstd`); `phtripb` B D B E; `phtripc` C D C E; `phtripla` LA D LA E; `phtriplg` LG D LG E (= `tar`); `podro` C LG D LG (= `hgrpc`); `potro` B LG D LG (= `hgrpb`); `ptmb` HG B LG D LG E; `ptmc` HG C LG D LG E; `ptmd` HG B LG D C D E; `ptra` LG D C (= `thrd`); `ptra8` LG D C (= `thrd`); `ptripb` HG B D B E; `ptripc` HG C D C E; `ptripla` HG LA D LA E; `ptriplg` HG LG D LG E; `pttripb` HA B D B E; `pttripc` HA C D C E; `pttripla` HA LA D LA E; `pttriplg` HA LG D LG E |

**Abbreviations.** The taorluath, crunluath and lemluath abbreviations are drawn as a **T**, **c** or **L** under the
note *before* them (even across a barline) and add no gracenotes: `pt` `ptb` `phtla` `ptlg` `ptbrea` `ptbbrea` `ptblg`
`phtlabrea` `phtlg` are T; `pc` `pcb` `phcla` `pclg` `pcbrea` `pcbbrea` `pcblg` `phclabrea` `phclg` are c; `pl` `plb`
`plbrea` `plbbrea` `phlla` `phllabrea` are L.

**Undefined names.** `flick`, `hflick` and `sp2` occur in real files (22 times in the corpus) but no reference defines
them. A reader should report them rather than guess (`bww-tokens.json` → `notExpanded`).

**Grips on F and High G.** `ggrpf`, `hgrpf`, `tgrpf` and `tgrphg` are listed here with `LG F LG` in the grip, which
matches WebPiper's displayed engraving; BMW's symbol map plays `LG D LG`. They do not occur in the corpus. All other 217 names in the
first 28 families match the notes BMW plays exactly.

### 3.9 Accidentals, fermatas, text marks and layout
These tokens are drawn but change no lengths or pitches of the chanter notes:

| Token | Meaning |
|---|---|
| `flat<p>` `natural<p>` `sharp<p>` | Accidental on the next note, for example `naturalc`. `cnat` is an alias of `naturalc`. (`sharpf`/`sharpc` straight after `&` are the key signature instead.) |
| `fermat<p>` | Fermata on the note before it, for example `fermatd`. |
| `fine` `dacapoalfine` `dalsegno` `dacapoalcoda` `segno` `coda` `codasection` | Navigation marks. `codasection` (where the coda starts) is drawn as a coda sign. |
| `space` | Extra horizontal space in BMW's layout; no musical meaning. |

A mark written after the tune's last barline belongs to the last bar.

### 3.10 Anything else
Any other token is unknown. A reader should report it with its line and column and skip it, never drop it silently.
Other errors worth reporting:
- a dot or tie mark with no note before it;
- an old-style triplet without three notes before it in the bar;
- a tuplet that is never closed;
- gracenotes at the end of the tune;
- a `TuneTempo` that isn't a positive whole number;
- characters in text that can't be represented in the target format.

`.bww` files are often downloaded from the web, so a reader should treat them as untrusted input and cap the file
size, the text length and the number of notes and bars.

## 4. Bar lengths
A full bar is `count × 4 / unit` quarter-note beats of the meter in force (2/4 = 2, 6/8 = 3, 9/8 = 4.5). A bar's length
is the sum of its notes and rests (section 3.2); gracenotes take no time. A bar that isn't full is an error, unless it is:
- the first bar (a pickup);
- a bar that ends a repeat or the tune (`''!I`, `!I`);
- a short bar straight after one of those (the pickup into the next part), or just before a repeat start;
- one of the two halves of a bar split across a staff break, which together make one full bar.

## 5. Tune structure in brief
- A tune is a sequence of **bars**. A bar ends at a barline (section 3.5); a line break only happens at `&` or a `…t`
  barline.
- Each bar holds **notes** and **rests**. Each note can carry gracenotes (from the embellishment tokens before it),
  a dot, an accidental, a fermata, a tie or slur, and tuplet membership.
- Each bar can carry a time signature, an ending or bracket, text directions (`I` text, navigation marks,
  abbreviations) and a left and right barline style.

## 6. Recommended reading order of tokens around a note
Readers must accept any order, but files written by BMW, and the canonical form (section 7), use:

`^3s` → `^ts` → accidental → embellishment(s) → **note** → dot → fermata → `^te` → `^3e`

## 7. Canonical form (for programs that write BWW)
Many spellings mean the same music. A program that writes BWW should write just one, so output can be compared token
by token:

1. **Header:** the seven program lines from section 1, then `TuneTempo` if known, then the `T`, `Y`, `M` and `F` text
   lines with the attribute lists shown there. CRLF line endings, UTF-8.
2. **One music line per printed staff.** It starts `& sharpf sharpc`. The time signature follows on the first staff,
   and again wherever it changes (numbers, or `C`/`C_` when the sheet shows the symbol).
3. **Bars** are separated by `!`. A staff ends with its real barline: `!t` if it's a plain barline, otherwise `!!`,
   `''!I` or `!I` (the next `&` makes the line break). Don't use `''!It`, `!It`, `!!t` or `I!`. Repeats start with
   `I!''`.
4. **Order around a note** as in section 6.
5. **Embellishments:** use the canonical name for the gracenote sequence (the `canonical` field in `bww-tokens.json`:
   the first name in table order, so simpler families win: `gg` not `strhg`, `dbhg` not `gf`). A sequence that isn't
   in the table is written as the fewest canonical names that join up to it, matching the longest from the left. Every
   pitch has a single-gracenote name (`ag` … `tg`, plus `strlg` for Low G), so any sequence can be written.
6. **Beams:** `r` on every note of a beam group except the last, `l` on the last. No mark on unbeamed notes.
7. **Dots** use the note's own pitch: `Dr_8 'd`, `''d` for a double dot.
8. **Ties and slurs** are new style, `^ts` before the first note and `^te` after the last. **Tuplets** are
   `^3s … ^3e`, `^2s … ^2e` or `^nms … ^nme`, each closed. No old-style `^t<p>` or `^3<p>`.
9. **Rests** are `REST_<n>` (not `wrst`/`hrst`). `naturalc`, not `cnat`. No `space`; `bis_'` only to close a `'bis`
   bracket.
10. **Endings and brackets:** `'1`, `'2` (`'22`, `'224` as printed), `'si`, `'do`, `'intro`, `'bis` as the first token
    of the bar, and `_'` (`bis_'` for `'bis`) just before the barline that closes it.
11. Tokens are separated by one space.

## 8. Facts about real-world files
From the 4,194-file corpus:
- **Headers:** 2,932 start `Bagpipe Reader:1.0` and 1,210 `Bagpipe Music Writer Gold:1.0`; a few have stray
  characters before the header.
- **Quality:** 3,919 files read with no unknown tokens or other errors; 949 of those still have at least one bar of
  the wrong length. Real files are messy: a reader should report, not reject.
- **Token counts:** about 621k melody notes, 342k embellishment tokens, 119k dots, 91k `!`, 27k staves (`&`), 13k `!t`.
- **Most-used embellishments:** `gg` (139k), `dg`, `eg`, `dbe`, `thrd`, `strlg`, `tg`, `dbc`, `dbha`, `strla`.
- **Meters:** `2_4` and `6_8` (about 1,100 files each), then `4_4`, `C_`, `3_4`, `C`, `9_8`, `2_2`, `12_8`, `6_4`, `5_4`.
- **Ties:** new style `^ts`/`^te` in about 600–800 files, old style `^t<p>` in a few hundred.
- **Tuplets:** `^3s`/`^3e` about 1,600 times; old-style `^3<p>` about 2,300 times.
- **Tokens not in BMW's symbol map but seen in files:** `!!t`, `''!It`, `!It`, `wrst`, `hrst`, `cnat`, `!gg`, `flick`,
  `hflick`, `sp2`.

## 9. Examples
Each example uses only defined tokens and every bar has the right length. They are also in [`examples/`](examples/).

**2/4 march** (pickup bar, doublings, a throw, repeat):
```bww
& sharpf sharpc 2_4 I!'' gg LAr_8 'la Bl_16 ! dbc Cr_8 eg LAl_8 thrd D_4 ! gg Er_8 'e Fl_16 dbhg HGr_8 'hg El_16 ! tg HA_4 ''!I
```
Melody: A B · C A D · E F HG E · HA

**6/8 jig** (two staves, grip, strike on D):
```bww
& sharpf sharpc 6_8 I!'' gg LAr_8 Br_8 Cl_8 thrd D_4 'd ! gg Er_8 Dr_8 Cl_8 grp Br_8 LAr_8 LGl_8 !t
& sharpf sharpc gg LAr_8 Br_8 Cl_8 dbe E_4 'e ! gstd D_4 'd gg LA_4 'la ''!I
```
Melody: A B C D · E D C B A G · A B C E · D A

**3/4 slow air** (tie, triplet, fermata):
```bww
& sharpf sharpc 3_4 gg E_2 dbf F_4 ! ^ts HG_2 HG_4 ^te ! ^3s gg Fr_8 E_8 Dl_8 ^3e thrd D_2 ! gg LA_2 'la fermatla !I
```
Melody: E F · HG (held for 3 beats) · F E D D · A

## Appendix A. `bww-tokens.json`
The machine-readable tables:

| Key | Contents |
|---|---|
| `headerLines` | The program-settings lines (section 2.2). |
| `textTags` | The text tags and their meanings (section 2.3). |
| `pitches` | The nine pitches, upper- and lower-case forms, and a short letter name. |
| `durations`, `beamMarks` | Allowed note values and beam marks. |
| `patterns` | Regular expressions for the token families (notes, time signatures, dots, rests, accidentals, fermatas, endings, tuplets, old-style triplets and ties), each with examples. `context` is a minimal music line with `{token}` where the token goes. |
| `groups` | The fixed tokens (staves, meters, barlines, endings and brackets, ties and tuplets, rest aliases, marks, abbreviations, layout), each with a meaning, an example and `drawn` (whether it changes the engraved music). |
| `embellishments` | All 363 names: `graces` (in playing order), `family`, `canonical` (section 7), `corpusUses` (occurrences in the 4,194-file corpus) and `beforeD` (Appendix B). |
| `notExpanded` | Names seen in real files that no reference defines. |

## Appendix B. Note names for learners (informative)
Some tools print a note name above each melody note for pipers who don't read staff notation. A common convention:
- G, A, B, C, D, E, F, HG, HA, one per melody note, never for gracenotes. The second note of a tie is named again.
- A D whose gracenotes start on Low G and end on C is a **D throw**, "DT" (`thrd`, `hvthrd`, `tra`).
- A D whose gracenotes start on High G and end on Low G is a **D strike**, "DS" (`gstd`, `glg`, `ggrpd`, `peld`, …).
- Every other D is plain D, including `lgstd` (HG D C), the half throw `hthrd` (D C), `grp` (LG D LG) and `dbd`.

`bww-tokens.json` gives the name each embellishment produces on a D (`beforeD`).

## Appendix C. Cross-check against other implementations
On 2026-11-25 this reading was compared with BMW's symbol map and with the implementations listed in
[README.md](README.md#credits). The open-source readers were compared by reading their published source. WebPiper is
closed source and was compared only by looking at the sheet it displayed for a test tune; its code was not inspected or
reverse engineered. Where they agree:
- All 221 names in the first 28 embellishment families are BMW symbols, and 217 have exactly the notes BMW plays.
- Everything else BMW defines is covered: all 109 melody-note symbols, rests, dots and double dots, all 35 time
  signatures, barlines and repeats, endings `'1` to `'28` and `'224`, the tuplets `^2s` … `^76s`, accidentals,
  fermatas and the signs.
- The three "Bagpipe …" first lines are accepted by other readers too.
- WebPiper draws dots, double dots, triplet brackets, fermatas, naturals, common time, meter changes, endings and
  grips the same way.

Where they differ, and the choice made here:

| Topic | What the others do | This spec |
|---|---|---|
| `'si` `'do` `'bis` `'intro` | BMW files them as time lines; limepipes reads them as time lines; WebPiper draws labelled brackets. | Brackets over the bars up to `_'` (`bis_'` for `'bis`), like endings. No effect on playback. |
| Labels of `'22` … `'28`, `'224` | WebPiper prints "2of3", "2of2&4"; BMW-printed sheets show "2 of 2". | "2 of n", "2 of 2 & 4". |
| Grips `ggrpf`, `hgrpf`, `tgrpf`, `tgrphg` | BMW plays `LG D LG`; WebPiper draws `LG F LG`. | `LG F LG` (as drawn). Open question; not seen in the corpus. |
| A tie mark over different pitches | bww-to-lilypond and limepipes tie every note; WebPiper draws an arc between each pair. | A slur from first to last note; every note sounds. Open question: whether BMW draws one arc or a chain. |
| A dot whose pitch isn't its note's (`F_4 'e`) | WebPiper warns. | Accepted: very common in real files, and the dot still belongs to the note before. |
| `segno`, `fine`, `coda`, D.C., D.S. | WebPiper draws none of them; BMW defines them as signs. | Drawn. |
| `flick`, `hflick`, `sp2` and the quirks in section 8 | Not in BMW's map. | Read (or reported) for compatibility; not used in the canonical form. |

## Appendix D. Mapping to MEI (informative)
For engraving with an [MEI](https://music-encoding.org/) renderer such as [Verovio](https://www.verovio.org/), this
mapping is known to work:

| BWW | MEI |
|---|---|
| Notes, rests, dots, beams | `note`, `rest`, `@dots`, `beam` |
| Gracenotes | `graceGrp` of beamed 32nds, stems up |
| Ties, slurs, tuplets | `note@tie` (`i`, `m`, `t`), `slur@startid/@endid`, `tuplet` |
| Barlines, repeats, line breaks | `measure@left/@right`, `sb` |
| Endings and singling/doubling/bis/introduction brackets | `ending` with `@label` as printed. Where an ending and a bracket overlap, draw the ending. |
| Key signature, first time signature | `staffDef` with `keySig` and `meterSig` (`@sym` `common`/`cut` for `C`/`C_`) |
| A later time signature | `meterSig` as the first element of its bar's layer. (A new `scoreDef` makes Verovio print a courtesy meter at the end of the previous line, which BMW never does.) |
| Accidental | `note@accid` (replacing the key's `accid.ges`) |
| Fermata | `note@fermata="above"` |
| `fine`, D.C., D.S. | `dir` above the staff, on the note that follows |
| `segno`, `coda`, `codasection` | `repeatMark@func` `segno`/`coda` (renders as a glyph; a SMuFL character in a `dir` shows as an empty box in browsers) |
| `I` text | `dir` with bold text |
| Piobaireachd abbreviation (`pt`, `pc`, `pl` …) | `dir place="below"` holding T, c or L, on the note before |
| `T`, `Y`, `M`, `F` | `pgHead` (title centre, type left, composer right) and `pgFoot` |

Not drawn: `space`, `X` text, unclosed tie marks and the `notExpanded` names.
