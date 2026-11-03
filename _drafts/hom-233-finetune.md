---
layout: post
title: "I trained a model on four hundred of my own commits. Did it ship more tickets?"
---

<!-- Muse note: drafted under memory/jauzey-voice.md. Sentences marked [his call] state a belief I inferred;
     rewrite or cut them. Numbers are from memory/log/2026-11-03.md; the result numbers land 11-04. -->

## 1. The question (150 words)

Opening in his words, roughly: for six weeks an agent farm in my closet has been taking Linear tickets for
Elective, writing the code, running the tests, opening the PR. Nearly all of it runs on Claude. The local
model I own, a 30B Qwen on a Mac, scores 4 of 10 on the same tickets. The question I wanted answered: if I
show that model four hundred of my own commits, does it finish more tickets, or does it only learn to write
diffs that look like mine?

Why it matters, one paragraph [his call]: Elective's reason to exist is that a local reporter can follow a
race's money from a public URL. The farm is how one person keeps that shipping. Every ticket that a model I
own can take is a ticket that does not depend on a company's pricing page or its idea of acceptable use. The
political eye stays open: the hardware is mine, the weights are Alibaba's, the judge is still Claude.

## 2. What I had (120 words)

- Elective repo: 404 commits with a ticket key, per-file diffs, cap 8,000 tokens each, 1,711 pairs, 9.7M tokens.
- Ten tickets held out by key (so the model never saw their diffs) for the eval.
- The eval harness: full tier, ten tickets, a judge, a pinned baseline of 4/10 for the untuned model (Q4).
- A finding before training started: there are no merged Qwen farm runs to learn from. All 96 elective runs
  were Claude. So this is "learn the repo", not "learn from your own successes". Say that plainly.

## 3. What I did, in order (250 words)

- Download the 8-bit base (32 GB). MLX LoRA, rank 16, learning rate 1e-4 (ten times the full fine-tune rate,
  per Thinking Machines' "LoRA Without Regret"), all 48 layers, adapters on q/k/v/o only. The MoE expert
  layers were skipped for time; the same paper says that is where the capacity is. One epoch on a 6M-token
  subset (929 pairs), because the timing run said 11 hours and the real run said 16.
- The confound I almost shipped: the baseline is a 4-bit model and the tuned one would be 8-bit. Speed and
  quality both move. So an untuned 8-bit control gets the same eval. The control is the post.
- The wrong turns, with cost: the first dataset build did not name the file in the prompt; the dequantize
  flag; the ETA that slipped from 1 AM to 3:30 AM at 104 tokens per second on a laptop that was also asleep
  with the lid closed. [his call: keep the laptop detail; it is the point of the exercise]
- Val loss 1.48 to 0.94 in twenty steps, then the slow part.

## 4. The bar, set before the result (80 words)

Written down before the numbers: tuned above 6 of 10 with the 8-bit control still near 4 is signal. Tuned
and control both up means quantization, not training. Tuned up only on diff style (the judge's "looks like
the repo" notes) and not on pass count means "it learned my accent, not my job", and the cheaper answer is
AGENTS.md plus retrieval. Within 0.1 of the bar is a coin flip, and I will say so.

## 5. Result (blank until 11-04)

- Tuned, run 1: _/10. Tuned, run 2: _/10. 8-bit control: _/10. Baseline (Q4, pinned 10-26): 4/10.
- Per-ticket table: which flipped, which stayed.
- What the judge said about the diffs that passed and the ones that did not.
- Tokens, hours, dollars: training was free in dollars and 16 hours of a Mac; the eval cost N Claude judge calls.

## 6. What I make of it (150 words) [his call throughout]

Two drafts to choose from once the numbers exist:
- A. It moved. Then: the next run puts adapters on the experts and takes two epochs; the farm gets a
  `qwen3-coder-elective` provider for the small tickets; Claude keeps the ones that need judgment.
- B. It only moved style, or nothing. Then: the finding is that four hundred commits teach conventions, not
  competence, and competence for agentic work comes from on-policy data I do not have yet. The farm keeps
  logging; in three months there may be enough of my own successful local runs to try distillation.
Either way, one earned image at most, and the ending points forward: what the next run is, by date.

## Sources to link
- LoRA Without Regret, Thinking Machines (2025-09-29): https://thinkingmachines.ai/blog/lora/
- Research notes: notes/research/hom-233-tuning-concepts.md (40 sources); doc https://claude.ai/artifact/XQM39hy5PvMD7hrZzpTzAU
- Baseline findings doc: https://claude.ai/artifact/RWaJtVvdQkQ6HAgsVoCLn5
- Ticket HOM-233; dataset builder zotac/farm/tune/build_dataset.py on branch homelab/hom-233.
