# Phase 2 — Input Generation with ML/LLM

> **This is the most distinctive AI part of the project.** In Phase 1, you saw AFL++ stuck at the validity barrier: unable to generate, through random mutation, the valid structured inputs required to reach deep logic. Here, you will build the component that solves exactly that: a generator that uses an LLM to produce **valid** inputs (which pass parsing, overcoming the barrier) and **diverse** inputs (which explore different edge cases of the format). This is the most valuable AI contribution because it gives AFL++ precisely what it cannot achieve on its own: valid and varied inputs to bootstrap the exploration. However, there is a nuance running through this phase that proves you understand fuzzing: it is not enough for inputs to be valid; they must be **valid and diverse at the same time**, because it is diversity that drives coverage.

**Phase objective:** Generate valid and diverse inputs.  
**Duration:** Weeks 3–5.  
**By the end, you will have:** A generator that, using a local LLM, produces valid and varied inputs for the target format, ready to feed the fuzzer and overcome the validity barrier.

---

## The Big Picture

The workflow of this phase generates the fuel that AFL++ needs:

```
   The validity barrier (seen in Phase 1)
        │
        ▼
   [ Local LLM (Ollama) with varied prompts ]  ──►  generate VALID and DIVERSE inputs
        │
        ▼
   [ Validate and filter ]  ──►  keep those that pass parsing
        │
        ▼
   [ Seeds for AFL++ ]  ──►  valid and varied inputs to bootstrap exploration
```

Add the LLM dependency (the same as your RAG) and a parser for validation:

```bash
uv add llama-index-llms-ollama lxml
```

---

## Step 1 — The Concept: Why the LLM Generates Valid Inputs

It is worth understanding why the LLM is the right tool here, showing that this choice is not arbitrary. The validity barrier problem is, at its core, a problem of *knowing the structure*: to generate valid XML, you need to know what valid XML looks like (closing tags, syntax, formatting rules). And that is precisely what an LLM knows: it has seen countless structured documents (XML, JSON, HTML) during its training, so it "knows" common formats and can easily generate valid examples. Where AFL++'s random mutation produces garbage that the parser rejects, the LLM produces well-formed documents that pass through the gate.

This is not an idea forced onto the domain: it is precisely the approach applied by cutting-edge research in 2026, with systems using LLMs to generate or mutate inputs in structured formats, achieving better coverage than AFL++ alone. Moreover, it directly repurposes your RAG skills: the same local LLM via Ollama, and the same output handling. This is the AI piece that delivers what purely random fuzzing cannot: knowledge of the structure.

---

## Step 2 — Generating Valid Inputs with the LLM

The heart of this phase is the generator that requests valid inputs for the target format from the LLM. Create `src/generate/generate.py`:

```python
from llama_index.llms.ollama import Ollama

llm = Ollama(model="llama3", request_timeout=120.0)   # reused from your RAG

GENERATE_PROMPT = """Generate a VALID, well-formed XML document that exercises the following \
feature of the format: {caracteristica}.

Requirements:
- Must be valid and well-formed XML (must pass an XML parser).
- Explore uncommon but valid edge cases of that feature.
- Respond ONLY with the XML, without any explanation or extra text.
"""


def generar_una(caracteristica: str) -> str:
    respuesta = llm.complete(GENERATE_PROMPT.format(caracteristica=caracteristica))
    return limpiar(respuesta.text)   # remove markdown fences (reused from your RAG)
```

Notice that you are reusing two things from your RAG. First, the **local LLM via Ollama**, with its generous `request_timeout`. Second, the **output handling**: LLMs tend to wrap code in markdown fences (```` ```xml ... ``` ````) or add explanations, so you need to clean the response to keep only the pure XML, just as you did when parsing the structured output in RAG. The prompt explicitly requests valid and well-formed XML, which is what overcomes the validity barrier.

---

## Step 3 — Validity AND Diversity: The Key Nuance

This is the point that shows you truly understand fuzzing, and it marks the difference between a naive generator and a useful one. It is not enough to generate *valid* inputs: they must be valid **and diverse**. The reason lies in how coverage works. If the LLM generates a thousand valid but almost identical XML files, they will all exercise the same code path, doing nothing to help explore new areas—coverage won't grow. The value comes from inputs that are both valid *and varied*, because **diversity** is what drives the fuzzer down different program paths, ultimately boosting coverage.

That is why the generator actively seeks diversity, varying what it requests from the LLM to explore different features of the format:

```python
CARACTERISTICAS = [
    "attributes and namespaces",
    "deeply nested elements",
    "entities and entity references",
    "CDATA sections",
    "comments and processing instructions",
    "special characters and Unicode",
    "attributes with boundary values (very long, empty)",
    # ... each one pushes the LLM toward different inputs
]


def generar_entradas() -> list[str]:
    # Varying the feature forces diversity: each prompt explores a different part of the format
    return [generar_una(c) for c in CARACTERISTICAS]
```

