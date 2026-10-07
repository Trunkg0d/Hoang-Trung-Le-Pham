---
layout: posts
title: "Unlearning Inversion Attack"
date: 2025-04-16
description: "A unified guide to machine unlearning, inversion attacks, the attack surface created by forgetting, and how to evaluate privacy beyond accuracy."
reading_time: "18 min read"
writing_order: 1
status: published
content_origin: slop
content_origin_label: "Slop"
content_origin_note: "AI-drafted and translated"
categories: research security
permalink: /writing/unlearning-inversion-attack/
---

Machine unlearning is often described as an eraser for machine-learning models. When a user asks for their data to be deleted, removing the corresponding record from a database is not enough; the influence of that record must also be removed from the trained model. This is an important privacy requirement, but it raises a harder question:

> Does a model that has “forgotten” a data point truly stop revealing information about it?

An Unlearning Inversion Attack exploits the change between a model before and after unlearning. Instead of assuming that the post-unlearning model is automatically safer, an attacker treats the **transition between the two model states** as a new source of information.

This article brings together three closely related topics—machine unlearning, inversion attacks, and unlearning inversion attacks—into one coherent picture.

## Machine unlearning: how does a model forget? {#machine-unlearning}

### The basic problem

Assume that:

- $D$ is the original training dataset;
- $D_{forget}$ is the subset that must be removed;
- $D_{retain} = D \setminus D_{forget}$ is the data that remains;
- $A$ is the training algorithm;
- $M = A(D)$ is the original model.

The goal of an unlearning algorithm $U$ is to produce a model $M_u$ in which the influence of $D_{forget}$ has been removed while performance on $D_{retain}$ is preserved:

$$
M_u = U(M, D, D_{forget})
$$

The strongest natural reference is a model retrained from scratch on the retained data:

$$
M_{retrain} = A(D_{retain})
$$

If the output distribution of $M_u$ is close to that of $M_{retrain}$, the unlearning procedure is moving toward the intended behavior. However, being close in predictive accuracy does not necessarily mean being safe against inference attacks.

<figure>
  <img src="{{ '/images/machine-unlearning/naive_retraining_and_machine_unlearning.png' | relative_url }}" alt="Comparison between full retraining and machine unlearning">
  <figcaption>Retraining on the retained dataset is the natural reference, but it is usually more expensive than unlearning.</figcaption>
</figure>

### Exact and approximate unlearning

**Exact unlearning** aims to produce a model whose distribution is equivalent to that of a model retrained from scratch on $D_{retain}$. This is a strict objective and often requires substantial computation or storage.

**Approximate unlearning** accepts a bounded error in exchange for faster, more practical updates. Its objective can be expressed using a distributional distance:

$$
d(P_{M_u}, P_{M_{retrain}}) \leq \epsilon
$$

In practice, methods such as fine-tuning on the retained set, influence-function updates, and SISA typically fall into the approximate category. SISA divides the training data into independent shards, trains separate submodels, and retrains only the shard affected by a deletion request.

<figure>
  <img src="{{ '/images/machine-unlearning/sisa.png' | relative_url }}" alt="SISA architecture for machine unlearning">
  <figcaption>SISA limits the amount of retraining by partitioning data into independent shards.</figcaption>
</figure>

### Accuracy alone cannot evaluate unlearning

A meaningful evaluation should examine at least four dimensions:

1. **Removal quality:** does behavior on $D_{forget}$ resemble that of $M_{retrain}$?
2. **Retained utility:** is performance on $D_{retain}$ and the test set preserved?
3. **Efficiency:** how much time, memory, and computation does each deletion require?
4. **Privacy leakage:** what can still be extracted through membership inference, model inversion, or gradient inversion?

The fourth dimension is where inversion attacks become an important evaluation tool—and where a new attack surface emerges.

## Inversion attacks: reading a model backward {#inversion-attacks}

Inversion attacks use information exposed by a model to infer properties of its training data. Two common families are:

