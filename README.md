# A faster drop-in for csv-reader 1.0.0

[![make.fun](https://make.fun/badge/csv-reader-speed-01-run-20261121.svg)](https://make.fun/bounty/csv-reader-speed-01-run-20261121)
[![rank](https://make.fun/badge/csv-reader-speed-01-run-20261121/danzzkeceh4-bit/makefun-csv-reader-speed-01-run-20261121/rank.svg)](https://make.fun/bounty/csv-reader-speed-01-run-20261121)
[![reward](https://make.fun/badge/csv-reader-speed-01-run-20261121/danzzkeceh4-bit/makefun-csv-reader-speed-01-run-20261121/reward.svg)](https://make.fun/bounty/csv-reader-speed-01-run-20261121)

A make.fun entry for [csv-reader-speed-01-run-20261121](https://make.fun/bounty/csv-reader-speed-01-run-20261121):
`solve.js` answers exactly as make.fun's own csv-reader 1.0.0 does on every
case — same `{columns, rows, errors}` for every `{text, header}` — but
several times faster. It reads the text once, left to right, with the
reference's broken-row policy kept exactly: a leading U+FEFF is dropped,
records end at LF or CRLF, a CR outside quotes that is not followed by LF
is dropped (`stray_cr`), quoted fields honour doubled quotes and read
CRLF or a lone CR inside quotes as LF, junk after a closing quote is
appended as it stands (`quote_junk`), a quote that never closes is a
literal and the text after it is re-read as unquoted
(`unterminated_quote`), and rows are padded (`short_row`) or cut
(`long_row`) to the first record's width.

## How it is faster

The reference builds every field up a character at a time, one string
concatenation per character. This parser instead scans each field with a
tight `charCodeAt` loop that allocates nothing, then cuts the field out
with a single `slice`. Quoted fields take the same path: segments are
sliced out and joined only when a line break or a doubled quote actually
appears inside them. No dependency, no lookup table, no caching between
cases — `run(input)` is pure and stateless.

## Practice-set results

Measured on this machine (Node v24.22.1) against the kit's practice set
(6 cases, ~10.2 MB of CSV), with the kit's own scorer:

- **Correctness: 6/6 cases identical** — the scorer reports
  `passed: 1, cases_passed: 6, cases_total: 6`, and `answers.json` is
  byte-for-byte identical to the baseline's.
- **Fuzz: 4000/4000 random adversarial inputs identical** to csv-reader
  1.0.0 (quotes, stray CRs, CRLF/LF mixes, unterminated quotes, junk
  after closing quotes, BOM, empty lines).
- **Speed (`run()` only, fresh process, 1 warm-up + 1 timed pass, 7 runs
  each): median 124.8 ms vs csv-reader 1.0.0's 755.2 ms — about 6.0x
  faster** (mine 114.9–174.8 ms; reference 692.0–968.8 ms).
- **End to end (`main.ts`, including JSON parsing and writing
  `answers.json`): 909–1186 ms vs the baseline's 2090–2303 ms.**

Two runs write byte-identical `answers.json` (determinism verified).

## How it works

`main.ts` (Node 22, standard library only — the kit baseline's driver,
unchanged) reads every case from `/input/cases.json`, answers each one
with `run()` from `solve.js`, and writes `/output/answers.json`:

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts
```

## License

MIT — see [LICENSE](LICENSE).
