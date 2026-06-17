# Phase 1 — The Base Fuzzer (AFL++) and the Validity Barrier

> In this phase, you get hands-on with fuzzing for the first time, with two intertwined objectives. The first is to establish the **baseline**: measuring what AFL++ achieves on its own, because the entire project hinges on whether your AI *beats* this baseline, making it a solid and fair point of reference. The second, and nearly as important, is to see the **validity barrier** with your own eyes: when you fuzz a structured-input target and see how AFL++ gets stuck—with coverage plateauing because its inputs are rejected during parsing—you will visually and viscerally understand the problem that gives purpose to this entire project. The guiding idea of this phase is precisely that: to measure the opponent to beat, and to confront firsthand the limitation that your AI is going to target.

**Phase Objective:** Establish the baseline and see firsthand the limitation motivating the project.  
**Duration:** Weeks 1-3.  
**Upon completion, you will have:** AFL++ running on a structured target, the baseline of its performance measured, and a live, firsthand understanding of the validity barrier.

---

## The Overall Concept

The workflow of this phase gets AFL++ up and running and observes its limitations:

```
   [ 1. Install AFL++ + choose target ]  ──►  a structured-input parser (e.g., libxml2)
        │
        ▼
   [ 2. Compile with instrumentation + ASan ]  ──►  afl-clang-fast + AFL_USE_ASAN
        │
        ▼
   [ 3. Fuzz with AFL++ and measure ]  ──►  the baseline: coverage and bugs over time
        │
        ▼
   [ 4. Observe the validity barrier ]  ──►  coverage plateaus: the problem, live
```

---

## Step 1 — Install AFL++ and Choose the Target

The easiest way to get AFL++ fully compiled and ready is using the official Docker image, which also keeps your host system clean:

```bash
docker pull aflplusplus/aflplusplus
docker run -ti -v $(pwd):/src aflplusplus/aflplusplus
```

Choosing the target is the key decision of this phase, and it materializes the criteria you established in Phase 0: a target with **structured** input, because that is where the validity barrier exists and where your AI will make a difference. A canonical and highly documented target is **libxml2** (an XML parser): XML is structured, AFL++ has its own tutorial for it, and it includes the `xmllint` tool, which serves as the *harness* (the program that receives input from the fuzzer and passes it to the code under test) and reads an input file. Any parser of a known structured format will work just as well; libxml2 is a solid choice to start with.

---

## Step 2 — Compile with Instrumentation and AddressSanitizer

To allow AFL++ to guide itself using coverage, you compile the target with its **instrumentation** (`afl-clang-fast`) and, applying what you learned in the fundamentals, with **AddressSanitizer** (`AFL_USE_ASAN=1`), so that memory bugs that do not cause a visible crash are brought to light:

```bash
cd libxml2
# AFL++ Instrumentation + AddressSanitizer
AFL_USE_ASAN=1 ./configure --disable-shared
AFL_USE_ASAN=1 make CC=afl-clang-fast CXX=afl-clang-fast++
```

Here are a couple of details that demonstrate your knowledge of the tool. Always compile libraries **statically** (`--disable-shared`) when fuzzing to avoid issues with shared libraries. Also, when fuzzing with ASan later on, remember to use `-m none` (no memory limit): ASan reserves a massive amount of virtual memory, so any limit would terminate it prematurely. The combination of instrumentation (for coverage) and ASan (for making bugs visible) is the foundation of effective fuzzing.

---

## Step 3 — Fuzz with AFL++ and Measure the Baseline

With the target compiled, you prepare a seed corpus (valid starting inputs) and launch AFL++:

```bash
mkdir -p fuzz/in
cp test/*.xml fuzz/in/        # seeds: valid XMLs from libxml2's own tests
cd fuzz
# @@ is the input file; -m none for ASan
afl-fuzz -i in -o out -m none -- ./xmllint_cov @@
```

Upon starting, AFL++ displays a live *status screen*, and understanding what it shows means understanding fuzzing in action. Pay close attention to: **executions per second** (fuzzing speed), total **paths** (a measure of coverage: how many distinct program behaviors have been reached), **unique crashes** (bugs found), and the density of the coverage map. Let the fuzzer run for a decent amount of time (for example, several hours or a day) and record how coverage and crashes evolve over time: this is your **baseline**.

