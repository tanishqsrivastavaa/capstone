# Viva Preparation Notes — First Review

**Purpose:** a question bank and answer outline for the First Review viva, grounded
only in committed repository evidence (`docs/literature-review.md`,
`docs/gantt-plan.md`, `docs/crossover-explained.md`, `docs/deployment-dossier.md`,
`deliverables/review-1/review-1-presenter-guide.md`,
`deliverables/review-1/first-review-package.md`, `NEXT.md`,
`results/baseline/w4/integration_adjudication.json`).

**This is preparation material only.** It does not substitute for the two human
facts the package tracks: all four team members must independently explain the
project (four-member technical readiness), and the guide's dated acknowledgement of
the simulation-first Tier-1/no-required-hardware path must be obtained and recorded —
neither can be fabricated or inferred from this document.

Label every number you quote as **measured**, **bounded smoke**, or **planned**.

---

## Theme A — Problem and motivation

### Q1. What real problem does this project solve?

An edge camera must let a remote server classify what it sees across a link with
limited bandwidth and noise. Today's stack compresses, channel-codes and modulates
to recover *bits/pixels*, while the task metric is classification accuracy after
transmission. The project asks whether sending every image detail is necessary when
the receiver only needs to decide the class.

### Q2. Why is this an AI/ML project and not just a communications one?

The method is a supervised end-to-end learned system: a residual CNN encoder and a
dual-head decoder trained together through a differentiable AWGN channel with a
`cross-entropy + λ·MSE` loss. The evaluation task is image classification with a
frozen ResNet-18 reference classifier. It is supervised representation learning —
not reinforcement learning and not an LLM system.

### Q3. Where does prior work leave this problem? What is your gap?

Four families are synthesized in `docs/literature-review.md` (30 references):
finite-blocklength theory explains why short practical links pay overhead; learned
compression improves reconstruction trade-offs, not necessarily task accuracy;
DeepJSCC shows continuous learned transmission with graceful degradation but mostly
targets reconstruction; task-oriented communication studies useful features without
isolating task awareness from joint coding. The carried gap is a fair three-way
image-classification comparison that retains failures and separates those two effects.

---

## Theme B — Methodology: DJSCC vs separation

### Q4. Explain deep joint source-channel coding in plain language.

In conventional design, compression and error correction are optimized separately
from the receiver-side AI objective (Shannon separation). In DJSCC the sender's
encoder and the receiver's decoder/classifier train together through a
differentiable noisy channel, so backpropagation learns a transmitted
representation that preserves inference information after noise. Separation is only
optimal for infinitely long messages; short messages over noisy links pay
finite-blocklength penalties where joint coding may gain.

### Q5. If the learned system wins, doesn't that prove joint coding helps?

No — the gap conflates task-aware representation with joint source-channel coding.
That is why the experiment has three arms: classical image link (reference),
a digital feature control (ER-9) that sends quantized learned features over the
same LDPC/QPSK chain at matched k, and the learned joint arm. Only the third-arm
comparison beyond the second attributes a difference to joint coding itself.

### Q6. Why not just transmit the class label?

That moves the whole classifier to the sender and answers a different engineering
question. The deployment split fixes a small encoder at the camera and the decoder
plus classifier at the receiver, which is the realistic edge/cloud placement.

### Q7. Is this supervised or reinforcement learning?

Supervised end-to-end training through a differentiable channel model. There is no
agent, action space or reward; the loss is cross-entropy plus λ times MSE on
reconstruction. λ calibration is a later gated step (G-4) and has not been performed.

---

## Theme C — Baseline fairness

### Q8. Isn't your classical baseline deliberately weak?

No. It tunes JPEG 2000 quality, LDPC rate and modulation per SNR on validation data,
and may climb from QPSK to 16-QAM as the link cleans up — exactly as deployed radios
do (DEC-16). Every failure stays in the accuracy denominator, and exact byte and
physical-layer identity accounting prevents hidden overhead from helping any arm.

### Q9. Why JPEG 2000 and not JPEG?

JPEG's roughly 250–290 byte container floor is a large or total fraction of the
smallest channel budgets here, so a low-SNR cliff would be a file-format artifact
rather than a coding result (DEC-9). JPEG 2000 raw codestreams via OpenJPEG 2.5.4
are the preregistered headline codec; JPEG remains a labelled secondary curve.

### Q10. How do you know your LDPC implementation is correct?

G-2 measured BLER waterfalls for K=128/N=256, base graph 2, lifting size 22, rate 1/2,
offset min-sum (offset 0.5), 50 iterations at four SNR points per modulation with
5,000 trials per point, and all golden-vector and independent waterfall comparisons
passed the ≤0.5 dB displacement tolerance (`results/baseline/g2/g2_adjudication.json`;
see `docs/gantt-plan.md`). Anything outside the characterized identity returns an
explicit `uncharacterized` verdict and is ineligible rather than low-scoring.

### Q11. What happens when a transmission fails outright?

It is never skipped or scored as zero silently. The pipeline returns one of four
verdicts — structural infeasibility, codec infeasibility, decode failure or
delivered — and failed transmissions remain in the denominator. The bounded W4 smoke
deliberately included a −8 dB decode-failure cell alongside delivered cells.

---

## Theme D — Hypotheses (H1/H2 and the corrected H1 wording)

### Q12. State your primary hypothesis exactly.

H1 compares learned vs `classical_adaptive` at the headline ratio using paired
per-image outcomes: a point qualifies when the studentized paired mean exceeds 1.96;
`R_obs` is the longest consecutive qualifying run at or below the training SNR; H1 is
supported only if `R_obs ≥ 3` **and** the calibrated run p-value is at most 0.05.
The whole low-SNR-region mean paired difference is the effect size of record.

