# Phase 3 — The Hybrid Fuzzer: Integrating AI with AFL++

> In this phase, you bring the two pieces together and realize the central idea of the project: the **collaboration** between the AI and the fuzzer. In Phase 2, you built a generator that produces valid and diverse inputs; now you connect it to AFL++, so that the AI provides those inputs and AFL++ mutates and explores them. It is the concrete realization of the foundational concept: the AI provides the *structural knowledge* (valid inputs that bypass the barrier), and AFL++ provides the *guided exploration* (mutating and discovering paths). The guiding principle of this phase is that this is not about replacing AFL++, but rather giving it a much better starting point, and then verifying that, with this starting point, it reaches parts of the program it could not reach on its own.

**Phase objective:** Unite AI-based generation with the fuzzing engine.  
**Duration:** Weeks 5-6.  
**By the end, you will have:** A working end-to-end hybrid fuzzer (the AI provides inputs, AFL++ mutates and explores them) that penetrates deeper into the program than AFL++ alone.

---

## Overall Concept

The workflow for this phase connects the two pieces:

```
   [ AI Generator (Phase 2) ]  ──►  valid and diverse inputs (seeds)
        │
        ▼
   [ AFL++ starts from those seeds ]  ──►  mutates them and explores outward
        │
        ▼
   [ Hybrid fuzzer ]  ──►  reaches deep logic that AFL++ alone could not reach
        │
        ▼
   [ Check: more coverage than baseline? ]  (preliminary; rigor comes in Phase 5)
```

---

## Step 1 — The Concept: AI Provides Seeds, AFL++ Explores

Before integrating anything, it is worth understanding why the most natural and robust way to unite the two pieces is through the **seed corpus**, and why this matters so much in fuzzing. AFL++ explores *outward* from the seeds you provide: it takes an input from the corpus, mutates it, and if the mutation discovers new code, it saves it to continue mutating it. This means that the starting point (the seeds) heavily dictates how far it goes: if it starts with poor seeds (a minimal XML, or garbage), it gets stuck at the validity barrier, as you saw in Phase 1; but if it starts with *valid and diverse seeds that already reach the deep logic*, it explores the neighborhood of that deep logic, which is exactly where the interesting bugs lie.

That is why the AI's contribution is, precisely, *better seeds*. You do not modify the AFL++ engine (which is excellent); you give it a much better starting point. This connects to something well-known in fuzzing: seed selection is one of the factors that most heavily determines the success of a campaign. Your AI targets exactly this factor, providing AFL++ with seeds that bypass the validity barrier and exercise diverse code paths, so that its exploration begins where it previously could not even reach.

---

## Step 2 — Integrating AI Inputs into AFL++

The integration, in its primary form, is straightforward and robust: using the AI-generated inputs (from Phase 2) as the **initial corpus** for AFL++. Recall that in Phase 2 you saved them in `corpus/`; now you provide them to AFL++ as a starting point:

```bash
# The AI-generated seed corpus as input for AFL++.
# AFL++ starts from these valid and diverse seeds, and explores by mutating them.
afl-fuzz -i corpus/ -o out_hybrid -m none -- ./xmllint_cov @@
```

This is the hybrid fuzzer in its cleanest form: the AI generated the corpus, and AFL++ does its exploration work starting from there. It is robust (it does not modify the AFL++ engine, so you do not introduce instability) and directly addresses the problem (the validity barrier is bypassed because the seeds are already valid).

It is worth knowing the **more advanced alternative**, which demonstrates that you understand the available options: the AFL++ *Custom Mutator API*. Instead of (or in addition to) providing seeds at the beginning, you can write a custom mutator (AFL++ supports Python mutators) that calls the LLM *during* fuzzing, generating or mutating inputs on the fly. This is a more dynamic and powerful integration, but also more complex, and it comes with a challenge that must be managed (the cost of LLM inference, which can significantly slow down fuzzing). Therefore, the sensible approach is to start with the seed-based integration (which is robust and sufficient to prove the concept) and leave dynamic refinement for the next phase. The important thing right now is to have the hybrid fuzzer working end-to-end.

---

## Step 3 — Verifying That the Hybrid Penetrates Deeper

