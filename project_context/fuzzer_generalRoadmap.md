# Roadmap: Smart Fuzzer with ML

> A fuzzer that combines a coverage-guided fuzzing engine (AFL++) with an AI component that generates structurally valid and diverse inputs, to overcome the barrier that slows down traditional fuzzing (inputs rejected right at the start) and reach the deep logic where bugs hide. It combines vulnerability discovery (a cornerstone of security) with machine learning on a cutting-edge problem, producing one of the most tangible outcomes possible: real bugs found in real software.

**Estimated duration:** 8-10 weeks (part-time)
**Level:** Intermediate-High / Advanced
**Final outcome:** A hybrid fuzzer that augments AFL++ with AI-driven input generation, capable of achieving more coverage and finding more bugs than AFL++ alone on a structured-input target, with the findings triaged, understood, and responsibly disclosed if applicable.

> **The project's defining approach, from the start:** traditional fuzzing's random mutation is *blind*. When the target expects structured input (a file format, a protocol), most random inputs are rejected right at the doorstep (during parsing), so the fuzzer wastes almost all its effort bouncing off validation without ever reaching the deep logic where the interesting bugs lie. This is the core problem: the **validity barrier**. And this is where AI adds real value: learning the *structure* of valid inputs to generate inputs that pass the gate and let the fuzzer explore the actual logic. However, it is important to be honest, which is part of the project's maturity: **AI does not replace coverage-guided fuzzing (AFL++ is a very strong foundation); it augments it**, feeding it valid and diverse inputs so it can explore deeper. The bottleneck is not generating inputs, but generating *valid* inputs that reach far, and that is where learning the structure helps.

> **Note on responsible disclosure:** fuzzing finds **real vulnerabilities**. This project is built on a responsible framework: fuzzing practice targets or your own code (not third-party production software without permission), understanding that the goal of fuzzing is *constructive* (finding bugs to fix them and make software more secure), and, if a real vulnerability is found in third-party software, disclosing it **responsibly** (to the maintainer, giving them time to patch it) instead of publishing or exploiting it. Recognizing the dual-use nature of this work (a found bug could be exploited) and positioning yourself on the constructive side is a sign of professionalism.

---

## 1. What exactly you are going to build

A hybrid fuzzer, following this diagram:

```
   Target: a program that parses structured input (a parser)
        │
        ▼
   ┌────────────────────────────────────────────────────────┐
   │   Base fuzzer (AFL++): mutation + coverage feedback.   │
   │   BUT it gets stuck at the VALIDITY BARRIER            │
   │   (inputs rejected during parsing)                     │
   └────────────────────────────────────────────────────────┘
        │
        ▼
   [ AI Input Generation (LLM) ]  ──►  VALID and diverse inputs
        │  (learns/knows the format structure)
        ▼
   [ Hybrid fuzzer ]  ──►  AFL++ + smart inputs → explores deep logic
        │
        ▼
   [ Bugs / crashes found ]  ──►  triaged, understood, responsibly disclosed
```

The core idea, and what drives the value of the project, is what you already saw in the approach note: **overcoming the validity barrier with AI-generated inputs, not by replacing the existing coverage engine, but by boosting it**. The base fuzzer (AFL++) is excellent at exploring, but gets stuck when nearly all of its inputs are rejected before reaching the logic. The AI provides valid and diverse inputs as a starting point and fuel, allowing the coverage engine to do its real job, reaching parts of the program that were previously unreachable. You are building a collaboration between two techniques: AI provides the knowledge of the structure, and fuzzing provides the guided exploration.

---

## 2. The use case and why it is impressive

The project addresses a real and cutting-edge problem, combining several arguments that make it especially powerful.

**Produces tangible results: real bugs.** Unlike many AI projects whose results are abstract, this one produces something concrete and impressive: real bugs and vulnerabilities found in real software. "My fuzzer found a bug in library X" is one of the most convincing statements you can make, as it demonstrates measurable and verifiable impact.

**Works at the frontier of research.** AI-guided fuzzing (using LLMs and neural networks) is a very active area of research in 2026, with many recent systems combining LLM generation and engines like AFL++. Working in this area shows that you are at the cutting edge of vulnerability discovery.

**Demonstrates maturity in ML usage.** As we discussed, it would be easy to overhype and present AI as a replacement for traditional fuzzing. Recognizing that AFL++ is a very strong foundation, that AI *augments* it (especially for structured inputs where the validity barrier lies), and demonstrating this with an honest evaluation against that baseline is what distinguishes someone who truly understands where ML adds value from someone who just applies trendy AI to everything.

**Combines security with your AI skills.** The project repurposes your generative and LLM skills (from your RAG project) for input generation, applied to a security problem. It also demonstrates responsible vulnerability handling, which is a sign of professionalism.

