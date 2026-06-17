# Phase 5 — Rigorous Evaluation (Beating AFL++)

> **This phase is the heart of the project.** Up until now, you have seen *signs* that your hybrid adds value, but you have not yet proven it. This is where the central question is decided (does your hybrid actually beat AFL++ alone?) and, above all, where the credibility of the entire work is at stake. The value of this project does not lie in *having* an AI-powered fuzzer, but in *honestly demonstrating* whether that AI is useful. This matters especially here, as the field suffers from a well-documented problem: a landmark study found that fuzzing evaluations often lacked rigor, making them irreproducible and leading to misleading conclusions. The re-evaluation you saw in the roadmap (where plain AFL++ outperformed ML-based fuzzers) is that very problem in action. That is why this phase applies the standard rigorous methodology of the field, to answer with honesty. Rigor is not a formality: it is the intellectual core of the project.

**Phase Objective:** To honestly demonstrate whether the AI adds value compared to the baseline.  
**Duration:** Weeks 7-8.  
**Upon completion, you will have:** A rigorous evaluation of your hybrid against AFL++ alone, showing if it beats it, by how much, and in which cases, backed by the statistical robustness that the field demands.

---

## The Big Picture

The workflow of this phase addresses the central question with rigor:

```
   Hybrid  vs  AFL++ alone
        │
        ├──► [ 1. Multiple runs ]  ──►  due to fuzzing variance (the standard: 30 runs, 24h)
        │
        ├──► [ 2. Metrics + Ground Truth ]  ──►  coverage over time, bugs (Magma)
        │
        ├──► [ 3. Statistical Significance ]  ──►  Mann-Whitney U test (real or random chance?)
        │
        └──► [ 4. Where it helps vs where it doesn't + Honest analysis ]
        │
        ▼
   [ Does it beat the baseline? Where? By how much? — with honesty ]
```

---

## Step 1 — Why Rigor Is Necessary: Fuzzing Variance

Before measuring anything, it is essential to understand why a single run proves nothing, as this is the foundation of the entire methodology. Fuzzing is a **stochastic** (random) process: it relies on random mutations, meaning two identical campaigns can yield different results purely by chance. One run might find a bug in 3 hours, while another might not find it in 24 hours using the exact same fuzzer. Therefore, comparing two fuzzers with only a single run each is misleading; the difference you see could just be noise rather than a real difference.

The standard methodology in the field (established by Klees et al. in their landmark study) solves this by using **multiple independent runs** for each fuzzer, allowing us to discuss their *typical* performance rather than a fluke. The canonical recommendation is **30 runs** of **24 hours** each. For a portfolio project, performing at least a few (5-10) is already a massive leap forward compared to a single run, though it is worth mentioning the standard to demonstrate that you are aware of it. Additionally, duration matters because performance can *change over time*: a fuzzer might be ahead at 6 hours but fall behind at 24, so you must measure across the entire campaign and for a sufficient duration. This lesson explains why the check in Phase 3 was only preliminary: without multiple runs, there is no solid conclusion.

---

## Step 2 — Metrics and Ground Truth

