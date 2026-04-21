# Claude Skills for Mathematical Visualization: `algorithm-math-visualizer` & `pseudocode-extractor`

**Author:** Brooke Stevens for DS 5690 Generative AI Models in Theory & Practice

---

## Motivation and Personal Anecdote

This work is deeply personal to me, and I am genuinely excited to give something back to future students of this course.

My undergraduate background is in mathematics. As a visual learner, I consistently found that the mode of instruction did not match the way I actually processed and understood the material. Professors would write the "elegant" one-liner on the blackboard, the compact expression that captured everything important about a method or algorithm, and move on. But for me, that was rarely enough. Without a worked visual example to ground the notation, it was extraordinarily difficult to build intuition about what was actually happening. Every time, I had to go home and work through a concrete example myself, because it was simply never included in the instruction.

Even today, with a much stronger foundation in mathematical notation, multi-dimensional spaces and tensor operations can still make my head spin without a numerical example to anchor them. Formal mathematical notation is powerful and precise, but it is not always inherently intuitive.

DS 5690 dedicates a substantial portion of the semester to *Formal Algorithms for Transformers* (Phuong & Hutter, 2022), a paper written in exactly this kind of compact, formal mathematical style. For students without significant prior experience reading this style of notation, the paper can feel like a wall. As someone with expertise in both mathematics and in teaching and tutoring math, I wanted to build something that could lower that wall: a Claude skill that excels at taking formal pseudocode or mathematical algorithms and rendering them as clear, step-by-step numerical traces.

I would strongly encourage the use of these tools in DS 5690 alongside the investigation of the formal algorithms paper. As a math student, having access to a worked visual example for every algorithm I studied would have been a tremendous benefit to my education.

---

## Skills Overview

Two skills are provided. They are designed to work independently or in sequence with one another.

---

### Skill 1: `algorithm-math-visualizer`

**Purpose:** Given any algorithm in pseudocode, produce a complete, step-by-step numerical trace using small, human-readable example values.

This skill is optimized for the algorithms in *Formal Algorithms for Transformers*, but it works on any pseudocode. The output is a linear, scrollable document (no interaction, no collapsible sections) that walks through every line of the algorithm with concrete matrices and vectors displayed in rendered LaTeX.

#### What It Does

For each algorithm provided, the skill will:

1. Select small, non-trivial example dimensions (e.g., sequence length $T = 3$, embedding dim $d = 2$) and display them in a labeled dimension table.
2. Define all input matrices, weight matrices, and biases with explicit LaTeX display blocks.
3. Step through every line of the pseudocode in order, showing the formula symbolically, then substituting in the actual matrices, then computing the result.
4. Handle operations like Softmax column-by-column, verifying that outputs sum to 1.
5. Present a final output section interpreting what each column or row of the result represents physically.

#### Input

The user provides one of the following:

- The name of an algorithm from the Phuong & Hutter paper (e.g., *"walk me through Algorithm 3, Attention"*), with no pseudocode required (the skill loads the reference internally).
- Any pseudocode block (or an image of one) pasted directly into the chat.

Natural-language trigger phrases include: *"show me an example of"*, *"walk me through"*, *"trace through"*, *"what do the matrices look like"*, *"concrete values for"*, or simply pasting pseudocode and asking what happens to the numbers.

Optionally, the user may specify custom example values or the number of loop iterations to trace. If not specified, the skill selects sensible small defaults and proceeds immediately.

#### Output Format

All matrices and vectors are rendered as `$$\begin{bmatrix}...\end{bmatrix}$$` LaTeX display blocks. No ASCII grids, no code blocks, no Markdown pipe tables. Each step follows a fixed structure:

1. The formula in display LaTeX.
2. The formula with actual matrices substituted in.
3. The multiplication or operation carried out with both operands and the result shown.
4. The result matrix with its shape annotated.
5. An optional plain-text note on the geometric or conceptual meaning of the result.

#### Reference Files

The skill loads `references/formal-algos-transformers.md` automatically when the algorithm belongs to the Phuong & Hutter paper. This reference encodes the notation, algorithm numbering, and conventions of that paper so the skill can proceed without the user supplying pseudocode manually.

#### Directory Structure

```
algorithm-math-visualizer/
├── SKILL.md                              # Trigger description, output format spec,
│                                         # formatting rules, and example-value guidelines
└── references/
    └── formal-algos-transformers.md      # Full notation, algorithm definitions, and
                                          # conventions from Phuong & Hutter (2022);
                                          # loaded automatically for Transformer algorithms
```

---

### Skill 2: `pseudocode-extractor`

**Purpose:** Given a research paper (as an uploaded PDF or linked text), identify the core algorithmic contributions and rewrite them as clean, formal pseudocode in the style of Phuong & Hutter (2022).

Reading research papers is one of the most important and most difficult skills in this field. Dense prose, inconsistent notation, and the gap between a paper's described method and its actual implementation all create barriers to comprehension. This skill bridges that gap by reducing any paper's algorithmic content to a concise, self-contained formal specification.

