# Diagnostic lenses

Run the described problem through these lenses before selecting any pattern. Each lens that fires is
a **finding**: name the smell, name where it shows *at this boundary* (not in the abstract), and note
the remedy — which usually names a pattern in `pattern-matrix.md`, sometimes a plain move no pattern
owns. You match patterns to a named defect, never to a hunch.

## The depth principle — is the boundary shallow?

A module has two costs: what it costs to build, and what it costs every caller to learn and use. The
second is paid over and over, by people who never read the implementation. A **deep** module has a
narrow interface in front of a large amount of work — a caller states what it wants, the module
decides how. A **shallow** module's interface is nearly the whole module: every parameter maps to a
decision the module refused to own, so reading the interface teaches you the implementation anyway.

Ask: *does the interface hide roughly as much as it exposes, or does the caller pay almost the full
cost of the complexity the module supposedly contains?* If shallow, the remedy is usually a deepening
move (see `interface-sketches.md`) or a **Facade**. Depth is not always right — a module that hides
too much becomes an unextendable black box, and some seams belong to the caller. Judge on the real
call sites.

## The SOLID lens — five questions

Each principle is a question asked of the interface, and a "yes" is the violation.

- **Single Responsibility.** Would two unrelated callers want to change this interface for two
  unrelated reasons? If it answers to more than one axis of change, split it along that seam.
- **Open/Closed.** To add the next variant, does existing code have to be edited, or can the new case
  be *added*? If every new case means reopening the same conditional, the shape isn't open to
  extension — see **Strategy** / **State** / **Template Method** / **Visitor**.
- **Liskov Substitution.** Can any implementation stand in for another without a caller special-casing
  which concrete one it holds? If a caller must know the concrete type to use the interface
  correctly, the abstraction is false.
- **Interface Segregation.** Does the interface force a caller to depend on methods it never calls? An
  interface where each caller uses a different third of it is several interfaces wearing one name —
  segregate into role interfaces. No GoF pattern performs this split; it's a plain decomposition.
- **Dependency Inversion.** Does high-level policy depend on a concrete low-level detail — a specific
  library, transport, or store? If swapping that detail forces a change in the policy, invert it
  behind an interface the policy owns (often **Strategy** or a plain interface + **Adapter** at the edge).

## The leakage tests

A leak is any place a caller must know something about the inside to use the outside correctly. Leaks
show up as friction at the seam, in a few recognizable shapes:

- **A parameter that only makes sense with inside knowledge.** A flag whose correct value depends on
  which internal code path the module takes is the caller doing part of the module's job from outside.
- **A required call order.** If callers must `open` before `write` before `close`, or `validate`
  before `save`, the module has externalized its own state machine. Remedy: widen responsibility per
  call (deepening), or **Facade**.
- **Repeated call-site choreography.** The same multi-call sequence, in the same order, at two
  unrelated call sites, is a missing method — and every copy can drift. Remedy: pull the sequence
  behind the interface (**Facade** / a widened call).
- **A change inside forcing a change outside.** The sharpest test. If renaming a private field,
  swapping a data structure, or changing an internal algorithm forces editing a caller, the interface
  was never hiding that detail. (This is "shotgun surgery" below.)

Not every exposed detail is a leak — a module built to give the caller control of something (a
transaction boundary, an event ordering guarantee) is meant to expose it. The test is about
*accidental* exposure.

## The smell → remedy catalog

Each row is a smell, how it shows *at a boundary*, and the remedy — a `pattern-matrix.md` pattern, a
deepening move, or a plain split. Some restate a leak in smell vocabulary; the overlap is deliberate.

| Smell | How it shows at a boundary | Remedy |
|---|---|---|
| God class / object | One interface keeps growing and nearly every call site depends on it. | Split by responsibility (SRP); **Facade** over a coherent subsystem; **Strategy** for the varying policies. |
| Feature envy | A caller reads several fields off a module, then computes what the module itself should have decided. | Move the behavior to the data (deepening); where the data is a stable structure the behavior can't move into, gather the operation into a **Visitor**. |
| Shotgun surgery | One conceptual change forces edits across many call sites ("change inside forces change outside"). | Pull the repeated choreography behind the interface; often **Facade** or **Template Method**. |
| Primitive obsession | Bare strings/ints/maps carry an invariant every call site re-validates by hand. | A small type that makes the wrong state unrepresentable; **Builder** where construction is genuinely step-wise. |
| Reinvented / one-off data structure | A boundary invents a bespoke shape for a meaning an existing type already carries. | Reuse the existing type; where genuinely new, name it once as a shared type rather than an inline one-off every call site re-learns. |
| Fat / leaky interface | A required call order, or a flag that only makes sense with inside knowledge. | Segregate (ISP); widen responsibility per call; **Facade**. |
| Long parameter list | A call takes many arguments most callers copy unchanged. | Default the common case; a parameter object; **Builder** for step-wise construction. |
| Subclass explosion | Class count multiplies as independent axes cross (a subclass per feature-combination). | **Decorator** to stack responsibilities, or **Bridge** to split the axes. |
| Type/state switch sprawl | The same switch on a type or state field recurs, and a new case means editing each one (OCP). | **Strategy** (behavior selection) or **State** (behavior by internal state); **Visitor** to add operations over a stable structure; **Chain of Responsibility** where handlers form a pipeline. |
| Two mechanisms, one concern | A shape implements more than one branch for the same work, built differently — one composes through an interface, a sibling reimplements the equivalent on its own. | Route every branch through the same seam — **Strategy** for the varying cases, the divergent branch pulled behind the interface its sibling already uses. |
| Hard-coded concrete dependency | High-level code `new`s a specific concrete class (a store, client, format), so it can't be swapped or tested in isolation. | Invert behind an interface the policy owns (**Dependency Inversion**); **Factory Method** where the concrete type is chosen at runtime; **Abstract Factory** for a matched family. |
| Tangled many-to-many wiring | A cluster of objects each hold references to several others and coordinate directly; a change ripples across all of them. | Route interaction through a **Mediator** so each object knows only the mediator. |
| Manual observer wiring / polling | Code polls for a change, or the producer hard-codes every consumer it must notify. | **Observer** — let dependents subscribe; the subject notifies without knowing concrete types. |
| Single-use wrapper / needless indirection | A named method or class exactly one call site reaches, whose name restates the single call it forwards. | Inline it — a seam reached once that decides nothing is a name with extra steps. Keep only where it hides genuine complexity, gathers an invariant, or the call recurs. |

This last row is the inverse of the others: not too little structure, but a boundary drawn where none
was earned. When the "problem" is really over-engineering, the remedy is to *remove* a layer, and no
new pattern is the answer.