### Q13. Doesn't your success criterion require the curves to cross?

No — that was corrected. A crossover is reported if observed but is **not required**
(`docs/crossover-explained.md`): at low bandwidth ratios the learned system may
dominate everywhere, and requiring a crossing would treat stronger learned
performance as failure. Completion depends on running the preregistered protocol
correctly, never on the outcome.

### Q14. What if H1 is not supported?

Support, no support and adverse findings are all valid outcomes reported under the
same protocol. A null result triggers no retuning on the test split. The completion
criterion is proper execution of the fixed protocol, not a favorable result.

### Q15. What are H2–H4 about?

They are the preregistered companions in `spec/SPEC.md` §2: graceful degradation
relative to the classical cliff, behavior across bandwidth ratios, and statistical
power (declared before the test split opens). All hypotheses use paired per-image
outcomes; choices freeze before the single test campaign at gate G-12.

---

## Theme E — Results so far

### Q16. What evidence do you actually have today?

Measured feasibility gates, none of which is a final comparison:
G-1 — ResNet-18 trained from scratch reached 898/1000 = 89.8% validation top-1
(epoch 99), clearing the preregistered 0.88 floor, validation-only with zero test
calls; G-7 — the ~1.64M-parameter residual DJSCC model completed a full training-only
epoch at batch 32 within the RTX 4060 profile; G-2 — physical-layer conformance above;
W4 — a bounded 55-row integration smoke (~50 s) including 24 Imagenette-160 validation
images at 18 dB and 24 at −8 dB plus CIFAR-10 transport-only rows, explicitly labelled
*not* the BR-4 full sweep and not test evidence
(`results/baseline/w4/integration_adjudication.json`); G8_C — the Pascal successor
campaign accepted 3,213/3,213 identities at 5,000 trials each (16,065,000 trials) and
the measured-only successor BLER table is frozen at 153 curves / 3,213 points; G8_D —
validation-measurement tooling complete through D7.

### Q17. You cite an outage accuracy of 0.1 — is that assumed?

No, it is measured and count-derived: the constant-class outage policy counts labels
across the exactly stratified 1,000-image validation manifest, giving 100/1000 for
class 0, which equals 1/n_classes only because the split is perfectly balanced. The
artifact carries numerator, denominator and the full count vector precisely so the
assumption and the measurement can be told apart (`integration_adjudication.json`,
`outage_policy.json`).

### Q18. What did G8_C produce, in one sentence?

A complete physical-layer characterization: every one of 3,213 (modulation, SNR)
identities measured at 5,000 Monte-Carlo trials each on the qualified Pascal worker,
with protected counters (inference/training/validation-decoding/test access) all
zero, feeding the frozen successor BLER table used by BR-4 selection.

---

## Theme F — Scope, tiers, simulation-first

### Q19. Will you build real radio hardware?

Tier 1 — the full defensible capstone — is simulated AWGN and is the required scope;
the claim covers TS 38.212-derived LDPC/rate matching over abstract AWGN, not a
complete 5G NR link. Tier 2 (offline SDR replay over a conducted cable path) and
Tier 3 (Raspberry Pi-class live demo) are gated stretch goals behind purchase gate
G-5, with a pre-recorded demonstration as the expected outcome (DEC-14).

### Q20. Why simulate instead of testing on real hardware first?

AWGN isolates the scientific mechanism and supports exactly paired randomness between
arms; hardware adds synchronization, CFO, clipping and regulatory variables that would
confound the comparison. The deployment dossier keeps hardware work optional, budgeted
(HackRF One + RTL-SDR within the INR 25,000–40,000 envelope) and safety-bounded
(conducted loopback, no radiated operation).

---

## Theme G — Deployment and demo

### Q21. What does the final demonstration look like?

A live SNR slider driving both pipelines side-by-side on the same image, with the
accuracy-vs-SNR plot updating in real time; the Streamlit comparison can run locally,
CPU-capable, without SDR (`docs/deployment-dossier.md`).

### Q22. Does the SDR frame use your scientific symbol budget?

Not silently. The SDR wrapper adds preamble, repeated header, pilots and guard
symbols; its overhead must be reported separately and never presented as if it used
the scientific symbol budget. The payload path must deliver exactly `k` scientific
symbols or fail the frame closed.

---

## Not yet available — say so honestly

None of the following exists at this review basis. If asked, answer plainly and do
not present bounded smoke as a headline comparison:

- **Learned-vs-classical comparison** — "The final comparative experiment has not run;
  the foundations (G-1 classifier, G-2/LDPC characterization, G8_C BLER table, G8_D
  measurement tooling) exist, and the remaining gate-ordered path runs validation
  sweeps and pass-one/pass-two selection before any comparison."
- **Trained DJSCC model** — "Only the G-7 architecture/profile feasibility check has
  run within capacity bounds; end-to-end learned-system training and λ calibration are
  later planned stages."
- **Test-split results** — "The test split is sealed until gate G-12, after all
  choices freeze; test-access counters across all completed campaigns are zero."
- **Demo** — "The Streamlit side-by-side demo is planned for after Tier-1 results
  freeze; nothing live exists yet."

Also unavailable and equally honest to state: operating-ratio selection (G-8/G8_G),
ER-9 digital feature control implementation, thesis chapters, paper, poster,
plagiarism report and any hardware execution. Reviewers should hear these as
scheduled future work, not gaps hidden behind jargon.