#### What It Does

The skill works in five internal phases:

1. **Read and understand**: Identifies the core algorithmic contributions, separates them from experimental setup and motivation, and maps out the inputs, outputs, and shapes involved.
2. **Plan the structure**: Determines how many distinct Algorithm blocks are needed, orders them by dependency (sub-routines first, top-level last), and identifies shared parameters.
3. **Write the pseudocode**: Produces one Algorithm block per logical unit, using math notation rather than code idioms. Loops that are simply reductions are replaced with summation notation. Sequential or order-dependent loops are preserved.
4. **Write the notation table**: Produces a unified table of every symbol used across all algorithms, with shape and type annotations.
5. **Review**: Checks that all symbols are defined, all inputs and outputs are typed and shaped, and no code idioms remain.

#### Input

The user uploads a research paper as a PDF or provides a link, then asks for pseudocode extraction. Trigger phrases include: *"extract the algorithm"*, *"write pseudocode for this paper"*, *"formalize this method"*, *"convert this paper to pseudocode"*, or simply *"can you pseudocode this?"*

#### Output Format

The output is a clean reference sheet, not a summary or review. It contains:

1. A one-line title identifying the source paper.
2. Algorithm blocks in dependency order, each beginning with a short descriptive label and followed by the formal pseudocode with typed, shaped inputs and outputs.
3. A notation table covering all symbols.
4. Brief notes (if needed) flagging inferred shapes or notational ambiguities.

#### Reference Files

The skill loads `references/phuong-hutter-style.md` before writing any pseudocode. This reference encodes the precise formatting conventions, notation standards, and style rules of the Phuong & Hutter paper, ensuring all output is consistent with that style regardless of the source paper's own notation.

#### Directory Structure

```
pseudocode-extractor/
├── SKILL.md                              # Trigger description, five-phase extraction
│                                         # workflow, output format spec, and review checklist
└── references/
    └── phuong-hutter-style.md            # Style rules, formatting conventions, and notation
                                          # standards from Phuong & Hutter (2022); loaded
                                          # before any pseudocode is written
```

---

## Design Rationale

A few design choices deserve explanation.

**Why `$$\begin{bmatrix}...\end{bmatrix}$$` as the *only* acceptable output for matrices:** Early iterations of the visualizer produced mixed output: sometimes ASCII grids, sometimes code blocks of Python-like arrays, sometimes Markdown pipe tables, occasionally prose descriptions ("a 2×2 identity matrix"). Each of these modes has a failure case: ASCII misaligns on wide matrices, code blocks invite the reader to treat the output as runnable rather than pedagogical, pipe tables collapse visually in some clients, and prose descriptions defeat the entire point of a visual trace. Collapsing the allowed set to a single mandatory rendering mode is what makes the output reliably readable. The SKILL.md codifies this as a hard rule with an explicit list of forbidden alternatives.

**Why reference files rather than inlining everything in SKILL.md:** Both skills load a companion reference file from `references/` before producing output. This is a deliberate separation tuned to this course's use case. SKILL.md describes what the skill does and when to trigger it. The reference file carries course-specific content: the full notation, algorithm numbering, and conventions of *Formal Algorithms for Transformers* (Phuong & Hutter, 2022), so that the visualizer can trace any algorithm in that paper by name without the student needing to paste pseudocode, and so that the extractor produces output conforming to the same notational standard DS 5690 students are already learning to read. Keeping this material in `references/` rather than inlining it in SKILL.md means the course-specific payload only loads when it is actually needed, the Phuong & Hutter reference can be updated or extended without touching the trigger logic, and SKILL.md stays short enough to behave reliably under Claude's context-loading patterns.

**Why five phases in the pseudocode extractor, ending with a review pass:** Extracting an algorithm from a paper is not a one-shot translation task. Papers routinely conflate motivation with method, leave shapes implicit, or reuse a symbol for two purposes. A single-phase "just write the pseudocode" prompt tends to produce output that is fluent but wrong in ways that are hard to spot. Breaking the work into *understand → plan → write → notate → review* forces each concern to be handled on its own, and the final review pass is the step where most small bugs get caught (an undefined symbol, a stray `for i in range(n)`, an inferred shape that was never flagged).

**Why no interactive elements:** A natural instinct is to make the visualizer interactive, with sliders for dimensions, collapsible steps, a button to recompute. I deliberately forbade all of this. A linear scrolling document matches how students actually read proofs and algorithm traces, with every step visible at once.

---

## Composing the Two Skills for Research Papers and Notes

The two skills are designed to chain naturally. Starting from any research paper or notes document:

1. Use `pseudocode-extractor` to convert the paper's algorithmic content into formal pseudocode.
2. Use `algorithm-math-visualizer` on the resulting pseudocode to produce a step-by-step numerical trace.

