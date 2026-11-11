================================================================================
           CLASSICAL AI REASONING SYSTEM - SUBMISSION README
================================================================================

PROJECT TITLE: General-Purpose Classical AI Reasoning System for Entailment Verification
COURSE:        Artificial Intelligence
DEPARTMENT:    Computer Science and Engineering
ROLL NUMBER:   M26CSA014

--------------------------------------------------------------------------------
1. SUBMISSION MANIFEST
--------------------------------------------------------------------------------
The final submission archive (submission.zip) contains strictly the four required deliverables:

submission/
|-- reasoning_agent.ipynb  (Completely self-contained Jupyter Notebook implementing the
|                           streamlined classical First-Order Logic reasoning pipeline, unification,
|                           natural language semantic parser, direct logic formula AST solver,
|                           all 33 regression tests, public dataset evaluations, step-by-step
|                           reasoning traces, embedded offline fallback data, and submission.csv
|                           generation with 0 external dependencies)
|-- report.pdf             (Strictly 3-page comprehensive technical report covering
|                           dual input modes, representations, parser precedence, OWA vs CWA
|                           semantics, empirical benchmarks, latency profiling, and traces)
|-- submission.csv         (Primary CSV containing the 85 authentic evaluation IDs and
|                           predicted boolean labels)
`-- README.txt             (Execution instructions, dual-mode architecture, and documentation)

Note: The notebook submission/reasoning_agent.ipynb is completely self-contained and executes cleanly
with zero dependencies on external project Python files, internet access, or local data directories.

--------------------------------------------------------------------------------
2. SYSTEM ARCHITECTURE & DUAL INPUT MODES
--------------------------------------------------------------------------------
The system supports two distinct input modes sharing a unified symbolic reasoning core:

MODE 1: NATURAL LANGUAGE INPUT (NL -> Internal FOL -> Inference -> True/False)
  Example:
    Context: Bob is young. If someone is young then they are smart.
    Query:   Bob is smart.
  Semantic Parser Hierarchy:
  - Disjunctive Partitions: "There are N types of Category: Type1, Type2, ..." (CNF disjunctive clauses)
  - Classical Neither...Nor: "X is neither A nor B" -> ~A(X) ^ ~B(X)
  - Conditional Rules: "If <premises>, then <consequent>" (with antecedent subject anaphora)
  - Universal Quantified Rules: "All / Every <props> <noun> are/is [not] <concl>"
    (e.g., "All furry, round people are red" -> furry(?x) ^ round(?x) => red(?x))
  - Generic Bare Plural Rules: "<props> <generic_noun> are/is [not] <concl>"
    (e.g., "Kind, young things are not smart" -> kind(?x) ^ young(?x) => ~smart(?x))
  - Ground Facts: Ground copula, relational, perfect-tense, and ternary sentences
  - Causal Networks: Extracts direct and joint causal dependencies ("X causes Y", "A and B cause C")

MODE 2: DIRECT LOGIC INPUT (Logic Formula -> AST / CNF -> Inference -> True/False)
  Per the professor's explicit instruction:
  "In the Assignment you will be given logic directly for evaluation."
  Supports standard formal notation directly without natural-language parsing:
  - Connectives: Negation (~, ¬), Conjunction (&, ∧), Disjunction (|, ∨),
                 Implication (->, →, =>), Biconditional (<->, ↔, <=>)
  - Quantifiers: Universal ("forall x", "∀x"), Existential ("exists x", "∃x")
  - Multi-argument predicates: Parent(John, Mary), Grandparent(x, z)
  - Direct Formula Solver:
    * Classical syllogisms & universal instantiation via Generalized Modus Ponens
    * First-order resolution refutation with Set-of-Support and factoring
    * Sound Principle of Explosion (Ex Falso Quodlibet): contradictory KBs entail any formula
      with explicit empty clause derivation traces
    * Multi-argument transitive chaining and Herbrand existential witness search

--------------------------------------------------------------------------------
3. ARCHITECTURAL SIMPLIFICATION & OPTIMIZATIONS
--------------------------------------------------------------------------------
The simplified engine reduces code bulk by ~44% (from 2,279 lines across core modules
down to 1,276 lines in simplified_core.py) while preserving 100% of all reasoning capabilities:
1. Unified Resolution Refutation Engine: Consolidated duplicate CNF refutation loops,
   combining KB inconsistency verification and goal-refutation into a unified solver.
2. Compact AST Node Hierarchy: Standardized binary operations (And, Or, Implies, Iff)
   under a shared BinaryNode structure, dramatically pruning redundant visitor classes.
3. Streamlined Forward/Backward Inference: Integrated subsumption and occurs-checked
   unification directly into core chaining loops without extraneous wrappers.
4. Preserved Full Expressivity: Retained all natural language parsing precedence rules,
   structural counterfactual reasoning, stratified NAF under CWA, and open-world semantics.

--------------------------------------------------------------------------------
4. OFFLINE DATASET FALLBACK ARCHITECTURE
--------------------------------------------------------------------------------
To guarantee 100% standalone reproducibility when executing without network connectivity
and without an external data/ directory, the notebook incorporates verified compressed
offline evaluation caches (_B64_AUTH_DATA and _B64_SYNTH_DATA):
- Loading Priority:
  1. Online Hugging Face API (when network is connected or force_online=True).
  2. Local data/ directory (if provided).
  3. Embedded verified offline cache (automatic fallback when offline).
- Sample Counts Verified:
  * RuleTaker (Depths 0-5): 50 authentic test instances
  * FOLIO Validation Set:   15 authentic validation instances
  * CounterBench Set:       20 authentic causal counterfactual instances
  * Synthetic Benchmark:    52 instances
  * Causal Semantics:       6 instances
- Integrity Guarantee: If both online download and embedded fallback fail, a descriptive
  RuntimeError is raised immediately. Empty dataset lists are never silently passed to evaluators.

--------------------------------------------------------------------------------
5. VERIFIED EMPIRICAL RESULTS (ALL 8 BASELINES PRESERVED)
--------------------------------------------------------------------------------
1. Initial Automated Unit Validations: 7/7 PASSED (100.0%)
2. Direct Logic & Proof Status Suite:   33/33 PASSED (100.0%)
3. Direct Logic CSV Sample:             10/10 PASSED (100.0%)
4. RuleTaker Test Split (Depths 0-5):   50/50 PASSED (100.0%)  [Avg latency: 0.235 ms/q]
5. FOLIO Validation Split (OWA):        11/15 PASSED (73.33%)  [Avg latency: 0.330 ms/q]
6. CounterBench Benchmark Split:        20/20 PASSED (100.0%)  [Avg latency: 0.011 ms/q]
7. Synthetic Regression Suite:          52/52 PASSED (100.0%)
8. Causal Semantics Suite:              6/6 PASSED (100.0%)

Combined Authentic Public Dataset Accuracy: 81/85 (95.29%)

--------------------------------------------------------------------------------
6. INPUT REQUIREMENTS FOR PRIVATE EVALUATION FILES
--------------------------------------------------------------------------------
The system supports evaluating arbitrary external or private test files:
- CSV Format: Columns `id`, `context` (or `kb`), `query` (or `question`), and optional `label`.
- JSON / JSONL Format: Objects with keys `id`, `context` (or `premises`), `query` (or `conclusion`), and `label`.
- Plain Text Format: Consecutive premise lines followed by a query line.
- Direct Logic Mode: `DatasetLoader.load_logic_file("test.csv")` automatically activates Mode 2.
- Submission Generation: Calling `DatasetLoader.generate_submission_csv()` writes working-directory
  relative `submission.csv` with columns `id,prediction` using the original sample IDs.

--------------------------------------------------------------------------------
7. SYSTEM LIMITATIONS & SOUNDNESS BOUNDARIES
--------------------------------------------------------------------------------
1. First-Order Logic Semi-Decidability: Entailment in unrestricted FOL is semi-decidable.
   For unprovable queries in infinite domains, resolution refutation terminates at `max_steps`
   and soundly reports `logical_state="unresolved"` with status `resource_limit_reached`.
2. Open-World Assumption: In Mode 1 (OWA) or Mode 2, failure to prove a positive query does
   not prove its negation. The system strictly distinguishes `undetermined` (non-entailed)
   from `contradicted` (negation proved).
3. Natural Language Ambiguity: Complex nested subordinate clauses or non-classical idioms
   outside the precedence grammar fall back to unparsed sentence tracking without crashing.

--------------------------------------------------------------------------------
8. EXECUTION INSTRUCTIONS
--------------------------------------------------------------------------------
Option 1: Run the Self-Contained Jupyter Notebook
  Execute submission/reasoning_agent.ipynb in Jupyter Lab, VS Code, or Google Colab.
  All cells run sequentially from top to bottom with zero external file requirements.

Option 2: Programmatic Execution via Python
  from reasoning_agent import ClassicalReasoningAgent
  agent = ClassicalReasoningAgent()

  # Natural language query
  res_nl = agent.answer_query("Bob is young. If someone is young then they are smart.", "Bob is smart.")
  print("NL Result:", res_nl["prediction"])

  # Direct logic query
  res_logic = agent.answer_logic(["Human(Socrates)", "forall x (Human(x) -> Mortal(x))"], "Mortal(Socrates)")
  print("Logic Result:", res_logic["prediction"])
================================================================================
