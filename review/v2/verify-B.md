<!-- CRITIC B · mimo-v2.5-free · family:xiaomi · pass 3 · 2026-08-28T16:53:30Z -->
CRITIC: mimo-v2.5-free (family xiaomi, actor mimo-v2.5-free@opencode-zen)
DATE: 2026-08-28
PASS: 3
AUTO-TALLIED VERDICT: PUBLISH

---

# Critic review — rogerai-labs--linux-for-language-models v2

```
CRITIC:    opencode/mimo-v2.5-free · Xiaomi MiMo · OpenCode 1.18.23 / OpenCode Zen (Seat B)
DATE:      2026-08-28
PASS:      3 (verification)
READ:      delta (v1->v2, chapter 3)
```

## Verdict summary

All four Pass-2 blocking findings are resolved or fairly rebutted in the v2 delta. C-1 (MemFree) receives a precise rewrite that is both textually corrected and substantively superior to the original — it names the allocator's free lists, distinguishes MemFree from Cached/Buffers, and is verbatim supported by proc(5). C-2 (uptime padding) is rebutted with a clear argument that uptime calibrates every boot-relative accumulator in the chapter, which is factually correct. C-3 (race-condition caveat) is fixed at both the CPU and diskstats instances with technically sound, non-hand-wavy guidance: measure the gap rather than trust it, lengthen the gap to shrink the error, do not attempt to lock reads. C-4 (missing deliberate-sampling example) is fully resolved by the bounded-burst-sampler listing, which is a well-designed, runnable example with a real transcript that demonstrates its diagnostic value. The revisions introduce no new blocking debt: the awk command in the burst sampler is syntactically correct and produces the claimed output; the jq parenthetical is appropriately hedged; the loadavg parenthetical about CPU counts is a genuine calibration point, not padding. The fact-check sample confirms all revised claims against the cited kernel documentation. **PUBLISH** — the delta is clean, the fixes are substantive, and no new issues are introduced.

## Blocking findings

None.

## Suggestions (non-blocking)

1. The burst sampler's awk field-separator trick (`-F"avg10="`) is clever but non-obvious. A one-line comment in the code block (e.g., `# split on the "avg10=" delimiter, grab the value`) would help a reader unfamiliar with awk field separators, without changing the listing's runtime behavior.

2. The new race-condition paragraph after diskstats says "Do not try to lock the two reads together" — this is correct advice, but a reader might wonder *why* (the answer: /proc has no transaction API). A brief clause like "there is no kernel mechanism to freeze both counters simultaneously" would close that residual question.

3. The output values in the burst sampler (2.16 down to 0.19) are plausible but unlabeled regarding which machine/day. The surrounding prose says "the authoring machine" but does not date the sample. A dated footnote or "(machine-specific, 2026-08-27)" tag would maintain the chapter's established honesty convention.

## Fact-check sample

| Claim (quoted) | Location | Cited source | Supported? (yes/no/partly) |
|---|---|---|---|
| "`MemFree` counts only pages on the allocator's free lists — the kernel documentation defines it as the sum of the zones' free pages" | ch3, Memory section (revised) | proc(5)/proc_meminfo: "MemFree: The sum of LowFree+HighFree." | yes |
| "`MemAvailable`, an estimate the kernel itself computes and publishes precisely because the naive subtraction misled a generation of monitoring scripts" | ch3, Memory section (revised) | proc(5)/proc_meminfo: "MemAvailable (since 3.14): An estimate of how much memory is available for starting new applications, without swapping." | yes |
| "The fourth field, `50/5997`, is runnable threads over total threads; the fifth is the PID most recently assigned" | ch3, opening shot (unchanged, but contextually referenced by loadavg parenthetical) | proc_loadavg(5): 4th field = "number of currently runnable kernel scheduling entities (processes, threads)"; 5th field = "PID of the process most recently created" | yes |
| "Each `some` line reports the percentage of time… during which *at least one task sat stalled* waiting for that resource" | ch3, Pressure section (unchanged, but directly invoked by burst sampler's `io-some` label) | kernel PSI doc: "the share of time in which at least some tasks are stalled on a given resource" | yes |
| "`sleep 1` guarantees *at least* a second… the uncertainty lives entirely in the gap's length" | ch3, Rates honesty note (new in v2) | POSIX nanosleep: sleep guarantees a minimum, not exact, interval; gap measurement is the standard technique for rate accuracy | yes |
| "each read of a `/proc` counter file is internally consistent" | ch3, Rates honesty note (new in v2) | kernel design: /proc counter files are generated atomically on read; no partial snapshots | yes |

## Scores (1-5)
accuracy: 5 · clarity: 5 · completeness-for-tier: 5 · density: 5 · originality: 4

## Pass-3 only: findings ledger

| Finding # (from Pass 2) | Status: resolved / rebutted-accepted / still-open | Note |
|---|---|---|
| C-1 (MemFree "memory doing nothing") | resolved | Rewritten to name allocator free lists and zones; cites kernel documentation; explicitly refutes the imprecise "not allocated to a process" gloss. Substantively correct. |
| C-2 (uptime padding) | rebutted-accepted | Author argues uptime calibrates every boot-relative accumulator; the v2 text explains this explicitly. The argument is factually sound — counter deltas from /proc/stat and /proc/diskstats are meaningless without knowing the denominator of time since boot. |
| C-3 (race-condition / interval-accuracy caveat) | resolved | Fixed at both instances (CPU section and diskstats section). The advice is technically precise: measure the gap with /proc/uptime, lengthen the gap to reduce relative error, do not attempt to synchronize reads. No hand-waving. |
| C-4 (no concrete deliberate-sampling examples) | resolved | Bounded burst sampler added with working bash code, sample output, and interpretation of a real diagnostic (I/O pressure spike in mid-decay). The example demonstrates exactly the kind of concrete guidance the finding demanded. |