Additionally, this project naturally connects with the previous ones: it shares the **cybersecurity** theme of your honeypot, RL defense, and forensics projects (specifically, vulnerability discovery here), and reuses the **generative and LLM capabilities** of your RAG. It is the piece that demonstrates AI applied to vulnerability discovery with concrete results.

---

## 3. Tech Stack (updated to 2026)

| Layer | Tool | Why this choice |
|------|-------------|--------------|
| Language | Python 3.11+ with **uv** (orchestration) | Standard; reuses your foundation |
| Fuzzing engine | **AFL++** | The reference coverage-guided fuzzer (the baseline to beat) |
| Target | A structured-input parser | Where the validity barrier (and AI) matter |
| Input generation | **Local LLM** (Ollama) or generative model | Generates valid and diverse inputs (reused from your RAG) |
| Sanitizers | **AddressSanitizer (ASan)** | Detects memory corruption that does not cause a visible crash |
| Crash triage | AFL++ tools, debugger | Deduplicates and helps understand the bugs |
| Quality | pytest, ruff, Docker | Reuses your best practices |

Some notes on these choices, as the reasoning demonstrates solid criteria:

**AFL++ as the engine and baseline.** AFL++ is the reference coverage-guided fuzzer: it uses a genetic algorithm, mutations, and coverage feedback to evolve inputs that exercise more code. It is both the *engine* you will enhance and the *baseline* you must beat to prove your AI adds value. The fact that all recent research is built "on top of AFL++" confirms it is the correct foundation.

**A target with structured input.** The choice of target is key, showing you understand exactly where AI adds value. You choose a program that parses *structured* input (a file format, a protocol, a language), because that is precisely where the validity barrier lies (random inputs get rejected) and, therefore, where the AI makes a difference. For unstructured inputs (purely computational logic), AI is of little help, and it is good to know this. You can use a well-known target for practice (a common parsing library) or write your own.

**Local LLM for input generation (reusing your RAG).** The AI component directly leverages your skills: a local LLM (via Ollama) that generates valid and diverse inputs for the target's format, taking advantage of the fact that LLMs "know" or quickly learn common structured formats. This is not a forced idea; it is exactly the approach that cutting-edge 2026 research applies (systems like ChatFuzz or those that adapt models like Llama to AFL for formats such as XML, achieving multiple times more coverage). Alternatively, you can use a trained generative model (an LSTM or a transformer) that learns the structure from an input corpus.

**AddressSanitizer.** A detail that shows you understand fuzzing: many memory bugs (out-of-bounds reads/writes) do not cause a visible crash, so they would otherwise go unnoticed. Compiling the target with AddressSanitizer forces these errors to manifest as crashes, multiplying the number of bugs the fuzzer can find.

---

## 4. Repository Architecture

```
ml-fuzzer/
├── src/
│   ├── target/             # target to fuzz (a parser) + the harness
│   ├── generate/           # ML/LLM-based input generation
│   ├── fuzz/               # integration with AFL++ (the hybrid fuzzer)
│   ├── triage/             # triage of found crashes
│   ├── evaluation/         # evaluation (coverage, bugs vs. AFL++)
│   └── config.py
├── corpus/                 # seed corpus (gitignored if large)
├── crashes/                # found crashes (gitignored)
├── docs/
│   ├── context.md          # fuzzing, coverage-guided, the approach
│   └── responsible_disclosure.md   # responsible disclosure, dual-use
├── notebooks/
├── tests/
├── pyproject.toml
├── Makefile
└── README.md
```

The structure follows the project flow: `target` (the target and its harness), `generate` (the AI that generates inputs), `fuzz` (integration with AFL++), `triage` (understanding crashes), and `evaluation`. Two things stand out. The `corpus/` and `crashes/` folders are in `.gitignore` (inputs and crashes can be numerous and heavy, and crashes could contain details of vulnerabilities that should not be published outright). Additionally, the `docs/responsible_disclosure.md` is dedicated to responsible disclosure and dual-use: documenting a responsible framework shows you understand the sensitive nature of vulnerability discovery.

---

## 5. Phase-by-Phase Roadmap

Each phase has an objective, tasks, deliverable, and "definition of done".

---

### 🔹 Phase 0 — Setup, Foundations, and Approach (Week 1, early days)

**Objective:** environment ready, understanding fuzzing and ML for fuzzing, and setting an honest and responsible approach.

**Tasks:**
- [ ] Create the repository with the defined structure, environment setup with `uv`, code quality tools, and Makefile (reusing your past projects).
- [ ] Understand fuzzing, coverage-guided fuzzing (AFL++), and how ML can help.
- [ ] Document the approach (AI augments AFL++ to overcome the validity barrier, it does not replace it) and the reasoning behind it.
- [ ] Document the responsible disclosure framework and dual-use nature.

