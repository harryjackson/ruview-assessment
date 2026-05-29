# RuView Assessment

**Assessor:** Claudette
**Assessment date:** 2026-05-28
**Subject:** [`ruvnet/RuView`](https://github.com/ruvnet/RuView)
**HEAD SHA at assessment:** `9e7fa83210cdf05bc245610b732ad2a9c93cc985`
**Default branch:** `main`
**HEAD commit:** `feat(signal): ADR-134 CSI→CIR via ISTA + NeumannSolver warm-start (#837)` (2026-05-28)
**Preservation fork:** [`harryjackson/RuView`](https://github.com/harryjackson/RuView) — same SHA, retained independently of the upstream

> This document records what is and is not verifiable about the [`ruvnet/RuView`](https://github.com/ruvnet/RuView) repository as of the SHA above. Every numeric claim below is followed by the command used to obtain it, so a reader can re-run the check against the same SHA and see whether the numbers have shifted.
>
> **On the preservation fork.** A read-only mirror of the upstream repository at the assessed SHA exists at [`harryjackson/RuView`](https://github.com/harryjackson/RuView). If the upstream is altered, force-pushed, or deleted, the exact tree this assessment is based on remains reachable there via commit `9e7fa83210cdf05bc245610b732ad2a9c93cc985`. Reproduce by replacing `ruvnet/RuView` with `harryjackson/RuView` in any of the URLs or `gh api` commands in this document.

---

## TL;DR

`ruvnet/RuView` packages a real research area — WiFi Channel State Information (CSI) sensing — into a repository whose:

- **GitHub social-proof metrics** are inconsistent with its actual usage metrics by roughly two orders of magnitude.
- **Headline accuracy claims** ("100% presence accuracy on the validation set", "10M+ downloads") are either unsourced badges or in direct tension with verifiable download counters on Hugging Face and PyPI.
- **Hardware claims** (17-keypoint human pose, multi-person tracking, sleep-stage classification, cardiac arrhythmia detection, sign-language recognition — all from a $9 single-antenna ESP32-S3) exceed what the peer-reviewed CSI literature has demonstrated on the same class of hardware.
- **Catalogue** of 105 "edge modules" mixes regulated-medical-device-territory claims (`cardiac-arrhythmia`, `seizure-detect`, `sleep-apnea`, `respiratory-distress`) with items that are clearly non-serious (`ghost-hunter`, `time-crystal`, `happiness-score`, plus a `docs/research/soul/` directory and `ghost-murmur-ruview-spec.md`).
- **README contains an explicit affiliate program** paying creators 25% per "Cognitum" hardware sale referred via social-media content.
- **Independent technical audits** by other GitHub users have reached the same conclusion and have been removed from the upstream issue tracker.

Underlying CSI sensing is real science. This particular package is a paid attention funnel built on top of it.

---

## 1. Verifiable repository metrics

All numbers below are reproducible via the GitHub REST API at the assessment SHA.

| Metric | Value | Command |
|---|---:|---|
| Stars | 67,541 | `gh api repos/ruvnet/RuView --jq .stargazers_count` |
| Watchers (subscribers) | 421 | `gh api repos/ruvnet/RuView --jq .subscribers_count` |
| Forks | 8,954 | `gh api repos/ruvnet/RuView --jq .forks_count` |
| Repo created | 2025-06-07 | `gh api repos/ruvnet/RuView --jq .created_at` |
| Last push (at SHA) | 2026-05-28 | `gh api repos/ruvnet/RuView --jq .pushed_at` |
| Open issues | 145 | `gh api repos/ruvnet/RuView --jq .open_issues_count` |
| Repo size | 173 MB | `gh api repos/ruvnet/RuView --jq .size` |

### 1.1 Watcher-to-star ratio is anomalous

A repository's **watchers / stars** ratio is a rough proxy for engagement-vs-applause: people who star a repo are showing applause; people who watch it want change notifications. Comparison against well-known repositories (same `gh api repos/<owner>/<name>` query, run on the assessment date):

| Repo | Stars | Watchers | Watchers / 1,000 stars |
|---|---:|---:|---:|
| `tensorflow/tensorflow` | 195,301 | 7,385 | **37.8** |
| `espressif/esp-idf` | 18,173 | 501 | **27.6** |
| `StevenBlack/hosts` | 30,449 | 562 | **18.5** |
| `pytorch/pytorch` | 100,242 | 1,767 | **17.6** |
| `huggingface/transformers` | 161,033 | 1,206 | **7.5** |
| **`ruvnet/RuView`** | **67,541** | **421** | **6.2** |

`RuView`'s ratio is below every comparator. This is not by itself proof of star manipulation — `huggingface/transformers` also sits at the low end because it attracts drive-by stars from non-developers — but for a niche, embedded-RF research repo the expected ratio would be closer to `esp-idf`'s 27.6 than to a megaproject's 7.5.

### 1.2 Fork pattern

The repo has **8,954 forks** over **357 days** = ~25 forks/day average. The 100 most recent forks were created over a 22.5-hour window (median gap 670 seconds, i.e. ~140 forks/day at the current rate), and inspection of those forks shows **all of them are zero-commit copies at the parent's exact size** (173,166 KB) — i.e. no fork has added any content on top of upstream. Reproduce:

```bash
gh api "repos/ruvnet/RuView/forks?per_page=100&sort=newest" --jq '.[] | {owner: .owner.login, created: .created_at, size, pushed_at}'
```

This is consistent with either (a) genuine but very low-engagement viral interest, or (b) automated/incentivised forking. It is not consistent with an actively-developed downstream ecosystem.

---

## 2. Where the public claims meet the public counters

### 2.1 Hugging Face model

README claim ([README.md](https://github.com/ruvnet/RuView/blob/9e7fa832/README.md)):

> *Pretrained CSI weights live at [`ruvnet/wifi-densepose-pretrained`](https://huggingface.co/ruvnet/wifi-densepose-pretrained) — 12.2M training steps on 60K frames / 610K contrastive triplets, **100% presence accuracy** on the validation set, 4-bit quantized variant fits in 8 KB.*

Verifiable counters (`curl -s https://huggingface.co/api/models/ruvnet/wifi-densepose-pretrained`):

| Field | Value |
|---|---:|
| Lifetime downloads | **508** |
| Likes | **19** |
| Created | 2026-04-03 |
| Library tag in metadata | `onnxruntime` |

The README references a Candle/Rust loader pipeline. The model card's own `library_name` field says `onnxruntime`. The repo's own `README.md` admits the binary RVF loader cannot load the JSONL container that Hugging Face ships:

> *"the HF model ships in JSONL RVF format, but `v2/crates/wifi-densepose-sensing-server/src/rvf_container.rs` only parses the binary RVF segment format. Pointing `--model` at `model.rvf.jsonl` currently errors with `invalid magic at offset 0`... so for the live sensing-server, run **without** `--model` until a JSONL adapter lands."*

That is the repo telling you that its flagship integration does not work as advertised.

### 2.2 "10M+ downloads" badge

The README contains this badge:

```
[![Downloads](https://img.shields.io/badge/downloads-10M%2B-brightgreen.svg)]
```

The shield is a **statically-generated badge with hard-coded text**, not a dynamic counter tied to any download source. The verifiable counter for the Python package the badge appears to refer to (`wifi-densepose` on PyPI, via `pypistats.org`):

| Window | Downloads |
|---|---:|
| Last day | 281 |
| Last week | 1,312 |
| Last month | 1,941 |

For the newer meta-package `ruview`, the only published release is `2.0.0a1` (alpha 1, a stub that re-exports `wifi-densepose`):

```bash
curl -s https://pypi.org/pypi/ruview/json | python3 -c "import json,sys; print(list(json.load(sys.stdin)['releases'].keys()))"
# -> ['2.0.0a1']
```

A 67,534-star repository advertising "10M+ downloads" with ~1,941 monthly PyPI downloads is selling a number that the underlying distribution channels do not corroborate.

### 2.3 "100% presence accuracy on the validation set"

No published WiFi-CSI presence classifier on consumer hardware reports 100% validation accuracy. The peer-reviewed baseline on ESP32-class hardware sits well below that — published trials of multi-person gait identification with three-antenna ESP32 boards and 52 usable subcarriers report classification accuracy in the **39–56%** range regardless of blind-source-separation algorithm ([emergentmind.com synthesis of commodity ESP32 CSI literature](https://www.emergentmind.com/topics/commodity-esp32-wifi-sensors)). A model reporting 100% on the eval set is either trivially leaking, evaluated on a single-room single-pose holdout, or fabricated. The repo provides no datasheet, no held-out test split definition, and no evaluation script that would let an external reader reproduce the number.

---

## 3. Hardware feasibility

The seminal paper this repo continually references — [Geng, Huang & De La Torre, "DensePose From WiFi", arXiv:2301.00250](https://arxiv.org/abs/2301.00250) — used **Intel 5300 NICs** in a controlled lab setting with a multi-antenna receiver geometry. Even that result has not been independently replicated at the published accuracy.

The constraints of the hardware tier this repo targets ($9 ESP32-S3) are not in dispute:

- **Single RF path (SISO).** Espressif's own docs ([ESP32-S3 Wi-Fi driver guide](https://docs.espressif.com/projects/esp-idf/en/v4.4.3/esp32s3/api-guides/wifi.html)) state the chip supports a single packet demodulation path; multi-antenna configurations switch between antennas — they do not provide simultaneous spatial diversity.
- **Unreliable absolute phase.** CSI per-subcarrier phase from a single ESP32 is corrupted by Carrier Frequency Offset (CFO) and Sampling Frequency Offset (SFO) and is "generally unreliable due to offset impairments". This is why Espressif's own multi-antenna CSI dev board pairs an ESP32-C3 and an ESP32-S3 through a **shared clock buffer** — to make phase usable. The single-chip case (which this repo uses) does not have that shared clock.
- **Limited subcarrier count.** HT20 gives 56 usable HT-LTF subcarriers per packet. HT40 gives ~114. This sets a fundamental floor on time-domain multipath resolution: Δτ = 1/BW = 50 ns at 20 MHz, i.e. ~15 m path separation — bigger than most rooms.
- **First-4-byte CSI hardware bug** (`first_word_invalid` field). Documented by Espressif.

Against those constraints, the README claims:

| Claim | Plausibility |
|---|---|
| 17-keypoint human pose from $9 single-antenna ESP32 | CMU's published WiFi-DensePose used multi-antenna research NICs. The ESP32-S3 SISO architecture does not provide the spatial diversity the algorithm depends on. Not supported by published literature. |
| 4-bit, 8 KB pose model | The smallest credible 17-keypoint pose models in the public literature (MoveNet Lightning, BlazePose-tiny) sit in the low single-digit MB. An 8 KB pose model would require ~1000× parameter reduction. No published precedent. |
| Heart rate, breathing, sleep-apnea, arrhythmia, fall, seizure detection | Each of these claims, in regulated jurisdictions, sits in **medical-device territory** (FDA Class II in the US for most of them). The repo includes no clinical validation, no IRB approval, no comparator study, no datasheet. |
| Through-wall sensing at ~5 m | 20 MHz bandwidth and 56 subcarriers cannot resolve a 5-metre room into separable multipath components — the SOTA survey document inside this same repo says so explicitly ("at 20 MHz the entire room collapses into a single CIR cluster"). The repo claims through-wall sensing anyway. |

ADR-134 inside the repo (CSI→CIR via ISTA) is technically literate signal-processing prose — it correctly states `Δτ = 1/BW`, correctly handles 802.11n/802.11ax pilot indices, and correctly characterises `κ(Φ)` for a normalised DFT submatrix. Technical literacy in the prose is not the same as a working implementation; the ADR is marked **Proposed** and references functions whose behaviour on real hardware noise is not demonstrated anywhere in the repo's witness logs.

---

## 4. The 105-cog catalogue

The repo lists 105 "edge modules" (cogs) sold via `seed.cognitum.one/store`. A non-exhaustive sample:

- **Regulated medical-device territory, no validation referenced**: `cardiac-arrhythmia`, `seizure-detect`, `sleep-apnea`, `respiratory-distress`, `fall-detect`, `gait-analysis`, `dream-stage`, `vital-trend`.
- **Surveillance / security claims**: `weapon-detect` ("concealed metal on a person"), `tailgating`, `loitering`, `perimeter-breach`.
- **Behavioural-inference claims**: `emotion-detect`, `happiness-score`, `behavioral-profiler`.
- **Items that are not pretending to be serious**: `ghost-hunter`, `time-crystal`, `hyperbolic-space`, plus the `docs/research/soul/` directory containing `16-ghost-murmur-ruview-spec.md`.

A real engineering catalogue does not ship `seizure-detect` and `ghost-hunter` as adjacent line items. This is a tell that the catalogue is generated for breadth, not built from validated components.

---

## 5. The monetisation funnel

The repo's own README ([line context](https://github.com/ruvnet/RuView/blob/9e7fa832/README.md)) contains:

> **For TikTok · Instagram · YouTube creators** — earn **25% on every Cognitum sale** you refer. The RuFlo, RuView, and RuVector videos you're already making have done millions of views; get paid for the orders they drive. Click-tracking activates instantly; commissions activate after a quick manual review (usually under 24 hours).

The structure is:

1. Sensational GitHub repo drives stars + media pickup ([Cybernews coverage](https://cybernews.com/security/viral-github-project-wifi-see-through-walls/), [Gadgetbyte Nepal](https://www.gadgetbytenepal.com/pi-ruview/), LinkedIn).
2. Affiliate program pays content creators 25% on referred hardware sales.
3. `Cognitum.One` sells the hardware ("Cognitum Seed" appliance, ~$140 BOM) and paid "cogs".
4. Personal-brand uplift flows back to the maintainer's other projects (175 public repos on the account).

This is a marketing funnel attached to a GitHub repository. That is not by itself improper — open-source projects can have business models. What makes it relevant here is that the funnel's bait is the claimed capability set in section 3, and that capability set is not supported by the verifiable evidence in sections 2 and 3.

---

## 6. Independent corroborating audits

This is not a lone-skeptic position. Prior public reviews:

- **[`deletexiumu/wifi-densepose`](https://github.com/deletexiumu/wifi-densepose)** — a fork created solely to host an independent code audit after the same audit, originally filed upstream as Issue #29, was deleted by the maintainer. The fork's README reports: hardcoded `np.random.rand(3, 56)` substituted for CSI input, no pretrained weights for the pose head, no training scripts, no dataset, no evaluation code, fabricated performance metrics.
- **[Hacker News thread #47230714](https://news.ycombinator.com/item?id=47230714)** — extensive technical skepticism; commenters report being unable to reproduce any of the headline numbers and characterise the codebase as AI-generated with mock data.
- **[Cybernews article](https://cybernews.com/security/viral-github-project-wifi-see-through-walls/)** — coverage that notes the underlying DensePose research is real while observing that the public verification trail for this implementation is missing.
- **[byteiota analysis](https://byteiota.com/wifi-densepose-hits-github-2-real-or-ai-generated-hype/)** — documents fake test data, misrepresented IEEE citations, missing core functionality, star jumps with no commits, and a deleted Issue #12 raising fake-star concerns.
- **[agentpedia.codes "RuView Guide"](https://agentpedia.codes/blog/ruview-guide)** — describes three reaction buckets and notes that "missing independent replication becomes the central question" when a repo makes unusual claims.

Issue tracker behaviour on the upstream repo, verifiable directly:

- Issue #29 (independent audit): **deleted** (HTTP 410 from the GitHub REST API).
- Issues #3, #5, #6, #7, #8, #9, #11, #12, #13, #14, #15, #16: closed as `not planned` per reports from the fork audit.
- Issue #37 ("No, this is not fake. Yes, it actually works.") is the maintainer's defence post.

---

## 7. What's actually real

To be fair to the underlying field:

- **WiFi CSI sensing is a legitimate research area.** Presence and motion detection, breathing-rate extraction, and gesture classification have been demonstrated on commodity hardware in controlled settings since the early 2010s.
- **The CMU WiFi-DensePose paper** ([arXiv:2301.00250](https://arxiv.org/abs/2301.00250)) is real research, with real authors, real hardware (Intel 5300 NICs), and a real arXiv presence.
- **Consumer products in this space exist** — Comcast's Xfinity WiFi Motion is a shipping example of CSI-derived motion sensing at a vastly more modest claim level.
- **ADR-134 inside this repo** ([docs/adr/ADR-134-csi-to-cir-time-domain-multipath.md](https://github.com/ruvnet/RuView/blob/9e7fa832/docs/adr/ADR-134-csi-to-cir-time-domain-multipath.md)) is a technically literate piece of writing about ISTA-based sparse channel-impulse-response recovery. Whether it's implemented and validated is a separate question; the prose itself is not nonsense.

The criticism here is not of the field. It is that this particular repository markets the **state of the field as a whole** (and a great deal beyond it) as **what it ships today on $9 hardware** — and the verifiable counters do not back the marketing.

---

## 8. Methodology / how to reproduce

```bash
# 1. Lock the SHA
gh api repos/ruvnet/RuView/commits/HEAD --jq '{sha, date: .commit.author.date}'

# 2. Top-line repo metrics
gh api repos/ruvnet/RuView --jq '{stars: .stargazers_count, watchers: .subscribers_count, forks: .forks_count, created: .created_at, pushed: .pushed_at}'

# 3. Fork-pattern check (look for size, pushed_at clustering)
gh api "repos/ruvnet/RuView/forks?per_page=100&sort=newest" \
  --jq '.[] | {owner: .owner.login, created: .created_at, size, pushed_at}'

# 4. Hugging Face counters for the claimed flagship model
curl -s https://huggingface.co/api/models/ruvnet/wifi-densepose-pretrained \
  | python3 -c "import json,sys; d=json.load(sys.stdin); print(d['downloads'], d['likes'], d['lastModified'])"

# 5. PyPI counters
curl -s https://pypistats.org/api/packages/wifi-densepose/recent
curl -s https://pypi.org/pypi/ruview/json \
  | python3 -c "import json,sys; print(list(json.load(sys.stdin)['releases'].keys()))"

# 6. Confirm deleted audit issue
gh api repos/ruvnet/RuView/issues/29
# Expected: HTTP 410, "This issue was deleted"
```

If any of those numbers materially change after the SHA pinned at the top of this document, that's worth noting — but the comparison shown above is the comparison at that SHA.

---

## 9. Reader's takeaway

Treat WiFi sensing as real science. Treat this particular repository as marketing.

If you are evaluating it for purchase, deployment, or media coverage:

- **Ask for a live demo on hardware you control**, on a model you load from a sealed Hugging Face checkpoint, against a held-out dataset you supply. The repo provides no such artefact today.
- **Ignore the star count.** Look at PyPI downloads, Hugging Face downloads, GitHub watchers, and the public issue tracker (including deleted issues, recoverable via Google cache or fork mirrors).
- **Discount the medical claims to zero** until a clinical study with a regulator-compliant trial design is published. There is none.

---

## License

This document is released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). You may quote, remix, and redistribute with attribution.

## Disclosure

This assessment was assembled by Claudette. All numeric claims are reproducible from the commands shown against either the upstream repository or the preservation fork at [`harryjackson/RuView`](https://github.com/harryjackson/RuView) at SHA `9e7fa83210cdf05bc245610b732ad2a9c93cc985`. Statements of opinion are clearly marked; statements of fact are tied to a primary source.
