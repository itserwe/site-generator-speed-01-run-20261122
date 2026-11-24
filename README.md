[![make.fun](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/itserwe/site-generator-speed-01-run-20261122/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/itserwe/site-generator-speed-01-run-20261122/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)

# Faster makefun-site

A faster drop-in replacement for makefun-site 1.0.0: `solve.js` is a dependency-free
ES module whose `run(input)` returns exactly the answer makefun-site 1.0.0 returns for
every input — the output tree of a site build, byte for byte — and `main.ts` is the
harness driver that reads every case from `/input/cases.json` and writes
`/output/answers.json`.

## How it is faster

The reference spends its time escaping HTML one character at a time inside its inline
Markdown renderer, building throwaway token objects while lexing, and normalising
every slug through Unicode NFKD. This implementation:

- renders Markdown in a single pass over the lines, with no token objects;
- escapes plain text runs in bulk, stopping only at the four characters that can open
  an inline construct;
- takes an ASCII fast path for slugs and memoises the pure slug function within a run;
- renders templates into one chunk array joined once, instead of concatenating strings
  node by node;
- counts words with a direct scan instead of allocating a word list.

No WebAssembly, no native addon, no prebuilt binary, no network, no clock, no
randomness: the output is a function of the input alone, and every case is answered
alone (the slug memo is cleared at the start of each `run()`).

## Files

- `main.ts` — the harness driver (reads cases, writes answers).
- `solve.js` — the fast implementation (`run(input)`).
- `LICENSE` — MIT.

## Local check

With the kit unpacked, run the entry on the practice set and score it:

```sh
MAKEFUN_INPUT_DIR=<kit>/practice/input MAKEFUN_OUTPUT_DIR=/tmp/out node main.ts
MAKEFUN_OUTPUT_DIR=/tmp/out MAKEFUN_REFERENCE_DIR=<kit>/practice/reference \
  MAKEFUN_SCORES_DIR=/tmp/scores node <kit>/program/scorer/main.ts
```

`scores.json` should read `{"passed":1,"cases_passed":3,"cases_total":3}`.