This is the core purpose of this phase: verifying that the hybrid indeed reaches parts of the program that AFL++ alone could not. This is the first indication of whether your AI adds value. The verification consists of comparing the coverage of the hybrid (AFL++ starting from the AI seeds) with the Phase 1 baseline (AFL++ starting from poor or minimal seeds), and seeing if the hybrid bypasses the validity barrier where the baseline stalled:

```python
# Measure the hybrid's coverage and compare it to the Phase 1 baseline.
# Does the hybrid reach the parser's deep logic where AFL++ alone got stuck?
# (recompile with coverage instrumentation and generate a report on discovered paths)
```

If the concept works, you expect to see the hybrid reaching more code (especially in the deep processing logic, not just superficial validation) because it started from seeds that had already passed through the gate. Seeing this difference is a preliminary confirmation that providing better seeds to AFL++ helps.

And here is a point of honesty that demonstrates maturity, linking to the core of the project: this is a **preliminary** check, not a rigorous evaluation. A single run proves nothing conclusively because fuzzing involves high variance (two identical campaigns can yield different results purely by chance). The serious evaluation, with multiple runs and statistical rigor against a fair baseline, is Phase 5. For now, it is enough to see indications that the hybrid reaches deeper; do not get carried away by a favorable initial comparison, as rigor will come later. This caution is precisely the mindset this project aims to cultivate.

---

## Step 4 — Sanity Check

Since this phase is primarily about integration and orchestration, verification consists of checking that the hybrid runs end-to-end. Confirm that: the AI seed corpus is ready and valid (all seeds pass parsing, from Phase 2); AFL++ starts from that corpus and fuzzes (you can see executions and paths on the status screen); and you have an initial coverage measurement of the hybrid to compare against the baseline. Here is a simple check to ensure the seeds are correct before launching the fuzzer:

```python
from pathlib import Path
from src.generate.validate import is_valid

def check_corpus(folder="corpus"):
    seeds = list(Path(folder).glob("*.xml"))
    assert len(seeds) > 0, "The seed corpus is empty"
    assert all(is_valid(s.read_text()) for s in seeds), "There are invalid seeds in the corpus"
    print(f"Corpus OK: {len(seeds)} valid seeds, ready for AFL++")
```

This check confirms that the seeds feeding the hybrid are valid (the foundation for the integration to make sense) before launching the campaign.

---

## Verification: The Definition of "Done"

The phase is complete when the following are met:

- [ ] You have integrated the AI-generated inputs into AFL++ (as a seed corpus).
- [ ] The hybrid fuzzer works end-to-end: the AI provides valid and diverse inputs, and AFL++ mutates and explores them.
- [ ] You understand the advanced alternative (a custom mutator that calls the LLM during fuzzing) and why starting with seeds is preferred.
- [ ] You have verified (preliminarily) that the hybrid reaches code that AFL++ alone could not reach.
- [ ] You understand clearly that this check is preliminary, and that rigor arrives in Phase 5.
- [ ] **The key test:** You have a fuzzer that combines AI generation with the coverage engine, and it explores deeper than AFL++ alone.

The key test is having the hybrid running and showing signs of contribution: if you have the AI and AFL++ working together (the AI providing valid and diverse seeds, AFL++ exploring), and you see that the hybrid reaches deeper than AFL++ alone, you have realized the collaboration that lies at the core of the project. The key word here is *indications*: at this stage, you are looking for promising signs, not a conclusive proof, which is what you will rigorously build in Phase 5.

---

## Deliverables and What's Next

By completing Phase 3, you have a functioning hybrid fuzzer: the AI provides valid and diverse inputs, AFL++ mutates and explores them, and, in an initial check, the hybrid reaches parts of the program that AFL++ alone could not reach. You have materialized the collaboration between structural knowledge (the AI) and guided exploration (the fuzzer), tackling the validity barrier through the most robust route (better seeds). And you have done so with the caution of knowing that the rigorous testing comes later.

The next step, **Phase 4**, makes this collaboration more effective: **improving the feedback loop**. You will refine how and when the AI provides inputs (for example, regenerating when coverage plateaus, rather than only at the beginning), optionally explore semantic feedback (prioritizing inputs that trigger novel behavior), and iterate to improve coverage and discover more bugs. This is the transition from a working hybrid to a fine-tuned one, where the AI and the fuzzer collaborate more dynamically. You have moved from "I have a working hybrid that shows promising signs" to being on the verge of "I have a fine-tuned hybrid that makes better use of the collaboration."
