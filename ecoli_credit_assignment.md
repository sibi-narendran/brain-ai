# E. coli and the Origin of Credit Assignment

## The Core Insight

E. coli solved temporal credit assignment 3.5 billion years ago using 6 proteins.
No neurons, no gradients, no loss function.

The mechanism is:
1. A fast signal (now)
2. A slow signal (recent past, stored in molecular state)
3. Their difference drives action
4. The slow signal adapts toward the fast one over time

Evolution didn't reinvent credit assignment at the neuron level. It reused the same molecular trick and scaled it.

---

## The Problem E. coli Faces

It can't sense *direction* of food. It can only sense *concentration at this instant*.

It has two modes:
- **Run** — swim straight
- **Tumble** — randomly reorient

It can't steer directly. So how does it find food?

---

## The Solution — A Molecular Delay Line

It compares two things:
- **Now** — current receptor activation
- **~1 second ago** — stored in methylation state of the receptor protein

```
If now > before → things are improving → keep running
If now < before → things are getting worse → tumble
```

That comparison across time **is** credit assignment. The "credit" goes to the
current direction of travel based on whether recent history improved or worsened.

---

## The Molecular Machinery

```
Receptor → CheA (kinase) → CheY → flagellar motor
                ↕
           CheR / CheB (methylation enzymes)
```

- **CheA** — detects ligand binding, activates downstream signaling
- **CheY** — carries signal to flagella. More CheY = tumble
- **CheR** — slowly methylates receptors, increases sensitivity (adapts to current level)
- **CheB** — demethylates, decreases sensitivity
- **Methylation state** — the memory. Encodes what "normal" was ~1 second ago

---

## The Credit Assignment Loop

1. Receptor detects concentration now
2. Methylation state encodes concentration from ~1 second ago
3. The *difference* between them drives CheA activity
4. CheA drives tumbling probability
5. If swimming was good → methylation catches up → receptor adapts → resets for next comparison

---

## The Parallel to Eligibility Traces in R-STDP

| E. coli | R-STDP |
|---|---|
| Methylation state | Eligibility trace |
| ~1 second window | Trace decay time constant |
| Chemical gradient reward | Scalar reward signal |
| Run vs tumble | Spike vs silence |
| CheR/CheB balance | Learning rate |

The methylation state is **literally** an eligibility trace implemented in chemistry.
It tags recent receptor activity so that when the "reward" signal (improving
concentration) arrives, it can modulate behavior.

---

## What E. coli Cannot Do

- Assign credit across multiple steps (only ~1 second lookback)
- Learn permanently — it resets, no long-term weight change
- Generalize — same behavior in every environment, just recalibrated sensitivity

---

## The Deeper Stack — What Is More Fundamental Than a Neuron?

Going deeper than the neuron, the common elements across all life:

| Layer | What it is | Who has it |
|---|---|---|
| Lipid membrane | Barrier that maintains voltage | All life |
| Electrochemical gradient | Inside negative, outside positive | All life including bacteria |
| Ion channels | Voltage-gated Na⁺, K⁺ | All animals, conserved for 600M years |
| ATP | Energy currency for ion pumps | All life |
| Feedback loop | Sense → compare → act → repeat | All life |
| Spike | Action potential | All animals with neurons |

The neuron is a specialized cell that scaled the electrochemical gradient into
discrete spikes. It didn't invent the mechanism — it inherited it.

---

## What This Means for This Project

The `PROJECT_POV.md` asks: what is the minimal algorithm for credit assignment?

E. coli gives a candidate answer:

> A fast signal, a slow decaying memory of the recent past, and a mechanism
> that takes their difference to drive action.

That's the algorithm. STDP eligibility traces are the neuron-level implementation
of the same idea. The open question this project is exploring — how to do deep
credit assignment locally — is asking whether you can chain this primitive
into something that handles structural depth, not just temporal delay.

---

## Literature Starting Points

- Sourjik & Wingreen (2012) — "Responding to chemical gradients: bacterial chemotaxis"
- Izhikevich (2007) — eligibility traces and reward-modulated STDP
- The parallel between chemotaxis adaptation and synaptic eligibility traces
  is not widely formalized — this may be original framing worth developing
