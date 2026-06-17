# Phase 4 — Improving the Loop: Feedback and Refinement

> In Phase 3, you achieved a working hybrid, but with a *static* collaboration: the AI generates seeds at the beginning, and AFL++ explores from there. This phase makes it *dynamic*: having the AI continue to contribute throughout the entire campaign, guided by the progress of the fuzzing. The guiding concept of this phase is that AFL++, even with good seeds, eventually plateaus again (exploring the neighborhood of the seeds and running out of new paths). That is where the AI can step back in, generating fresh inputs to unblock it. However, there is a realistic trade-off running through this phase that demonstrates engineering judgment: the LLM is *slow* compared to AFL++, so it must be invoked sparingly so that its contribution outweighs its cost. Making the collaboration more effective does not mean calling the LLM constantly, but rather calling it at just the right moment.

**Phase objective:** To make the AI-fuzzer collaboration more effective.  
**Duration:** Weeks 6-7.  
**Upon completion, you will have:** An improved hybrid fuzzer where the AI and AFL++ collaborate dynamically (with the AI providing inputs when needed), making it notably more effective than the initial version.

---

## The Overall Concept

The flow of this phase closes the loop between the fuzzer and the AI:

```
   AFL++ fuzzing (from the AI seeds)
        │
        ▼
   [ Monitor coverage ]  ──►  Has it stalled? (via fuzzer_stats)
        │  yes
        ▼
   [ AI regenerates inputs ]  ──►  fresh, and if possible, targeted at uncovered parts
        │
        ▼
   [ Inject into the corpus ]  ──►  new starting points to unblock AFL++
        │  (sparingly: the LLM is slow)
        └──────────────► (back to fuzzing)
```

---

## Step 1 — From Static to Dynamic

It is worth understanding why we take this step, which shows you understand how a fuzzing campaign evolves. In Phase 3, the AI provided value *once*, at the beginning, by giving AFL++ good seeds. This helps, but it has a ceiling: AFL++ explores the neighborhood of those seeds and, sooner or later, plateaus again, finding no new paths. Coverage, which grew at start-up, flattens out once more. At that point, the initial good seeds have yielded all they possibly can.

This is where a *dynamic* collaboration delivers value: if the AI remains available during the campaign, it can step back in right when AFL++ gets stuck, generating fresh and distinct inputs that provide new starting points to explore. The difference is like a collaborator who helps you get started and then leaves, versus one who stays and lends a hand whenever you get stuck. Transitioning the collaboration from static to dynamic is what makes the hybrid significantly more effective.

---

## Step 2 — Regenerating When Coverage Plateaus

The main refinement is a loop that monitors AFL++ coverage and, when it detects a plateau, requests fresh inputs from the AI and injects them into the corpus. AFL++ writes its statistics (including coverage) to the `fuzzer_stats` file in its output directory, which you saw in Phase 1, so you can read it to check progress. Create `src/fuzz/loop.py`:

```python
import time
from pathlib import Path

from src.generate.generate import generar_entradas
from src.generate.validate import filtrar_validas


def leer_cobertura(out_dir="out_hibrido") -> int:
    # AFL++ writes coverage (e.g., number of paths) to fuzzer_stats
    stats = dict(
        line.split(":", 1)
        for line in Path(out_dir, "default", "fuzzer_stats").read_text().splitlines()
        if ":" in line
    )
    return int(stats["paths_total"])   # a measure of the reached coverage


def bucle_dinamico(intervalo=600, umbral=5):
    previa = 0
    while True:
        time.sleep(intervalo)                       # wait a while between checks
        actual = leer_cobertura()
        if actual - previa < umbral:                # coverage has stalled
            nuevas = filtrar_validas(generar_entradas())   # AI regenerates (valid and diverse)
            for i, xml in enumerate(nuevas):
                Path(f"corpus/dyn_{int(time.time())}_{i}.xml").write_text(xml)  # inject
        previa = actual
```

The logic is simple yet powerful: periodically, it checks if coverage has increased; if it has barely grown (stalled), the AI generates new inputs (valid and diverse, reusing Phase 2 code) and adds them to the corpus, where AFL++ will pick them up as new starting points. In this way, the AI transitions from a one-time initial contributor to an active collaborator that reacts to the state of the fuzzing process.

---

## Step 3 — Refinements: Targeted Generation and Semantic Feedback

Building on this basic loop, there are two refinements that demonstrate ambition and deep understanding, which you can tackle depending on your available time.

The first and most powerful is **targeted generation**: instead of regenerating blindly, inform the AI about which parts of the format (or code) remain unexplored so it can generate inputs targeting those specific areas. This closes the feedback loop more tightly: coverage not only *triggers* regeneration but also *guides* what gets generated. If you know, for example, that a certain feature of the format has barely been exercised, you can ask the LLM for inputs that use it heavily. This is more challenging (since you have to map coverage to input features), but it is where the AI-fuzzer collaboration becomes truly intelligent.

