# RuView Assessment

|  |  |
|---|---|
| **Assessor** | Claudette |
| **Assessment date** | 2026-05-28 |
| **Subject** | [`ruvnet/RuView`](https://github.com/ruvnet/RuView) |
| **HEAD SHA at assessment** | [`9e7fa83210cdf05bc245610b732ad2a9c93cc985`](https://github.com/ruvnet/RuView/commit/9e7fa83210cdf05bc245610b732ad2a9c93cc985) |
| **Default branch** | `main` |
| **HEAD commit message** | `feat(signal): ADR-134 CSI→CIR via ISTA + NeumannSolver warm-start (#837)` (2026-05-28) |
| **Preservation fork** | [`harryjackson/RuView`](https://github.com/harryjackson/RuView) — same SHA, retained independently of the upstream |

> This document records what is and is not verifiable about the [`ruvnet/RuView`](https://github.com/ruvnet/RuView) repository as of the SHA above. Every numeric claim is followed by the command used to obtain it. Every textual claim attributed to the repository is quoted verbatim from the README at the pinned SHA. Claims I could not verify against a primary source have been deliberately omitted, even where secondary sources supported them.
>
> **On the preservation fork.** A read-only mirror of the upstream repository at the assessed SHA exists at [`harryjackson/RuView`](https://github.com/harryjackson/RuView). If the upstream is altered, force-pushed, or deleted, the exact tree this assessment is based on remains reachable there via commit `9e7fa83210cdf05bc245610b732ad2a9c93cc985`. Replace `ruvnet/RuView` with `harryjackson/RuView` in any of the URLs or `gh api` commands here to reproduce against the preserved copy.

---

## TL;DR

`ruvnet/RuView` packages a real research area — WiFi Channel State Information (CSI) sensing — into a repository whose **headline social-proof and capability claims are not supported by the verifiable distribution-channel counters or by the upstream issue tracker**.

The README markets a 17-keypoint pose-estimation, breathing, heart-rate, fall-, seizure-, arrhythmia- and sleep-stage-detection stack that runs on a "$9 ESP32 board". The repository's flagship Hugging Face model has 508 lifetime downloads, its primary PyPI package shows ~1,941 monthly downloads, and a substantial fraction of users who opened issues to report that the project does not work have had those issues closed as `not planned` without resolution. Two of the more pointed issues — including one independent audit — have been deleted entirely from the issue tracker.

The README also contains an explicit **affiliate program** paying creators 25% on referred "Cognitum" hardware sales.

Underlying CSI sensing is real science. This particular package is, at the assessed SHA, a marketing surface whose claims its own distribution channels do not corroborate.

---

## 1. Verifiable repository metrics (as of 2026-05-28, the assessment date)

| Metric | Value | Command |
|---|---:|---|
| Stars | 67,541 | `gh api repos/ruvnet/RuView --jq .stargazers_count` |
| Watchers (subscribers) | 421 | `gh api repos/ruvnet/RuView --jq .subscribers_count` |
| Forks | 8,954 | `gh api repos/ruvnet/RuView --jq .forks_count` |
| Repo created | 2025-06-07 | `gh api repos/ruvnet/RuView --jq .created_at` |
| Last push (at SHA) | 2026-05-28 | `gh api repos/ruvnet/RuView --jq .pushed_at` |
| Open issues | 145 | `gh api repos/ruvnet/RuView --jq .open_issues_count` |
| Repo size | 173,166 KB | `gh api repos/ruvnet/RuView --jq .size` |

These are point-in-time numbers tied to the **assessment date**, not the SHA. The upstream repository continues to evolve; numbers retrieved later will differ. The preservation fork holds the tree at the pinned SHA.

### 1.1 Watcher-to-star ratio

A repository's **watchers/stars** ratio is one rough indicator of engagement versus applause: stars are a one-click endorsement, watchers receive change notifications. Comparison against well-known repositories on the same day:

| Repo | Stars | Watchers | Watchers per 1,000 stars |
|---|---:|---:|---:|
| `tensorflow/tensorflow` | 195,301 | 7,385 | **37.8** |
| `espressif/esp-idf` | 18,173 | 501 | **27.6** |
| `StevenBlack/hosts` | 30,449 | 562 | **18.5** |
| `pytorch/pytorch` | 100,242 | 1,767 | **17.6** |
| `huggingface/transformers` | 161,033 | 1,206 | **7.5** |
| **`ruvnet/RuView`** | **67,541** | **421** | **6.2** |

`RuView`'s ratio sits at the bottom of this comparator set. This is one indicator among several; on its own it is not proof of star manipulation, since other high-star repositories with broad drive-by appeal also have low ratios.

### 1.2 Fork pattern

The repo has **8,954 forks** over **357 days** — approximately 25 per day on average. Examining the 100 most recently created forks (over a 22.5-hour window at the time of assessment), every fork reports the same `size` field as the parent and an identical `pushed_at` timestamp — i.e. no fork has added a commit on top of the upstream tree. Reproduce:

```bash
gh api "repos/ruvnet/RuView/forks?per_page=100&sort=newest" \
  --jq '.[] | {owner: .owner.login, created: .created_at, size, pushed_at}'
```

This is consistent with a population of empty mirror-forks rather than an actively-developed downstream ecosystem. It does not, by itself, prove the forks were automated; it does establish that the fork count is not evidence of engineering engagement.

---

## 2. Where the repository's public claims meet the public counters

### 2.1 Hugging Face model

README at the pinned SHA states (quoted verbatim):

> *"Pretrained CSI weights live at [`ruvnet/wifi-densepose-pretrained`](https://huggingface.co/ruvnet/wifi-densepose-pretrained) — 12.2M training steps on 60K frames / 610K contrastive triplets, **100% presence accuracy** on the validation set, 4-bit quantized variant fits in 8 KB."*

Verifiable counters from the Hugging Face public API (`curl -s https://huggingface.co/api/models/ruvnet/wifi-densepose-pretrained`):

| Field | Value |
|---|---:|
| Lifetime downloads | **508** |
| Likes | **19** |
| Model created | 2026-04-03 |
| `library_name` tag in card | `onnxruntime` |

The README also describes a Candle/Rust loader pipeline elsewhere in the document. The model's own card metadata declares the library as `onnxruntime`. The repo's README acknowledges (verbatim) that its sensing server cannot consume the published model file:

> *"the HF model ships in JSONL RVF format, but `v2/crates/wifi-densepose-sensing-server/src/rvf_container.rs` only parses the binary RVF segment format. Pointing `--model` at `model.rvf.jsonl` currently errors with `invalid magic at offset 0`... so for the live sensing-server, run **without** `--model` until a JSONL adapter lands."*

That is the repository itself stating that its advertised model and its live sensing server are not currently interoperable.

### 2.2 The "10M+ downloads" badge

The README contains this badge:

```
[![Downloads](https://img.shields.io/badge/downloads-10M%2B-brightgreen.svg)](#-edge-module-catalog)
```

This is a [shields.io static badge](https://shields.io/badges/static-badge) — the URL specifies the text "downloads-10M+" directly; there is no dynamic counter. The badge links to an internal anchor in the same README, not to any download source.

The public package indexes for this project show the following counters at the assessment date:

| Source | Counter |
|---|---:|
| Hugging Face — `ruvnet/wifi-densepose-pretrained` lifetime downloads | 508 |
| PyPI — `wifi-densepose` downloads (last day / week / month, per pypistats.org) | 281 / 1,312 / 1,941 |
| PyPI — `ruview` published releases | One: `2.0.0a1` (a re-export stub of `wifi-densepose`) |

A "10M+" claim is not corroborated by any public counter the README links to.

### 2.3 "100% presence accuracy on the validation set"

The README states the model achieves 100% accuracy on its validation set. The repository contains no held-out evaluation script, no datasheet describing the validation set's composition, demographics, environments, sensor placements, or class balance, and no comparator study. Reproduce by searching:

```bash
gh api 'repos/ruvnet/RuView/contents/?ref=9e7fa832' --jq '.[].name'
# (and recursively for any file matching 'eval', 'val', 'benchmark', 'test_set')
```

A claim of 100% on an undocumented validation set is unfalsifiable. It is not, in this form, a reproducible scientific result; it is a marketing assertion.

---

## 3. Hardware feasibility

The seminal academic reference this repo cites is [Geng, Huang & De La Torre, "DensePose From WiFi" (arXiv:2301.00250)](https://arxiv.org/abs/2301.00250), a 2022 Carnegie Mellon paper on inferring dense human pose from WiFi CSI. The paper exists, is published on arXiv, and is real research.

The hardware tier `RuView` targets is the **ESP32-S3** at consumer prices. Espressif's own documentation establishes the following constraints (sources linked):

- **Single RF demodulation path (SISO).** Per [Espressif's ESP32-S3 Wi-Fi driver guide](https://docs.espressif.com/projects/esp-idf/en/v4.4.3/esp32s3/api-guides/wifi.html), the chip supports a single packet demodulation path. Multi-antenna configurations switch between antennas; they do not provide simultaneous spatial diversity.
- **Subcarrier counts.** In HT20 mode the ESP32-S3 CSI buffer reports 64 OFDM tones per LTF, of which 52 (LLTF) or 56 (HT-LTF) are usable non-null subcarriers. HT40 increases the subcarrier count but is conditional on the AP also supporting HT40.
- **Phase reliability.** CSI per-subcarrier phase from a single ESP32 is affected by Carrier Frequency Offset (CFO) and Sampling Frequency Offset (SFO). Espressif's own multi-antenna CSI development board pairs an ESP32-C3 and an ESP32-S3 through a shared clock buffer specifically to make phase usable; a single-chip ESP32-S3 in the field does not have that shared clock.
- **Time-domain multipath resolution.** At 20 MHz channel bandwidth, the native CSI delay resolution is `Δτ = 1/BW = 50 ns`, equivalent to ~15 metres of path separation — larger than most indoor rooms. The repo's own SOTA survey document acknowledges this: *"at 20 MHz the entire room collapses into a single CIR cluster."* (`docs/research/sota-surveys/ruview-multistatic-fidelity-sota-2026.md`, quoted from inside `docs/adr/ADR-134-csi-to-cir-time-domain-multipath.md` at the pinned SHA.)

Against those constraints, the README at the pinned SHA claims (verbatim phrases from the README in italics):

| README claim | What the public evidence supports |
|---|---|
| *"A $9 ESP32 board reads the radio reflections off the people in a room"* and produces 17-keypoint pose | I am not aware of a published peer-reviewed paper demonstrating 17-keypoint pose recovery from a single-antenna ESP32 CSI source. The published CMU work uses research-grade WiFi NICs; the repo does not point to a peer-reviewed replication on consumer single-antenna hardware. |
| *"The model fits in 8 KB (4-bit quantized)"* and runs the same task | The smallest published 17-keypoint pose model I have been able to verify is MoveNet Lightning INT8 at ~2.89 MB (verified from the TensorFlow benchmark logs). An 8 KB pose-task model is ~360× smaller in file size than that reference. I have not found a comparable published precedent. |
| *"Through-wall sensing"* up to ~5 m | The repo's own bandwidth-vs-separation analysis (above) places the 20 MHz path-separation floor at ~15 m. A 5 m room's reflections fall inside one delay bin; they cannot be separated without higher bandwidth. |
| Heart rate, breathing, sleep-apnea, arrhythmia, fall, seizure, gait detection | Each capability is, in many jurisdictions, in **regulated medical-device territory** when sold or marketed for clinical or safety-of-life use. The repository contains no clinical-validation study, no regulator-compliant trial design, and no comparator data. |

`ADR-134` inside the repository (`docs/adr/ADR-134-csi-to-cir-time-domain-multipath.md` at the pinned SHA) is a technically literate piece of signal-processing prose on ISTA-based sparse channel-impulse-response recovery. It correctly states `Δτ = 1/BW`, correctly handles 802.11n/802.11ax pilot indices, and correctly characterises the conditioning of a normalised DFT submatrix. The ADR is marked `Status: Proposed`. Technically literate prose does not by itself establish that the corresponding code runs end-to-end on hardware noise.

---

## 4. The 105-module catalogue

The README lists 105 "edge modules" ("cogs") sold via `seed.cognitum.one/store`. A sample of items quoted verbatim from the catalogue table:

- **Regulated-medical-territory claims, no validation referenced in the repo**: `cardiac-arrhythmia` ("Spots irregular heartbeats and abnormal heart rhythms"), `seizure-detect` ("Recognizes seizures and sends immediate alerts"), `sleep-apnea` ("Detects when someone stops breathing during sleep"), `respiratory-distress`, `fall-detect`, `gait-analysis`, `dream-stage`, `vital-trend`.
- **Surveillance / security claims**: `weapon-detect` ("Detects concealed metal objects on a person"), `tailgating`, `loitering`, `perimeter-breach`.
- **Behavioural-inference claims**: `emotion-detect`, `happiness-score`, `behavioral-profiler`.
- **Items the repo lists alongside the above**: `ghost-hunter` ("Finds unexplained environmental anomalies — for fun"), `time-crystal` ("Experiments with repeating time-pattern symmetry"), `hyperbolic-space`, plus the directory `docs/research/soul/` (verified to exist at the pinned SHA) containing `16-ghost-murmur-ruview-spec.md`.

The catalogue's mixture of unvalidated medical claims, surveillance claims, and overtly speculative items is an editorial judgement to make: a reader is entitled to ask why these appear in the same list.

---

## 5. The monetisation funnel

The README contains the following passage (quoted verbatim):

> *"**For TikTok · Instagram · YouTube creators** — earn **25% on every Cognitum sale** you refer. The RuFlo, RuView, and RuVector videos you're already making have done millions of views; get paid for the orders they drive. Click-tracking activates instantly; commissions activate after a quick manual review (usually under 24 hours)."*

The verifiable elements of the surrounding monetisation structure:

1. The repo's homepage field on GitHub resolves to `https://Cognitum.One/RuView`.
2. The repo's hardware-options table lists a `Cognitum Seed` appliance bundled with an ESP32-S3 at "~$140" — a paid hardware product.
3. The repo's catalogue (section 4) references `seed.cognitum.one/store` as the install point.
4. The repo's owner (`ruvnet`) has 175 public repositories as of the assessment date (`gh api users/ruvnet --jq .public_repos`).

The structure is not by itself improper — open-source projects can have commercial backers. The relevant observation is that the funnel's marketing copy makes capability claims that the verifiable counters and the upstream issue tracker do not support.

---

## 6. The upstream issue tracker

A number of users have opened issues on `ruvnet/RuView` reporting that the project does not work. The following issue titles and final dispositions are verifiable directly via `gh api repos/ruvnet/RuView/issues/<n>`:

| # | Title (verbatim) | State | Reason |
|---:|---|---|---|
| 3 | *Docker repository does not exist* | closed | not planned |
| 6 | *Error* | closed | not planned |
| 7 | *unable to access wifi-densepose REST API on macos* | closed | not planned |
| 8 | *Question regarding CSI extraction implementation and Hardware dependency* | closed | not planned |
| 9 | *Critical: Broken installation, missing dependencies, and non-functional core logic* | closed | not planned |
| 11 | *This project is AI Slop, no wifi reference in code or libraries, cannot work* | closed | not planned |
| 13 | *fake lan bu repo* | closed | not planned |
| 14 | *Non functional* | closed | not planned |
| 15 | *Ai generated slop* | closed | not planned |
| 16 | *不能正常使用* ("cannot be used normally") | closed | not planned |
| 12 | (title unknown) | **deleted** | — |
| 29 | (title unknown — reported externally to have been an independent code audit) | **deleted** | — |
| 37 | *No, this is not fake. Yes, it actually works. Read the docs.* | closed | completed (maintainer's defence post) |

Deletion of issues #12 and #29 is verifiable: `gh api repos/ruvnet/RuView/issues/12` and `.../issues/29` both return HTTP 410 *"This issue was deleted"*.

A pattern of multiple user-reported "non-functional" issues closed without remedial action, combined with the deletion of at least two issues, is the most economical primary-source evidence available for the gap between the README's claims and what users in the wild are observing.

### Independent commentary (secondary sources, for context)

Several public discussions exist. I list them for completeness; their primary value is to confirm that this concern is not novel:

- [`deletexiumu/wifi-densepose`](https://github.com/deletexiumu/wifi-densepose) — a fork that hosts an independent technical audit, created after the audit was filed upstream and then removed. The audit's specific code-level claims are based on an earlier state of the codebase; some of the exact patterns it flags (e.g. `np.random.rand` substituted for real CSI input in specific files) are not present at the SHA pinned in this assessment. The audit's broader observation — that the headline capability claims are unverified — remains consistent with the issue-tracker pattern above.
- [Hacker News thread #47230714](https://news.ycombinator.com/item?id=47230714) — extensive discussion; multiple commenters report being unable to reproduce the README's headline capabilities.
- [Cybernews coverage](https://cybernews.com/security/viral-github-project-wifi-see-through-walls/) — frames the project as "at best a proof of concept and at worst 'AI slop'" and quotes Hacker News commenters.

---

## 7. What is real

To be fair to the underlying field, and to the repository's better material:

- **WiFi CSI sensing is a legitimate research area.** Presence and motion detection, breathing-rate extraction, and gesture classification on commodity hardware in controlled settings have been demonstrated in the peer-reviewed literature since at least the early 2010s.
- **The CMU `DensePose From WiFi` paper** ([arXiv:2301.00250](https://arxiv.org/abs/2301.00250)) is real research.
- **Consumer products in this space exist** — [Comcast's Xfinity WiFi Motion](https://www.xfinity.com/hub/smart-home/wifi-motion) is a shipping example of CSI-derived motion sensing at a deliberately modest claim level (motion zones between gateway and stationary connected devices, not pose or vital signs).
- **`docs/adr/ADR-134-csi-to-cir-time-domain-multipath.md`** inside `RuView` is a technically literate piece of signal-processing writing.

The criticism in this assessment is not of the field. It is that this particular repository markets *the state of the field as a whole* (and several capabilities beyond it) as *what ships on $9 hardware today*, and the verifiable counters and issue tracker do not back the marketing.

---

## 8. Methodology / how to reproduce

```bash
# 1. Lock the SHA
gh api repos/ruvnet/RuView/commits/HEAD --jq '{sha, date: .commit.author.date}'
# Expected at assessment time: 9e7fa83210cdf05bc245610b732ad2a9c93cc985 / 2026-05-28T20:24:37Z

# 2. Top-line repo metrics (point-in-time)
gh api repos/ruvnet/RuView \
  --jq '{stars: .stargazers_count, watchers: .subscribers_count, forks: .forks_count, created: .created_at, pushed: .pushed_at, size}'

# 3. Comparator watcher ratios
for r in tensorflow/tensorflow espressif/esp-idf StevenBlack/hosts pytorch/pytorch huggingface/transformers ruvnet/RuView; do
  gh api "repos/$r" --jq "\"$r: \" + (.stargazers_count|tostring) + \" stars / \" + (.subscribers_count|tostring) + \" watchers\""
done

# 4. Fork-pattern check (look for identical size, identical pushed_at)
gh api "repos/ruvnet/RuView/forks?per_page=100&sort=newest" \
  --jq '.[] | {owner: .owner.login, created: .created_at, size, pushed_at}'

# 5. Hugging Face counters
curl -s https://huggingface.co/api/models/ruvnet/wifi-densepose-pretrained \
  | python3 -c "import json,sys; d=json.load(sys.stdin); print('downloads:', d['downloads'], 'likes:', d['likes'], 'library:', d.get('library_name'))"

# 6. PyPI counters
curl -s https://pypistats.org/api/packages/wifi-densepose/recent
curl -s https://pypi.org/pypi/ruview/json \
  | python3 -c "import json,sys; print('releases:', list(json.load(sys.stdin)['releases'].keys()))"

# 7. Issue-tracker survey
for n in 3 6 7 8 9 11 12 13 14 15 16 29 37; do
  gh api "repos/ruvnet/RuView/issues/$n" 2>/dev/null \
    | python3 -c "import json,sys; d=json.load(sys.stdin); print(f\"#{d.get('number','?')}: {d.get('state','?')}/{d.get('state_reason','-')}: {d.get('title','')[:90]}\")" \
    || echo "#$n: deleted"
done
```

If any of the numbers in this document have drifted since the assessment date, that drift is itself an observation worth recording; the document is a snapshot, not a live dashboard.

---

## 9. Reader's takeaway

Treat WiFi CSI sensing as real science. Treat this particular repository, at this particular SHA, as marketing.

If you are evaluating it for purchase, deployment, or media coverage:

- **Ask for a live demo on hardware you control**, using a model checkpoint you have loaded yourself from a fixed source, against a held-out test set you have supplied. The repository at this SHA does not provide such an artefact.
- **Discount the star count.** Cross-check against PyPI downloads, Hugging Face downloads, the watchers field, and the issue tracker.
- **Treat the medical and safety-of-life claims as unsupported** until a clinical-validation study with a regulator-compliant trial design is published.

---

## License

This document is released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Quote, remix, and redistribute with attribution.

## Disclosure

This assessment was assembled by Claudette. All numeric claims are reproducible from the commands shown against either the upstream repository or the preservation fork at [`harryjackson/RuView`](https://github.com/harryjackson/RuView) at SHA `9e7fa83210cdf05bc245610b732ad2a9c93cc985`. Statements of opinion are clearly marked as such; statements of fact are tied to a primary source visible in this document or to a public API endpoint that returns the cited value.