And here is a crucial point for the integrity of the project: the baseline must be **fair**, meaning AFL++ is giving its absolute best. AFL++ comes with powerful features that are worth enabling, such as **CMPLOG** (`AFL_LLVM_CMPLOG=1`, which is compiled as a separate binary) to help AFL++ bypass difficult comparisons like *magic bytes* and checksums. Using a well-configured AFL++ as your baseline (rather than a stripped-down version) is what will make proving your AI beats it actually mean something later on. A weak baseline would invalidate the entire comparison.

---

## Step 4 — Observe the Validity Barrier

This is conceptually the most important part of this phase, and it is worth pausing here, because it makes the very problem that gives the project its purpose tangible. While AFL++ fuzzes your structured target, observe how coverage behaves over time. What you will see, especially with a complex format, is that coverage **grows rapidly at first and then plateaus**: AFL++ easily explores shallow code paths (input validation, rejection of malformed inputs), but struggles immensely to reach deep processing logic because most of its random mutations produce invalid XML that the parser rejects right at the start.

This plateau *is* the validity barrier, witnessed live. To make it even more tangible, you can measure which parts of the code are reached (recompiling with coverage instrumentation and generating a report on discovered paths) and verify that the deep logic of the parser is barely touched. You are seeing, with your own eyes, why AFL++ alone is not enough for a structured target: not because its exploration is poor, but because it fails to generate the valid inputs required to go deep. This is exactly what your AI will tackle in the next phase, generating valid inputs that overcome this barrier. Confronting this problem firsthand is what provides meaning and motivation for everything to come, reaffirming—with evidence right in front of you—the approach you established in Phase 0.

---

## Step 5 — Verify

Since this phase primarily produces a fuzzing campaign and metrics rather than complex code, verification consists of checking that everything works and that you have established your baseline. Confirm that: the target compiles with instrumentation and ASan; AFL++ starts and fuzzes (you can see executions per second and paths growing on the status screen); you have recorded the evolution of coverage and crashes (the baseline); and you have observed the coverage plateau (the validity barrier). A simple way to document the baseline is to save, at regular intervals, the statistics that AFL++ writes to the `fuzzer_stats` file inside the output directory.

This check confirms that you have accurately measured the opponent to beat and have witnessed its limits, which is exactly what you need to move forward. It verifies that the foundation—a fair and thoroughly understood baseline—is properly laid before building on top of it.

---

## Verification: The "Definition of Done"

The phase is finished when the following are completed:

- [ ] You have installed AFL++ and chosen a target with structured input (a parser).
- [ ] You have compiled the target with instrumentation (`afl-clang-fast`) and AddressSanitizer (`AFL_USE_ASAN=1`).
- [ ] You have fuzzed with AFL++ and measured the baseline (coverage and crashes over time) with a well-configured AFL++ setup (a fair baseline).
- [ ] You have observed the validity barrier: the plateau in coverage and that deep logic is barely reached.
- [ ] **The key test:** You have a measured baseline and have seen that AFL++ does not reach deep logic due to the validity barrier.

The key test has two parts reflecting the dual objective of this phase: having a measured **baseline** (the opponent to beat, fairly configured) and having **witnessed the validity barrier** (the problem to solve). That both matter is what characterizes this project: the baseline is the reference against which everything else will be measured, and confronting the validity barrier firsthand is what gives purpose to the AI you are going to build. It is not enough to just get AFL++ up and running; you must properly measure its performance and understand, by observing it, why it falls short.

---

## Deliverables and What Comes Next

Upon closing Phase 1, you have the two core foundations of the project: a well-measured AFL++ baseline (a fair and solid opponent against which you can prove, later on, whether your AI adds value) and a live, firsthand understanding of the validity barrier (how AFL++ plateaus on a structured target because its inputs are rejected during parsing). You have used standard fuzzing tools (instrumentation, ASan, the status screen) and confronted the very problem your AI is designed to solve.

The next step, **Phase 2**, is the most distinctive AI component: **ML/LLM-based input generation**. You will build a generator using a local LLM (reusing your RAG) that produces inputs that are both *valid* for the target's format (XML, in this example) and *diverse*, in order to overcome the validity barrier you have just witnessed. This is a direct attack on the limitation you observed: providing AFL++ with the valid inputs it cannot generate on its own. You have progressed from "I have the baseline and have seen the validity barrier" to being on the verge of "I have an AI that generates the valid inputs to overcome it."
