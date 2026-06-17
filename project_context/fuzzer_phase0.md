# Phase 0 — Setup, Foundations, and Approach

> This phase lays the foundation, and in this project, it has a dual distinctiveness that must be kept completely clear: **the honest approach and the responsibility framework take precedence over any code**. There is an easy temptation (presenting AI as a replacement for traditional fuzzing, with hype), and the most important decision of the project is to avoid falling into it: committing, right from the start, to an AI that *augments* AFL++ to overcome the validity barrier, rather than replacing it. There is also a dimension in vulnerability discovery that is not optional: responsible disclosure and awareness of dual use. If you do not establish these two elements now (the approach and responsibility), it is easy to drift into a project that exaggerates and loses credibility, or that mishandles something as sensitive as a vulnerability. That is why this phase is not just about "setting up the environment," but about "getting the approach and responsibility right."

**Phase Objective:** Environment ready, understand fuzzing and ML for fuzzing, and establish the honest and responsible approach.  
**Duration:** First few days of Week 1.  
**Upon completion, you will have:** A well-structured repository, a solid understanding of coverage-guided fuzzing and where ML helps, and, most importantly, the project's approach defined (AI augments AFL++, it does not replace it) and the responsible disclosure framework documented.

---

## The Big Picture

This phase is about getting the foundations right across four fronts—and note that nothing is being fuzzed yet (that begins in Phase 1): this is where we prepare and make key decisions.

```
   [ 1. Repository and environment ]     ──►  the engineering foundation
   [ 2. Understand the problem ]         ──►  fuzzing, coverage-guided, the validity barrier
   [ 3. Define the approach ]            ──►  AI AUGMENTS AFL++, does not replace it; and why
   [ 4. Establish responsibility ]       ──►  dual use, constructive focus, responsible disclosure
```

---

## Step 1 — The Repository and the Environment

You begin by setting up the engineering foundation that you already master and reuse: the repository with the foundational structure, the environment with `uv`, code quality with Ruff, and a Makefile.

```bash
uv init ml-fuzzer
cd ml-fuzzer
uv add --dev pytest ruff
mkdir -p src/{target,generate,fuzz,triage,evaluation} corpus crashes docs notebooks tests
```

There is a detail regarding responsibility that is best established now in the `.gitignore`: the `corpus/` and `crashes/` folders should **not** be tracked in the repository. Inputs and crashes can be numerous and heavy, but more importantly, crashes could contain details of vulnerabilities that should not be disclosed without due process. Setting this up from the very first commit demonstrates that you understand the sensitive nature of what you will be handling.

As for project-specific dependencies: you will install the AFL++ fuzzer in Phase 1 (when setting it up on a target), and the AI dependency (Ollama for the LLM) in the phase where input generation is implemented. In this phase, the lightweight Python base is sufficient, avoiding installing everything at once before it is actually needed.

---

## Step 2 — Understanding the Problem: Fuzzing and Where ML Helps

Before writing any code, dedicate time to thoroughly understanding the domain. Here, there is a difference compared to some of your other projects: although you bring a cybersecurity mindset from honeypots, RL-based defense, and forensics, *fuzzing* and coverage-guided fuzzing are largely new territory, so truly understanding them is a top priority. Document your findings in `docs/context.md`, which will also serve as material for the README.

Make sure you are clear on the concepts covered in the fundamentals. What **fuzzing** is (feeding a program tons of inputs to find flaws, with the benefit of producing a tangible result). What **coverage-guided fuzzing** is (AFL++: instrumenting the program, using coverage as a compass, and mutating inputs that reach new code, akin to a genetic algorithm). And, above all, the **validity barrier**: the fact that AFL++'s random mutations get stuck when the target expects structured input, because almost all of them are rejected during parsing, preventing the fuzzer from reaching deeper logic. Also, internalize where ML helps: generating valid and diverse inputs that bypass that barrier.

Developing a solid understanding of this domain now is what will allow you, in subsequent phases, to select the target wisely, design the generator with clear criteria, and evaluate the results honestly. It is the investment that makes everything else possible.

---

## Step 3 — Establishing the Honest Approach

This is one of the two most important steps of this phase, and the intellectual core of the project. It is where you commit, in writing, to the correct approach to avoid the temptation of overhyping. It is important to make this reasoning very clear, as being able to explain it demonstrates maturity.