The idea is simple yet crucial: by varying the prompt (requesting a different format feature each time), you force the LLM to generate inputs that exercise different parts of the parser, which translates into covering different areas of the code. This is the difference between giving AFL++ the same entry door a thousand times versus giving it many different doors. Seeking both validity *and* diversity is what makes the generator truly useful for fuzzing.

---

## Step 4 — Validate and Filter

The LLM does not always get it right: sometimes it will generate something that looks like XML but is not fully valid. Since the goal is to supply AFL++ with inputs that *pass* the validity barrier, you validate the generated inputs and keep only those that actually pass parsing. Create `src/generate/validate.py`:

```python
from lxml import etree


def es_valido(xml: str) -> bool:
    """Does the parser accept this input? (i.e., does it overcome the validity barrier?)"""
    try:
        etree.fromstring(xml.encode())
        return True
    except etree.XMLSyntaxError:
        return False


def filtrar_validas(entradas: list[str]) -> list[str]:
    return [x for x in entradas if es_valido(x)]
```

Validating by parsing is the direct check for what actually matters: that the input overcomes the validity barrier. You keep the valid ones (and, if you want to fine-tune it, you can try to *repair* those that are close, though filtering is usually enough). It is also important to be clear about the role of these inputs within the project's architecture: the generated inputs are not the final test cases, but rather **seeds** for AFL++. The AI provides valid and diverse seeds; AFL++ will take them as a starting point and mutate them to explore and find bugs. This is the collaboration you saw in the fundamentals: AI provides structural knowledge, while the fuzzer handles exploration. This is why you save them in the seed corpus:

```python
from pathlib import Path

for i, xml in enumerate(filtrar_validas(generar_entradas())):
    Path(f"corpus/gen_{i}.xml").write_text(xml)   # seeds for AFL++ (Phase 3 integrates them)
```

---

## Step 5 — Tests

Verify the logic that does not require the model (validation, output cleanup, and ensuring the prompt requests the right thing). Complete `tests/test_generate.py`:

```python
from src.generate.generate import GENERATE_PROMPT, limpiar
from src.generate.validate import es_valido, filtrar_validas


def test_la_validacion_distingue_xml_valido_de_invalido():
    assert es_valido("<root><a>1</a></root>") is True
    assert es_valido("<root><a>1</root>") is False        # unclosed tag
    assert filtrar_validas(["<a/>", "<b>"]) == ["<a/>"]   # filters invalid ones


def test_la_limpieza_quita_los_fences_de_markdown():
    sucio = "```xml\n<root/>\n```"
    assert limpiar(sucio).strip() == "<root/>"            # reused from your RAG


def test_el_prompt_pide_validez_y_explora_una_caracteristica():
    prompt = GENERATE_PROMPT.format(caracteristica="namespaces")
    assert "VALID" in prompt and "namespaces" in prompt  # validity + diversity per feature
```

The tests verify that validation distinguishes valid from invalid inputs (the foundation of overcoming the barrier), that cleanup removes markdown fences (reused from RAG), and that the prompt asks for validity while exploring a specific feature (diversity). Run them with `make test`.

---

## Verification: The "Definition of Done"

This phase is complete when the following criteria are met:

- [ ] You have built a generator using a local LLM (Ollama, reusing your RAG) that produces inputs for the target format.
- [ ] The generator seeks diversity (varying what it requests from the LLM to explore different features).
- [ ] You validate the generated inputs and keep only those that pass parsing (overcoming the validity barrier).
- [ ] You understand that the generated inputs are seeds for AFL++, not the final test cases.
- [ ] **The key test:** The AI generates inputs that pass the validity barrier and are varied.

The key test has two parts reflecting the main nuance of this phase: that the inputs **pass the validity barrier** (are valid) and that they are **varied** (diverse). The fact that both aspects matter is the hallmark of a strong fuzzing generator: validity overcomes the barrier, but diversity is what actually drives coverage. It is not enough to generate XML that parses; you must generate XML that is both valid *and* varied, which is what will help AFL++ explore.

---

## Deliverables and What Comes Next

By wrapping up Phase 2, you have built the most distinctive AI component of the project: a generator that, using a local LLM, produces valid and diverse inputs for the target format, directly tackling the validity barrier identified in Phase 1. You have successfully repurposed your RAG skills (the local LLM, the output handling) for a new problem, demonstrating the maturity to understand that true value lies in both validity *and* diversity, not just generating inputs that parse. You now have the fuel that AFL++ needs.

The next step, **Phase 3**, unites these two pieces: **the hybrid fuzzer**. You will integrate the AI-generated inputs into the AFL++ loop (as seeds and by feedback-feeding the corpus) so that the AI supplies the valid and diverse inputs, while AFL++ mutates and explores them. You will verify that this hybrid setup reaches parts of the program that AFL++ alone could not, manifesting the collaboration between structural knowledge (the AI) and guided exploration (the fuzzer). You have progressed from "I have an AI that generates valid and diverse inputs" to being on the verge of "I have a hybrid fuzzer that uses them to explore deeper than AFL++ could alone."