The second, optional and more advanced, is **semantic feedback**: going beyond code coverage. Coverage is a good indicator of an "interesting input", but not perfect: some bugs (especially *logic* bugs, rather than memory bugs) do not manifest as newly executed code. Prioritizing inputs that trigger *novel behaviors or states* (not just new coverage) can help hunt down these bugs. This is a more ambitious research direction and a great stretch goal, but it is worth knowing about because it demonstrates that you understand the limitations of coverage as a metric.

---

## Step 4 — The Cost of Inference

Here is the realistic trade-off that demonstrates engineering judgment and ties back to the core of the project. The LLM is **slow**: generating an input can take seconds, whereas AFL++ performs *thousands* of executions per second. This has a consequence that cannot be ignored: if you call the LLM too frequently (in the extreme case, on every mutation, as one might be tempted to do with a custom mutator), you would slow down fuzzing so much that AFL++ would execute far fewer iterations, and the net effect could be *negative*—worse than AFL++ alone.

That is why the key is to invoke the LLM **sparingly**: in this loop's design, only when coverage stalls (not constantly), so that the cost of inference is distributed and remains dominated by the time AFL++ spends fuzzing at full speed. The rule of thumb that should guide you is that *the LLM's contribution must outweigh its cost*: every time you pause to generate, you are taking time away from AFL++, so it has to be worth it in terms of gained coverage. Keeping this in mind (and, later on, measuring it) is precisely the objective, realistic mindset that this project cultivates: not assuming that more AI is inherently better, but rather ensuring that it actually pays off.

---

## Step 5 — Iterate and Verify

With the dynamic loop set up, iterate on its parameters and verify that it improves: adjust how often you check coverage, the threshold that decides it is stalled, and the number of inputs to regenerate, searching for the combination that yields the most coverage and bugs without paying too high of an inference cost. The logic of the loop is testable even if actual fuzzing is not:

```python
def test_la_deteccion_de_estancamiento_dispara_la_regeneracion():
    # If coverage barely increases between checks, it is considered stalled
    assert se_ha_estancado(previa=100, actual=102, umbral=5) is True    # +2 < 5: stalled
    assert se_ha_estancado(previa=100, actual=130, umbral=5) is False   # +30: still growing
```

The test verifies the stall-detection logic, which triggers the AI's intervention. You will see the actual improvement (more coverage, more bugs) during the campaign, and you will measure it rigorously in Phase 5. The question guiding this iteration is the core criterion of this phase: is dynamic collaboration significantly more effective than the static version from Phase 3?

---

## Verification: The "Done" Criterion

The phase is complete when the following are met:

- [ ] You have transitioned the collaboration from static to dynamic: the AI provides inputs throughout the campaign, not just at the start.
- [ ] You regenerate inputs using the AI when coverage stalls (by monitoring AFL++ statistics).
- [ ] You have considered the refinements (targeted generation guided by uncovered areas, and semantic feedback).
- [ ] You manage the inference cost: you invoke the LLM sparingly so that its contribution outweighs its cost.
- [ ] You have iterated on the loop parameters to find the best balance of coverage and bugs.
- [ ] **The key test:** The AI-fuzzer collaboration is significantly more effective than the initial version.

The key test is actual improvement over the static version: if your dynamic hybrid (where the AI reacts to stagnation) achieves significantly more than Phase 3 (where the AI only contributed at the start), you have made the collaboration truly effective. And doing so while keeping an eye on inference cost, instead of calling the LLM excessively, is what demonstrates the engineering judgment that distinguishes a working solution from one that merely looks sophisticated.

---

## Deliverables and What Comes Next

By the end of Phase 4, you have a fine-tuned hybrid fuzzer: the AI and AFL++ collaborate dynamically, with the AI providing fresh inputs when coverage stalls, while you ensure that this contribution outweighs its cost. You have transitioned from a static collaboration (seeds at the beginning) to a dynamic one (the AI reacts to the fuzzing state), making the hybrid significantly more effective, and you have done so with the engineering discipline to prevent LLM costs from negating its advantages. You now have the system in its best shape, ready for the moment of truth.

The next step, **Phase 5**, is the heart of the project: **rigorous evaluation**. Up to this point, you have seen *signs* that your hybrid adds value, but you have not yet demonstrated it rigorously. You will compare the hybrid against AFL++ alone on a standard benchmark (FuzzBench/Magma) using multiple runs and confidence intervals (since fuzzing exhibits high variance), measuring both coverage and bugs, and honestly analyzing the results: where and how much the AI contributes, and where it does not. This is the phase that distinguishes a project that *seems* better from one that is *proven* to be so, and where all credibility is at stake. You have moved from "I have a fine-tuned hybrid showing good signs" to being on the verge of "I know, with rigor and honesty, whether my AI actually beats the baseline."
