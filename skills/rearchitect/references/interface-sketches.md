# Interface sketches

A candidate pattern is worthless to the developer until they can see the *interface* it produces —
what the caller writes after the refactor, and what moved behind the boundary. A sketch is that shape,
in the codebase's actual language, at just enough detail to judge the trade. It is not the
implementation. This file is how to write one that carries its weight.

## What a sketch contains

For each selected pattern, show:

1. **The new boundary** — the type(s), interface, or method signatures the refactor introduces, in
   the language the code is actually written in (PHP, TS, Python, whatever the problem is in). Match
   the existing naming and idiom; a sketch in a foreign style is noise.
2. **The caller's surface** — one short before/after of what a representative call site writes. This
   is where the win or the cost becomes visible. Keep it to the line or two that changes.
3. **What moved behind the interface** — name the complexity the caller no longer sees, or the
   decision it no longer makes. If nothing moved, the pattern bought nothing.
4. **Buys / Costs** — the specific change that becomes easy, and the honest price (new types, an added
   hop for the reader, indirection). Tie both to *this* problem, not to the pattern in general.

Keep each sketch tight — a dozen lines of signatures and a before/after call site, not a working
module. The developer is choosing between shapes, not reviewing a PR.

## The "who decides what" test

The core of every good sketch is a deliberate re-allocation of a decision. A refactor that only
renames things or moves code between files without changing *who decides what* is not a refactor worth
proposing. For each candidate, be able to state the decision that moved:

- Strategy: the *choice of algorithm* moved from a hard-coded conditional to the caller (or a config).
- Facade: the *orchestration order* moved from every call site into one entry point.
- Factory Method: the *choice of concrete class* moved from the call site to a factory/subclass.
- Decorator: the *choice of which behaviors to combine* moved to composition at runtime.

If you can't name the decision that moved, the sketch isn't showing a real refactor — reconsider
whether the trigger actually fired.

## Deepening moves — for when the remedy is plain, not a pattern

Often the honest remedy is not a GoF pattern but a plain deepening move. Sketch these the same way.
Each moves a cost the caller was paying into the module, where it's paid once.

1. **Pull complexity down behind the interface.** If a caller orchestrates several of the module's
   concerns from outside, pull that orchestration inside. The caller states the outcome, not the steps.
2. **Widen responsibility per call.** If callers make a fixed sequence of calls in the same order to
   get one outcome, give the sequence one name. Now they can't run it out of order or skip a step.
3. **Collapse pass-through layers.** If a layer's method only calls the equivalent method below it —
   same name, same args, no added decision — collapse it. A layer that translates, enforces a policy,
   or caches is doing work; one that only forwards is a detour.
4. **Default the common case.** If nearly every call site passes the same values for the same optional
   parameters, default them and put the rest behind one escape hatch.

## Honest costs — always name them

Every pattern has a price, and the developer is trusting you to name it so the recommendation means
something. Common costs to state plainly:

- **Indirection** — one more hop from call site to the code that does the work; harder to trace.
- **New types / files** — more surface to name, place, and navigate.
- **Premature generality** — an interface with one implementation is a strategy that bought nothing
  yet; say when the pattern only pays off once the *second* case actually arrives.
- **Reader cost** — a pattern the team doesn't already use is a tax on everyone who reads it next.

A sketch whose Costs line is blank is a sales pitch, not a proposal. If a pattern genuinely has near-zero
cost here, say *why* — don't just omit it.

## Example sketch (shape, not content to copy)

```
### 1. Strategy — swap the pricing rule without reopening Checkout
Interface sketch (PHP):

    interface PricingRule {
        public function apply(Cart $cart): Money;
    }
    // StandardPricing, MemberPricing, PromoPricing implement it.

    final class Checkout {
        public function __construct(private PricingRule $rule) {}
        public function total(Cart $cart): Money { return $this->rule->apply($cart); }
    }

Caller, before:  $total = $checkout->total($cart, $isMember, $promoCode);   // switch inside
Caller, after:   $total = (new Checkout($rule))->total($cart);              // rule chosen at wiring

Moved: the choice of pricing rule moved out of Checkout's switch into the caller's wiring.
- Buys: a new pricing rule is a new class, no edit to Checkout (OCP).
- Costs: one interface + one class per rule; the rule must be selected somewhere upstream.
```
