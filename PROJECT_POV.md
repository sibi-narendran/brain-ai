# Project POV: Local Credit Assignment Without Backprop

This repo is built around one central question:

Can we build learning systems that do useful credit assignment without ordinary
backpropagation, without dense all-to-all communication, and without every unit
needing a global view of the network?

The motivating answer is: yes, at small and medium scales, through biologically
plausible local learning rules. The deeper frontier is making those rules scale
without losing the efficiency and locality that make them interesting in the
first place.

## Starting Point: Spikes, Locality, And Sparse Wiring

The base computational unit is a leaky integrate-and-fire neuron.

Each neuron has a membrane voltage `V` that integrates incoming weighted spikes
and leaks back toward rest between them:

```text
tau * dV/dt = -(V - V_rest) + I(t)
```

When `V` crosses a threshold, the neuron emits a spike and resets. Information
lives in the timing of spikes: binary events, silence between events, and
computation that is naturally event-driven.

That matters because the neuron only sees spikes arriving at its own synapses.
It does not need a global view of the network. The wiring should match that:
sparse, local, and far from fully connected. Most entries in the weight matrix
should be zero.

## Local Learning: STDP

The simplest local learning rule is spike-timing-dependent plasticity, or STDP.

A synapse changes according to the timing relationship between its presynaptic
and postsynaptic spikes:

- If the presynaptic neuron fires shortly before the postsynaptic neuron, the
  synapse is strengthened. The presynaptic spike plausibly helped cause the
  postsynaptic spike.
- If the presynaptic neuron fires shortly after the postsynaptic neuron, the
  synapse is weakened. It did not contribute causally to that spike.

In short:

```text
pre before post: Delta w = +A * exp(-Delta t / tau)
pre after post:  Delta w = -A * exp( Delta t / tau)
```

This is "fire together, wire together" with a causal arrow. The important part
is that the synapse needs only local information: its own pre-spike time and
post-spike time.

The limitation is also clear. STDP alone is unsupervised. It amplifies frequent
patterns, builds feature detectors, and self-organizes around statistical
regularities, but it does not know what the system is supposed to accomplish.
It can wire up the wrong behavior very efficiently.

## First Trick: Reward-Modulated STDP

To make local spiking learning task-directed, we add a third factor:
a global scalar reward.

The key mechanism is reward-modulated STDP with eligibility traces.

The learning update is split into two stages:

1. When local STDP detects a causal pre-to-post event, the synapse does not
   immediately change its weight. Instead, it lays down an eligibility trace:
   a temporary local tag saying, "this synapse recently participated in a
   potentially causal event."
2. Later, the environment emits a scalar reward. That scalar is broadcast
   globally, like a dopamine-like neuromodulatory signal. The actual update is:

```text
Delta w = eta * eligibility_trace * (R - baseline)
```

This is the elegant part:

- STDP decides which synapses are eligible for credit or blame.
- The global reward decides whether those eligible changes should be reinforced
  or suppressed.
- Each synapse needs only its own trace and the broadcast scalar.

There is no backward pass. There is no all-to-all feedback. There is no precise
per-weight gradient.

The eligibility trace solves the distal reward problem. Spikes happen now, but
reward may arrive later. The trace bridges that time gap: synapses that recently
participated remain locally tagged when reward arrives, so the scalar reward
modulates only the synapses that were causally active.

## Full R-STDP Learning Loop

The basic loop is:

1. Encode input as spike trains. For example, brighter pixels produce faster
   spikes.
2. Spikes propagate through the sparse network.
3. Neurons integrate incoming spikes and emit output spike patterns.
4. Output spikes define the network's action.
5. As spikes flow, participating synapses accumulate eligibility traces through
   local STDP.
6. The environment evaluates the action and emits reward `R`.
7. The reward is broadcast to all synapses.
8. Each synapse updates according to:

```text
Delta w = eta * trace * (R - baseline)
```

Over many trials, synapses that contribute to rewarded behavior strengthen, and
synapses that contribute to punished behavior weaken.

This is a complete backprop-free learning mechanism. It is local, sparse,
event-driven, online, and compatible with neuromorphic hardware.

## The Wall: One Scalar Is Not Enough For Deep Credit Assignment

The weakness is structural.

A global scalar reward carries very little information. It can say whether the
whole system did well or badly, but it cannot precisely tell thousands or
millions of deep synapses how each one contributed.

Backpropagation gives every weight a specific gradient. Reward-modulated STDP
does not. It performs a noisy form of policy-gradient-like search, guided by
local eligibility traces and a global scalar.

That means R-STDP is strong at:

- temporal credit assignment
- online adaptation
- shallow reinforcement learning tasks
- low-power neuromorphic computation
- biologically plausible learning

But it struggles with:

- structural credit assignment across deep networks
- high sample efficiency
- low-variance learning
- large-scale supervised accuracy
- ImageNet-scale capability

So the tradeoff is clear: backprop wins on scalable capability; local spiking
learning wins on energy, locality, and online adaptation.

## The Stronger Direction: Dendritic Credit Assignment

The next idea is to replace the single global scalar with richer local error
signals.

The key intuition:

What if the neuron could know which part of itself was surprised, and by how
much?

That points directly to dendritic credit assignment.

In these models, pyramidal neurons are not treated as simple point neurons.
They have compartments:

- basal dendrites receive feedforward input
- apical dendrites receive feedback or top-down input
- dendritic plateau potentials can influence bursting

The apical compartment becomes the place where mismatch, surprise, or error is
registered. Instead of a single scalar reward being broadcast everywhere, each
neuron can receive a more local, more specific feedback signal.

## Burst-Dependent Plasticity

Burst-dependent plasticity makes this idea operational.

The rough mechanism is:

1. Feedforward activity drives ordinary spiking.
2. Feedback arriving at apical dendrites induces dendritic plateau potentials.
3. Those plateau events cause high-frequency bursts.
4. The burst pattern acts as an instructive signal for plasticity.

In this view, a neuron multiplexes two things in its spike train:

- ordinary spikes communicate feedforward activity
- bursts communicate local error or surprise

This helps with a major problem in spiking systems: how to carry an instructive
error signal without ordinary signed gradients. The error is not sent as a
conventional backprop vector. It is expressed through the neuron's burst pattern.

That is the upgrade over scalar reward. The system moves from one global reward
number to many local, neuron-level credit signals.

## Why This Matters For This Repo

This repo exists to explore the path from simple local spiking learning toward
more credible credit assignment:

1. Start with leaky integrate-and-fire neurons.
2. Use sparse local connectivity.
3. Add STDP to get local causal plasticity.
4. Add eligibility traces and scalar reward to get reward-modulated STDP.
5. Acknowledge the scalar reward limitation directly.
6. Move toward dendritic or burst-based learning rules that carry richer local
   error signals.

The point is not to pretend this already beats backprop at scale. It does not.

The point is to study a different axis:

- lower energy
- local computation
- online adaptation
- sparse communication
- neuromorphic compatibility
- biologically plausible credit assignment

Backprop is still the dominant answer for large-scale capability. But it depends
on dense gradient transport, synchronized training phases, and hardware that is
very different from biological or neuromorphic computation.

This repo is about the alternative path.

## Honest Current Verdict

Reward-modulated STDP is buildable today. It is real, elegant, and useful for
small control tasks, bandits, toy reinforcement learning, and neuromorphic
experiments.

Its failure mode is also real: a scalar reward cannot carry enough information
for precise deep credit assignment.

Dendritic and burst-dependent approaches are the stronger frontier because they
replace the one global scalar with dense local feedback signals. They can
approximate backprop-like updates in small-to-medium settings, including work
around MNIST and CIFAR-scale tasks.

But they are not a finished replacement for backprop at large scale. The hard
part moves into the feedback pathway:

- the top-down signal must be correct
- feedback weights must be learned or aligned
- inhibition and compartment dynamics must shape useful error signals
- the system must remain stable while learning online

So the core position is:

Local surprise is necessary, but not sufficient. The frontier is making the
local surprise signal correct, stable, and scalable.

## Minimal Build Path

A practical toy implementation should start small:

1. Implement a leaky integrate-and-fire network.
2. Use sparse connectivity.
3. Rate-encode simple inputs as spike trains.
4. Start with a two-action bandit or tiny gridworld.
5. Implement STDP eligibility traces.
6. Broadcast scalar rewards.
7. Train with reward-modulated STDP.
8. Measure whether rewarded actions become more likely.
9. Then experiment with richer local error signals, such as burst-coded feedback.

Useful tools for this kind of prototype include:

- Brian2
- Nengo
- snnTorch

The first goal should not be ImageNet or a large benchmark. The first goal should
be watching a sparse spiking network learn from local timing plus one global
number, then using that failure mode to motivate dendritic credit assignment.

## Literature Trail

The relevant research direction includes:

- Izhikevich on reward-modulated STDP and eligibility traces
- Sacramento et al. 2018 on dendritic microcircuits approximating backprop
- Payeur et al. 2021 on burst-dependent plasticity
- BurstCCN
- Generalized Latent Equilibrium

This is the conceptual center of the repo:

Spikes give event-driven computation. Sparse wiring gives locality. STDP gives
causal local plasticity. Eligibility traces let delayed reward assign temporal
credit. Dendritic bursts point toward richer local error signals that may
approximate backprop without becoming ordinary backprop.

The open problem is not whether the idea is meaningful. It is how to make it
scale.