The approach is: **AI augments AFL++ to overcome the validity barrier, it does not replace it.** The reasoning, as seen in the fundamentals, is as follows: It would be tempting to present AI as the novel technology that replaces traditional fuzzing, but AFL++ is an extraordinarily strong baseline. Its coverage-guided exploration is excellent, and beating it is far from trivial. AI does not arrive to replace that; it comes to solve a specific weakness—the validity barrier—by providing valid and diverse inputs so that the fuzzer's engine can explore deep logic. The bottleneck is not just generating inputs, but generating *valid inputs that reach deep code*, and that is where learning the underlying structure helps.

Document this along with its reasoning, and add a corollary that demonstrates you understand where ML adds value: AI helps primarily with **structured** inputs (where the validity barrier exists), rather than unstructured inputs. Consequently, the target you choose in Phase 1 must accept structured input. Establishing this approach is not a minor detail: it is the decision that will guide all subsequent ones (especially the honest evaluation against the baseline in Phase 5), and when properly explained, it distinguishes someone who understands where ML fits from someone who simply applies trendy AI to everything.

---

## Step 4 — Establishing the Responsible Disclosure Framework

The other decisive step of this phase is documenting the responsibility framework, which is not optional in vulnerability discovery. Compile this in `docs/responsible_disclosure.md`. There are several key points to clarify.

First, an awareness of **dual use**: fuzzing finds real vulnerabilities, and any found bug could be used constructively (to patch it) or harmfully (to exploit it). Explicitly recognizing this is the starting point.

Second, a **constructive focus**: the project's goal is to find bugs so they can be fixed and make software more secure, not to exploit them. This intent must be explicitly stated.

Third, target selection: fuzz only **practice or self-owned targets**, not third-party production software without permission. There are plenty of practice targets and open-source libraries specifically designed for this purpose.

Fourth, **responsible disclosure**: if you ever discover a real-world vulnerability in third-party software, the correct path is to report it privately and in a coordinated manner to the maintainer, giving them time to fix it, rather than publishing or exploiting it. Documenting this framework, showing that you understand the responsibility it entails, is a strong sign of professionalism and distinguishes those who approach security seriously.

---

## Step 5 — Recording Decisions

As with your other projects, record key decisions in your documentation: the approach (AI augments AFL++, it does not replace it) and its reasoning, the structured target criteria, the responsible disclosure framework, the chosen tools (AFL++, local LLM) and why, and the reuse of your generative RAG skills. Having this in writing will serve well for the README, help maintain consistency, and show that you have made deliberate, well-considered decisions.

---

## Verification: The "Definition of Done"

This phase is complete when the following criteria are met:

- [ ] The repository contains the structure, the `uv` environment, code quality tools, and the Makefile; and the `corpus/` and `crashes/` directories are included in the `.gitignore`.
- [ ] You understand fuzzing, coverage-guided fuzzing (AFL++), and the validity barrier, and have documented this in `docs/context.md`.
- [ ] The approach is defined and documented: AI augments AFL++ (does not replace it), along with its reasoning and the structured target criteria.
- [ ] The responsible disclosure framework is documented in `docs/responsible_disclosure.md` (dual use, constructive focus, responsible disclosure).
- [ ] **The key test:** You understand fuzzing and why AI adds value primarily at the validity barrier, and you are clear on the responsible framework.

The key test has two halves reflecting the dual nature of this phase: comprehension (understanding fuzzing and where AI helps) and judgment (having established the honest approach and the responsible framework). The fact that the second half is so important is characteristic of this project: *getting the approach and responsibility right* is just as decisive as understanding the technical elements. A project that overstates the role of AI loses credibility, and one that mishandles a vulnerability is irresponsible, no matter how well-constructed the fuzzer is.

---

## Deliverables and What Comes Next

By closing Phase 0, you have laid a solid foundation: the engineering base is ready, you have a firm understanding of coverage-guided fuzzing and where ML helps, and, above all, the project approach is defined with clear criteria (AI augments AFL++, it does not replace it) and the responsible disclosure framework is documented. You have started the project exactly where a professional should: by ensuring you understand the problem and setting it up with honesty and responsibility before writing a single line of code.

The next step, **Phase 1**, moves you into hands-on fuzzing: **the baseline fuzzer (AFL++) and the validity barrier**. You will install AFL++, choose a target with structured input (compiling it with instrumentation and AddressSanitizer), fuzz it with AFL++ alone to establish the **baseline**, and see the validity barrier firsthand: how AFL++ gets stuck, showing minimal coverage of deep logic because its inputs are rejected during parsing. This is the baseline to measure against and the limitation your AI will address. You have progressed from "I understand the domain and have defined the approach and responsibility" to being ready for "I have the AFL++ baseline and have seen the problem I will solve with my own eyes."
