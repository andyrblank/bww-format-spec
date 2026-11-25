# BWW format specification

A community specification of the `.bww` file format used by **Bagpipe Music Writer** (BMW) and its successors for
Great Highland Bagpipe music.

BMW has never published a specification for its format, yet thousands of pipe tunes are shared as `.bww` files. This
repository writes down what the format means, so that anyone building a reader, converter, engraver, player or
music-recognition tool can get the details right and agree with one another.

## What's here

| File | Contents |
|---|---|
| [`SPEC.md`](SPEC.md) | The specification: file layout, every token, bar-length rules, a canonical form for writers, real-world quirks, examples, and a mapping to MEI. |
| [`bww-tokens.json`](bww-tokens.json) | Machine-readable token tables: pitches, regex patterns for the token families, the fixed tokens, and all 363 embellishment names with their gracenotes. |
| [`examples/`](examples/) | Complete `.bww` files for the examples in the spec. |

## How it was made

- **From real files.** The rules were written from, and tested against, a corpus of 4,194 real-world `.bww` files.
- **From BMW's own symbol map.** Bagpipe Music Writer's `bmw32.ini` lists each of its 658 symbols, its category and
  the notes BMW plays for it. Every embellishment and symbol here was compared with it. Only facts (names, categories
  and note sequences) were taken from it, for interoperability; none of its contents are reproduced in this repository.
- **Cross-checked against open-source readers.** The published source code of limepipes-plugin-bww,
  bww-to-lilypond and bww2tex was read to compare their reading of the format with this one. No code was copied.
- **Verified against WebPiper's displayed output.** WebPiper is closed source, and none of its code was inspected,
  decompiled or reverse engineered. It was used only as an ordinary user would use it: a test tune written for this
  project was typed into the editor, and the sheet it displayed was compared by eye with the result this spec
  describes. No other tunes were sent to it.
- Where any of these sources disagree, [SPEC.md, Appendix C](SPEC.md#appendix-c-cross-check-against-other-implementations)
  records the difference and the choice made.

## Credits

This specification stands on the work of others. Thank you to:

- **Bagpipe Music Writer** and its authors, who created the format, and whose symbol map
  (`bmw32.ini`) is the closest thing to an official definition.
- **[limepipes-plugin-bww](https://github.com/tomvodi/limepipes-plugin-bww)** (GitHub: tomvodi; MIT), a Go parser for BWW.
  Its symbol mapper, tokenizer and handling of ties, tuplets, time lines and header lines were a reference throughout.
- **[bww-to-lilypond](https://github.com/graham-macmaster/bww-to-lilypond)** (GitHub: graham-macmaster), a Python BWW to
  LilyPond converter. It showed how ties, slurs and embellishments are written, and it carries the copy of BMW's
  symbol map that was used for the cross-check.
- **[bww2tex](https://github.com/DavidGriffith/bww2tex)** (GitHub: DavidGriffith; GPL-3.0), a lex/yacc BWW to TeX
  converter, whose lexer confirmed many of the piobaireachd names.
- **[WebPiper](https://webpiper.org/)**, an online BWW editor and engraver. Comparing its displayed engraving of
  a test tune was a useful independent check of how brackets, ending labels, grips and the piobaireachd
  abbreviations are drawn. (Output comparison only; see "How it was made".)
- **[MuseScore](https://musescore.org/)**, whose BWW importer is another long-standing reader of the format.
- **[MEI](https://music-encoding.org/)** and **[Verovio](https://www.verovio.org/)**, used for the engraving mapping
  in Appendix D.

Versions compared (2026-11-25): limepipes-plugin-bww `f738dc2`, bww-to-lilypond `a4242cb`, bww2tex `ddd6320`,
WebPiper v1.7.0.

## Contributing

Corrections are welcome, especially:
- a sheet printed by BMW itself that settles one of the open questions in Appendix C (the F and High G grips, and how
  BMW draws a tie mark over different pitches);
- tokens found in real files that the spec doesn't cover;
- differences between this spec and another implementation.

Please include a short `.bww` snippet that shows the case. Keep `SPEC.md` and `bww-tokens.json` in step: a change to
one should be reflected in the other.

## Trademarks

Bagpipe Music Writer, MuseScore, WebPiper and the other names above belong to their owners. This project is
independent and is not affiliated with or endorsed by any of them.

## Licence

The specification, the token tables and the examples are licensed under
[Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE). You may use them for any purpose, including in
commercial software, as long as you give credit.