- **Model inversion attacks:** use predictions, confidence scores, logits, or model parameters to infer private attributes or representative training samples.
- **Gradient inversion attacks:** use gradients or parameter updates to reconstruct the inputs and labels that produced them.

Consider a simple linear model:

$$
f(x) = w_1x_1 + w_2x_2 + \dots + w_tx_t + b.
$$

If an attacker knows the output $y$, the model parameters, and every feature except $x_1$, the private feature can be recovered directly:

$$
\hat{x}_1 = \frac{y - (w_2x_2 + \dots + w_tx_t + b)}{w_1}.
$$

The inversion process is less direct in a deep neural network, but the intuition remains the same: optimize a synthetic input until its behavior or gradient matches the observed signal.

<figure>
  <img src="{{ '/images/machine-unlearning/mu-vs-inversion-attacks.png' | relative_url }}" alt="Relationship between machine unlearning and inversion attacks">
  <figcaption>Inversion attacks are both a privacy threat and a way to test whether unlearning has actually removed sensitive information.</figcaption>
</figure>

### Three important connections

Machine unlearning and inversion attacks intersect in three ways:

1. Inversion attacks can expose information that unlearning is expected to remove.
2. Resistance to inversion can serve as one signal of unlearning quality.
3. Some unlearning techniques can also mitigate gradient leakage in distributed systems.

The paradox is that deletion may reduce one form of leakage while the difference between two model versions creates a new signal for an attacker.

## Threat model for an Unlearning Inversion Attack {#threat-model}

Consider a Machine Learning as a Service system. A provider trains model $M$ on dataset $D$, receives a request to delete $D_{forget}$, and then deploys the unlearned model $M_u$.

An attacker may operate under one of two access models:

- **White-box attacker:** has access to the parameters of both $M$ and $M_u$, for example as a server operator or insider.
- **Black-box attacker:** can only query the two models through an API and observe confidence scores or logits.

The central vulnerability may not exist in either $M$ or $M_u$ alone. It exists in the **difference** between them. If that change was primarily introduced to remove $D_{forget}$, it may isolate the very signal that the system was supposed to erase.

### White-box attack: feature reconstruction

Under gradient-descent training, a parameter update can be written as:

$$
\theta_{k+1} = \theta_k - \alpha \nabla_\theta \mathcal{L}(\theta_k).
$$

A white-box attacker can use the parameter difference between the original and unlearned models as an approximation of the gradient signal associated with the deleted data:

$$
\nabla^\ast \approx \theta - \theta_u.
$$

The attacker then initializes a synthetic input $x'$ and optimizes it so that the resulting gradient $\nabla'$ aligns with $\nabla^\ast$:

$$
\arg\min_{x'} -\operatorname{cos}(\nabla', \nabla^\ast) + \lambda TV(x').
$$

Here, $TV(x')$ is a Total Variation regularizer that encourages smoother, more natural reconstructions. The optimized result may recover recognizable structure or attributes from a deleted sample even when the original sample is no longer directly available.

> Security implication: retaining both pre- and post-unlearning checkpoints may reveal more information than exposing either checkpoint on its own.

### Black-box attack: inferring a forgotten label

A black-box attacker does not need access to model parameters. Instead, the attacker creates **probing samples** for target classes and compares the responses of the two models.

A general procedure is:

1. Generate samples $x'$ that the original model $M$ classifies as class $y_t$ with high confidence.
2. Record the prediction vectors returned by $M$.
3. Submit the same samples to $M_u$.
4. Measure the change in confidence for each class.

For $m$ probing samples from one class, the average change can be written as:

$$
\Delta_p = \frac{1}{m}\sum_{i=1}^{m}\left(\hat{\mathcal{Y}}_i - \mathcal{Y}_i\right).
$$

If confidence in one class drops substantially after unlearning, that class becomes a candidate for the label of the forgotten data. To infer multiple classes, an attacker can rank them by confidence reduction and select the Top-$K$.

The attacker does not need to reconstruct the original input. A label alone—such as a diagnosis, location, demographic category, or behavior type—may already be sensitive.

## Why “forgotten” is not enough

Unlearning is usually optimized to move a model toward a desired final state. Security, however, depends not only on that final state but also on the information revealed by the transition.

Three factors make this distinction important:

- **Difference amplification:** subtracting two models can cancel shared behavior and highlight changes associated with the deleted data.
- **Overexposed outputs:** full logits or detailed confidence scores reveal more information than a top-1 label.
- **Weak evaluation:** lower accuracy on a forget set does not prove that identifying attributes or labels have disappeared.

A privacy claim for an unlearning method should therefore be tested under an explicit threat model rather than inferred from a performance metric.

## Evaluation and defense {#defense}

No single countermeasure covers the entire attack surface. A robust design should combine several layers.

### 1. Evaluate against retraining

Compare $M_u$ with $M_{retrain}$ on both utility and leakage. A comparison with the original model alone cannot tell us which changes reflect correct forgetting and which expose an abnormal trace.

### 2. Limit paired model access

- Do not give the same user long-term access to both pre- and post-unlearning versions.
- Avoid retaining or distributing unnecessary historical checkpoints.
- Separate operational, audit, and research privileges.

### 3. Reduce API detail

Return only the information required for the task. Top-1 labels, calibrated or quantized confidence, rate limits, and anomalous-query detection can make probing harder. These are mitigations, not proof that unlearning is safe.

### 4. Protect the update itself

Clipping, controlled noise, regularization, differential privacy, or aggregated updates may make it harder to isolate the signal of a single record. Each mechanism introduces trade-offs among privacy, utility, and cost.

### 5. Use attack-based evaluation

At minimum, an evaluation suite should include:

- Membership Inference Attacks;
- Model or Gradient Inversion Attacks;
- forgotten-label inference;
- both white-box and black-box access;
- multiple sizes and structures of $D_{forget}$.

If a method succeeds only against a weak attacker, the defensible conclusion is “safe under the tested conditions,” not “all information has been deleted.”

## A practical deployment checklist

Before making a privacy claim about an unlearning pipeline, ask:

1. Who can access the model before and after unlearning?
2. Does the API return a label, confidence values, or full logits?
3. How closely does the unlearned model match the retraining baseline?
4. Which attack-based metrics have been evaluated on the forget set?
5. Are audit logs, rate limits, and checkpoint-retention policies in place?

Machine unlearning is not only an optimization problem; it is a systems problem. The algorithm, API, access policy, version storage, and evaluation process jointly determine whether data has actually been forgotten.

## Conclusion

Unlearning Inversion Attacks reveal an important distinction: **removing a data point's influence from a model and preventing the model from revealing information about that data point are related but different goals**.

A model can preserve accuracy, remain close to the retraining baseline, and still expose information about deleted data when an attacker compares two versions. Unlearning evaluation should therefore move beyond the question “does the model still predict correctly on the forget set?” and ask the stronger question: “given realistic access, what can an attacker still infer?”

---

## References

- Hu, H., Wang, S., Dong, T., & Xue, M. (2024). [Learn What You Want to Unlearn: Unlearning Inversion Attacks against Machine Unlearning](https://arxiv.org/abs/2404.03233).
- Xu, J., Wu, Z., Wang, C., & Jia, X. (2024). [Machine Unlearning: Solutions and Challenges](https://doi.org/10.1109/TETCI.2024.3379240). *IEEE Transactions on Emerging Topics in Computational Intelligence*, 8(3), 2150–2168.
- Wang, W., Tian, Z., Zhang, C., & Yu, S. (2024). [Machine Unlearning: A Comprehensive Survey](https://arxiv.org/abs/2405.07406).
- Graves, L., Nagisetty, V., & Ganesh, V. (2020). [Amnesiac Machine Learning](https://arxiv.org/abs/2010.10981).
- [NeurIPS 2023 Machine Unlearning Challenge — Starting Kit](https://unlearning-challenge.github.io/).