**Deliverable:** ready environment + understanding of the problem + documented approach and responsibility framework.
**Definition of done:** you understand fuzzing and why AI adds value primarily at the validity barrier, and you have a clear understanding of the responsible framework.

---

### 🔹 Phase 1 — The Base Fuzzer (AFL++) and the Validity Barrier (Weeks 1-3)

**Objective:** establish the baseline and witness first-hand the limitation that motivates the project.

**Tasks:**
- [ ] Install AFL++ and choose a target with structured input (a parser), compiling it with instrumentation and AddressSanitizer.
- [ ] Fuzz the target with AFL++ alone, measuring coverage and found bugs (the baseline).
- [ ] Observe the validity barrier: how AFL++ gets stuck, achieving low coverage of the deep logic because its inputs are rejected during parsing.

**Deliverable:** AFL++ baseline and a clear understanding of its limitations.
**Definition of done:** you have a measured baseline and have observed that AFL++ fails to reach the deep logic due to the validity barrier.

---

### 🔹 Phase 2 — ML/LLM-Based Input Generation (Weeks 3-5)

**Objective:** generate valid and diverse inputs. **This is the most distinctive AI component of the project.**

**Tasks:**
- [ ] Build an input generator with a local LLM (Ollama, reusing your RAG) that produces valid inputs for the target's format.
- [ ] Ensure that the inputs are both **valid** (pass parsing) and **diverse** (explore different edge cases).
- [ ] Validate and, if necessary, repair generated inputs (filter out invalid ones).

**Deliverable:** an AI-powered generator of valid and diverse inputs.
**Definition of done:** the AI successfully generates varied inputs that bypass the validity barrier.

---

### 🔹 Phase 3 — The Hybrid Fuzzer: Integrating AI with AFL++ (Weeks 5-6)

**Objective:** merge the AI-driven generation with the fuzzing engine.

**Tasks:**
- [ ] Integrate the AI-generated inputs into the AFL++ loop (as seeds and/or by feeding the corpus).
- [ ] Make the hybrid fuzzer work end-to-end: AI provides valid inputs, AFL++ mutates and explores them.
- [ ] Verify that the hybrid fuzzer reaches parts of the program that AFL++ alone could not access.

**Deliverable:** a working hybrid fuzzer.
**Definition of done:** you have a running fuzzer that combines AI generation with the coverage engine, exploring deeper than AFL++ alone.

---

### 🔹 Phase 4 — Improving the Loop: Feedback and Refinement (Weeks 6-7)

**Objective:** make the AI-fuzzer collaboration more efficient and effective.

**Tasks:**
- [ ] Refine how and when the AI feeds inputs (e.g., regenerating when coverage plateaus).
- [ ] Optionally, explore semantic feedback (prioritizing inputs that trigger novel behavior, not just new code coverage).
- [ ] Iterate to improve coverage and find more bugs.

**Deliverable:** an enhanced hybrid fuzzer.
**Definition of done:** the AI-fuzzer collaboration is noticeably more effective than the initial version.

---

### 🔹 Phase 5 — Rigorous Evaluation (Beating AFL++) (Weeks 7-8)

**Objective:** honestly demonstrate whether the AI adds value by comparing it against the baseline.

**Tasks:**
- [ ] Compare the hybrid fuzzer against AFL++ alone: coverage over time, unique bugs found, time to first bug.
- [ ] Evaluate on the structured target (and, optionally, on an unstructured one to see where the AI does not help).
- [ ] Analyze the results honestly: where and by how much the AI contributes, and where it falls short.

**Deliverable:** a rigorous evaluation against the baseline.
**Definition of done:** you know if your hybrid fuzzer beats AFL++, to what extent, and under what conditions.

---

### 🔹 Phase 6 — Bug Triage and Understanding (Weeks 8-9)

**Objective:** understand the found bugs and handle them responsibly.

**Tasks:**
- [ ] Triage found crashes: deduplicate, classify them, and find the root cause.
- [ ] Understand the vulnerabilities (type of bug, why it occurs, what the potential impact is).
- [ ] If a real bug is found in third-party software, prepare a responsible disclosure report.

**Deliverable:** triaged and understood bugs, along with a responsible disclosure plan if applicable.
**Definition of done:** you thoroughly understand the bugs your fuzzer found and manage them responsibly.

---

### 🔹 Phase 7 — Packaging and Presentation (Weeks 9-10)

**Objective:** highlight and present the value of the project.

**Tasks:**
- [ ] Containerize the system (reusing your established patterns).
- [ ] Write the final README: the problem, the approach, design decisions, results, and responsible disclosure.
- [ ] Record a video demonstrating the fuzzer finding bugs and outperforming AFL++.
- [ ] Write a post about the project and what you learned regarding AI-driven fuzzing.