With the runs planned, you need to decide what to measure, and here lies an important distinction that demonstrates good judgment. There are two types of metrics. The first is **code coverage** over time (how much code each fuzzer executes). This is the metric used by **FuzzBench** (Google's benchmarking platform), which also comes with built-in multi-run experiments and statistical tests. Coverage is useful, but it is a *proxy metric*: more coverage does not necessarily equate to more bugs.

This is why the second, more decisive metric is **discovered bugs**, which brings in **ground truth**. Counting raw crashes is misleading (many crashes can be the same underlying bug, and coverage does not guarantee bug detection). The solution is **Magma**, a benchmark with *known* bugs injected into real-world programs, featuring a mechanism that detects exactly when each bug is triggered. With Magma, you can measure what truly matters: does your fuzzer find the known bugs? How many? How quickly? This is real bug detection, not an approximation. Therefore, measure coverage over time (using FuzzBench), known bugs found (using Magma), and time-to-first-bug, comparing the hybrid against AFL++ alone.

---

## Step 3 — Statistical Significance: The Mann-Whitney U Test

This is the rigor that separates a serious evaluation from a naive one, demonstrating scientific maturity. With multiple runs for each fuzzer, you have two sets of results (the hybrid's and AFL++'s), and the question is: is the difference between them *real*, or could it be due to chance? It is not enough to simply compare means or medians (which can be misleading); you must apply a **statistical test**. The standard in fuzzing (recommended by Klees et al. and used by FuzzBench and Magma) is the **Mann-Whitney U test**, a non-parametric test suitable because the results come from independent populations without a assumed distribution:

```python
from scipy.stats import mannwhitneyu

def is_significant_improvement(hybrid_results, afl_results):
    # Is the difference real or due to chance? (Mann-Whitney U test, the fuzzing standard)
    stat, p = mannwhitneyu(hybrid_results, afl_results, alternative="greater")
    return p, p < 0.05   # p < 0.05: the difference is statistically significant
```

It is also highly recommended to pair statistical significance with **effect size**. Just because a difference is significant (likely real) does not tell you *how much* better it is. For this, the standard is the **Vargha-Delaney Â₁₂** statistic, which quantifies how often one fuzzer outperforms the other. This combination (Mann-Whitney U for significance, Vargha-Delaney for effect size) is exactly what the field's rigorous methodology recommends, and using it is what makes your conclusion defensible. A claim like "my hybrid found more bugs" means nothing without this; "my hybrid found significantly more bugs (p < 0.05), with a large effect size" is a serious scientific claim.

---

## Step 4 — Where It Helps, Where It Doesn't, and Honest Analysis

This is the part that embodies the soul of the project, where maturity becomes visible. It has two sides.

The first is **where** to evaluate, which is a revealing design decision. Evaluate on a **structured** target (where the validity barrier exists, and where your AI *should* help), but, if possible, also evaluate on an **unstructured** target (where the AI is not expected to add value because there is no validity barrier to overcome). Showing that your AI helps with structured inputs *while* acknowledging that it does not help with unstructured ones is not a weakness: it is proof that you understand *where* ML adds value and where it does not. This is precisely the kind of judgment that distinguishes someone who knows how to use AI from someone who just applies it to everything. A nuanced result ("it helps here, but not there") is far more valuable and credible than an absolute one.

The second is the **honest analysis** of the results, which is where the project stakes its credibility. Analyze frankly: does the hybrid beat AFL++? On which targets? By how much (the effect size)? And where does it fall short? Given the field's reproducibility issues, report whatever you find, no matter what it is. If the result shows that the AI helps with structured inputs but not unstructured ones, that is an honest and valuable result. If the improvement is modest, say so. Even if it turns out that your hybrid does *not* conclusively beat a well-tuned AFL++, that is also a legitimate result. Given the strength of the baseline, it is nothing to be ashamed of—you would simply be replicating exactly what the field's rigorous re-evaluations have found. A modest or honest result presented with scientific rigor is worth far more (and is much more credible) than a hyped-up "my fuzzer beats them all" claim, which is precisely the kind of assertion that has given many ML-fuzzing evaluations a bad reputation. Your honesty here is your greatest asset.

---

## Verification: The Definition of "Done"

The phase is complete when the following conditions are met:

- [ ] You have performed multiple independent runs of each fuzzer (not just a single run), accounting for fuzzing variance.
- [ ] You have measured coverage over time and bugs found, using ground truth (Magma) and/or FuzzBench.
- [ ] You have verified statistical significance (Mann-Whitney U) and effect size (Vargha-Delaney).
- [ ] You have evaluated where the AI helps and where it does not (structured vs. unstructured targets, if possible).
- [ ] You have analyzed the results honestly, without exaggeration, reporting where it does not add value.
- [ ] **The key test:** You know whether your hybrid fuzzer beats AFL++, to what extent, and in which cases.

The key test is a rigorous and honest verdict: if you know—through multiple runs, statistical tests, and a fair baseline—whether your hybrid beats AFL++, by how much, and where, you have conducted an evaluation worthy of the field's standards. Most importantly, and what defines this project: that verdict is *honest*, whether it is favorable, nuanced, or negative. Resisting the temptation to exaggerate and letting the evidence (rather than expectations) decide is exactly the scientific maturity that this project demonstrates, and what will distinguish you the most.

---

## Deliverables and Next Steps

Upon wrapping up Phase 5, you will have completed the heart of the project: a rigorous and honest evaluation of your hybrid against AFL++. You will know, backed by multiple runs, ground-truth metrics, statistical tests, and a fair baseline, whether your AI actually adds value, how much, and in which scenarios. You have applied the methodology that the field considers correct (one that many evaluations omit) and reported the results with complete honesty, whatever they may be. You now have a defensible conclusion, which is far more valuable than an impressive but fragile claim.

The next step, **Phase 6**, turns that evaluation into insight and something presentable: **Analysis and Visualization**. You will dive deeper into *why* the AI helps where it does (and why it doesn't where it doesn't), and you will visualize the results (such as compared coverage curves and bugs found over time) clearly and convincingly. This is the transition from "I have the evaluation results" to "I understand what they mean and can present them at a glance," which is what will make your work understandable and credible to anyone looking at it. You have gone from "I know with rigor if my AI adds value" to being on the verge of "I understand why, and I can present it convincingly."