This two-step pipeline takes any research paper or notes document and produces an approachable visual walkthrough of exactly what the algorithm does to a small set of numbers. The goal is to make the algorithms in any paper accessible to any reader, regardless of their prior familiarity with the notation.

Beyond the Phuong & Hutter paper covered in DS 5690, this workflow applies equally to any paper a student encounters in research, coursework, or independent study.

---

## Demos

- [Algorithm Math Visualizer](https://claude.ai/share/45dba08e-81bf-48dd-b4da-b7fcdd26ebde) for *Formal Algorithms for Transformers* (Phuong & Hutter, 2022)
- [Chained Skills](https://claude.ai/share/b39387dc-a71b-4e37-b84d-440a016b38c3) for *LoRA* (Hu et al., 2021) and [Chained Skills](https://claude.ai/share/55149040-1eb7-402b-a72b-e8900c572ac1) for *RoFormer* (Su et al., 2023)
    - Research Paper → Pseudocode → Algorithm Math Visualizer
- [Chained Skills](https://claude.ai/share/8e59ac25-0541-418b-b886-9d389f400b30) for my own class notes from MATH 3620 Introduction to Numerical Mathematics
    - Class Notes → Pseudocode → Algorithm Math Visualizer

---

## Limitations and Known Failure Modes

- **Faithful to the pseudocode, not to the paper:** If the pseudocode-extractor misreads a paper (say, infers a wrong shape or mislabels a matrix transpose), the visualizer will faithfully trace the *wrong* algorithm. The chained pipeline compounds the risk: a subtle extraction bug propagates silently into the numerical trace.
- **Sequential-dependence algorithms are harder to trace cleanly:** Algorithms with autoregressive or recursive structure produce long, repetitive traces. The linear format holds up, but the payoff per line of output is lower than for feed-forward computations.
- **No comparison to a "bare Claude" baseline:** I did not formally evaluate how much the skills improve on what Claude would produce with no skill attached. Informally, the skills produce noticeably more consistent output (always LaTeX, always with shape annotations, always with a notation table) but a rigorous A/B evaluation is future work.

---

## Model and Implementation Details

- **Model used:** Claude Opus 4.6 (during skill development and in all demo traces).
- **Built with:** Anthropic's official `skill-creator` skill, following the standard SKILL.md + `references/` layout.
- **Intended use:** Educational support for students reading formal algorithmic papers. Primary target is DS 5690's engagement with *Formal Algorithms for Transformers*, with a secondary goal of generalizing to any paper a student encounters.
- **Out of scope:** These skills are not designed to verify correctness of an algorithm, to benchmark model implementations, or to produce production-quality algorithm documentation. They are teaching tools.
- **License:** MIT (open for educational reuse and modification). Attribution appreciated.
- **Ethical / bias consideration.** Both skills produce output that *looks* authoritative, which can make errors hard to spot. A student who treats a trace as a proof, rather than as an illustration, risks internalizing a subtle mistake. I have tried to mitigate this by keeping traces short and reproducible by hand, and by documenting failure modes above, but the risk is worth naming directly. The skills should be used *alongside* the paper, not as a complete substitute for it.

---

## Impact and Next Steps

**Impact:** The immediate impact is for future DS 5690 students engaging with *Formal Algorithms for Transformers*: the wall that paper presents to a student unfamiliar with compact mathematical notation is now more climbable. More broadly, any student reading an ML paper for the first time can chain the two skills to get a concrete worked example on any algorithm, on demand, at a small enough scale to follow by hand.

**What this reveals:** The need for a skill like this suggests something about the gap between how ML research is written and how it is learned. Papers optimize for compactness and precision, and many students need worked examples and concrete values.

**Next steps:**
- **Evaluation:** Run a proper A/B test: Claude with the skill vs. Claude without, same prompts, measure consistency of output format, rate of arithmetic errors, and student-rated clarity.
- **Expanded reference coverage:** Extend the Phuong & Hutter reference to cover additional papers frequently taught in ML courses (e.g., *Attention Is All You Need*, the original LoRA, flash attention), so the visualizer can trace those algorithms without a prior extraction step.
- **Complementary skill for diagrams:** A logical companion skill would render the computational graph of an algorithm as a complement to the numerical trace. Notation + numbers + graph is closer to how students actually draw these things on paper when learning them.
- **Classroom deployment:** Encourage the instructor to use the skills during classroom instruction, and/or encourage students to use them on their own time.

---

## Citations

Hu, E. J., Shen, Y., Wallis, P., Allen-Zhu, Z., Li, Y., Wang, S., Wang, L., & Chen, W. (2021). LoRA: Low-Rank Adaptation of Large Language Models. arXiv preprint arXiv:2106.09685.

Su, J., Lu, Y., Pan, S., Murtadha, A., Wen, B., & Liu, Y. (2023). RoFormer: Enhanced Transformer with Rotary Position Embedding. arXiv preprint arXiv:2104.09864.

Phuong, M., & Hutter, M. (2022). Formal Algorithms for Transformers. arXiv preprint arXiv:2207.09238.