**Deliverable:** containerized system + excellent README + video + post.
**Definition of done:** an outside observer can understand the project in two minutes, grasp its sophistication (fuzzing + AI), and see the bugs it uncovered.

---

## 6. The Perfect README (Brief Checklist)

The README should include: an engaging title and description; the problem and why it matters; an architecture diagram; a GIF or video of the fuzzer finding bugs; the tech stack with badges; instructions; a **design decisions** section (why AI augments AFL++ rather than replacing it, why a structured target, why an LLM); a **responsible disclosure** section (vulnerability handling, dual-use aspect); and **honest results** (how much it outperforms AFL++, in which cases, and the bugs found). (This section can be expanded in detail later.)

---

## 7. Common Pitfalls to Avoid

| Error / Pitfall | Why it is bad | What to do instead |
|---|---|---|
| Presenting AI as a replacement for fuzzing | AFL++ is a very strong foundation; overhyping hurts credibility | AI *augments* AFL++; prove it against the baseline |
| Choosing an unstructured target | Without a validity barrier, AI adds little value | Choose a structured-input target |
| Failing to compare with AFL++ alone | Without a baseline, you cannot know if the AI adds value | Always evaluate against AFL++ |
| Not using sanitizers | Many memory bugs do not cause visible crashes | Compile with AddressSanitizer |
| Generating valid but non-diverse inputs | Valid but identical inputs will not explore the code | Aim for both validity *and* diversity |
| Publishing or exploiting vulnerabilities | It is irresponsible and dangerous | Practice responsible disclosure; maintain a constructive focus |

---

## 8. Stretch Goals (Going Further)

- **Semantic feedback:** go beyond code coverage by prioritizing inputs that trigger novel behaviors or states, aiming to catch logic bugs in addition to memory bugs.
- **Stateful protocol fuzzing:** extend to stateful targets (like network protocols), where understanding the sequence of interactions is required (similar to AFLNet).
- **Directed fuzzing:** use AI to guide fuzzing towards specific parts of the program (such as new or suspicious code).
- **Fine-tuning the generative model:** train or fine-tune your own model for the target format and compare its performance with the generic LLM.
- **AI approach comparison:** compare LLM generation, trained generative models (LSTM/GAN), and gradient-guided fuzzing (similar to NEUZZ).
- **Automatic harness generation:** use an LLM to generate the fuzzing harness for a library, automating a tedious step.

---

## 9. Learning Resources

- **AFL++ Documentation:** for the fuzzing engine, its configuration options, instrumentation, and triage.
- **Fuzzing Resources:** guides on coverage-guided fuzzing, writing harnesses, and crash triage (excellent material exists from projects like OSS-Fuzz and the wider security community).
- **Literature on AI-driven Fuzzing:** cutting-edge papers from 2026 (including surveys on neural network fuzzing and LLM-based generation systems like ChatFuzz, LLAMAFUZZ, Fuzz4All, NEUZZ) establish the state of the art and, crucially, show how they are honestly compared against AFL++.
- **AddressSanitizer and Memory Error Detection Tools:** to help make silent memory bugs visible.
- **Responsible Disclosure Guides:** to learn how to properly and responsibly report vulnerabilities.
- **Core Concepts to Master:** fuzzing and coverage-guided fuzzing; the validity barrier and why structured inputs are challenging; AI input generation; crash triage and sanitizers; and why AI augments (rather than replaces) traditional fuzzing.

---

## 10. Milestone Summary

| Week | Milestone | Status |
|------|-----------|--------|
| 1 | Environment + foundations + approach | ⬜ |
| 1-3 | Base fuzzer (AFL++) and the validity barrier | ⬜ |
| 3-5 | ML/LLM-based input generation | ⬜ |
| 5-6 | Hybrid fuzzer (integrating AI + AFL++) | ⬜ |
| 6-7 | Improving the loop (feedback) | ⬜ |
| 7-8 | Rigorous evaluation (beating AFL++) | ⬜ |
| 8-9 | Bug triage and understanding | ⬜ |
| 9-10 | Packaging, README, video, post | ⬜ |

---

**The ultimate goal:** so that when someone looks at this project, they think, *"this person applies AI to vulnerability discovery, understands where it truly adds value, and finds real bugs."* It is not a fuzzer that claims to replace AFL++ with hype: it is a hybrid fuzzer that understands the validity barrier, overcomes it with generative AI, demonstrates this with an honest evaluation against the baseline, and produces tangible results (real bugs) handled with responsibility. It is the type of project that showcases not only technical capability but also strong judgment regarding where and how to apply ML.
