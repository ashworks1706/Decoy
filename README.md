# Decoy

<img width="2100" height="782" alt="image" src="https://github.com/user-attachments/assets/28c46390-7340-4d69-9616-259e0cfc7b8f" />

[Decoy] and PhishGuard 360 were my attempt at building a NLP based security agent (Reverse Social Engineering) that behaves well in an actual Gmail workflow and Vapi workflow, not just in isolated notebook tests. The main idea is simple: phishing emails vary a lot in complexity, so a single detector is usually either too shallow or too expensive. I wanted a system where obvious attacks get handled quickly, and only uncertain emails get routed to heavier analysis. That led to a three-layer pipeline with increasing depth and compute cost.

The first layer is deterministic and fast. It checks for known phishing patterns, sender and domain risk signals, and high-risk linguistic combinations. You can view it as a weighted rule score over binary indicators:
$$
s_1(e)=\sum_{i=1}^{n} w_i \cdot \mathbf{1}[\text{pattern}_i(e)].
$$
If $s_1(e)\ge \tau_1$, the email is immediately flagged as a threat and no further model inference is needed. This layer is intentionally optimized for low latency and high recall on known attack templates.

If an email passes Layer 1, it moves to a DistilBERT classifier that estimates phishing likelihood from content. Formally:
$$
p_{\text{threat}}(e)=\sigma(f_\theta(e)),
$$
where $f_\theta(e)$ is the model logit and $\sigma$ is the sigmoid function. In practice, this stage serves as the statistical middle ground: it clears high-confidence benign emails and escalates borderline cases. I found this layer useful for catching patterns that are too varied for static rules but still don’t require full contextual reasoning.

The final layer uses Gemini with retrieval-augmented context for harder social-engineering cases. At this point the system is no longer just classifying text in isolation; it is reasoning with email content plus retrieved user/context memory:
$$
\hat{y}=\arg\max_{y\in\{\text{safe},\text{threat}\}} P(y\mid e, C_k),
$$
where $C_k$ is the retrieved context set. This is the expensive stage, so it is intentionally conditional. The whole architecture follows a conditional-compute objective: keep average latency low while preserving deeper analysis capacity for difficult samples. A simple expectation form is:
$$
\mathbb{E}[T]=T_1+(1-r_1)T_2+(1-r_1)(1-r_2)T_3,
$$
with $T_i$ as per-layer latency and $r_i$ as stop rates from earlier layers.

Implementation-wise, this repository includes a Chrome extension (Manifest V3) for Gmail-side extraction and scan triggering, a Flask backend that orchestrates multi-layer routing, and a containerized runtime with proxy, caching, and monitoring components. There is also retrieval-backed user context handling and model-training utilities for iterating on layer-2 behavior. The intent is to keep this as a practical research-style system artifact: inspectable, testable, and easy to extend.

The development dataset used here is `hackathon-resources/se_phishing_test_set.csv` (1000+ labeled emails). Current smoke tests confirm end-to-end operation across health checks, benign classification, threat classification, and user-context retrieval. I treat those as systems-validation signals, not final benchmark claims. Proper ablation and calibrated metric reporting across attack subtypes is still ongoing work.

If you want to run it quickly:
```bash
git clone https://github.com/ashworks1706/Cybersec-360-hackathon.git
cd Cybersec-360-hackathon
cp .env.template .env
./deploy.sh
```

## References

- Vaswani et al. (2017), *Attention Is All You Need*  
  https://arxiv.org/abs/1706.03762

- Sanh et al. (2019), *DistilBERT, a distilled version of BERT*  
  https://arxiv.org/abs/1910.01108

- Lewis et al. (2020), *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*  
  https://arxiv.org/abs/2005.11401

- Wei et al. (2022), *Chain-of-Thought Prompting Elicits Reasoning in Large Language Models*  
  https://arxiv.org/abs/2201.11903# PhishGuard 360

<img width="2100" height="782" alt="image" src="https://github.com/user-attachments/assets/28c46390-7340-4d69-9616-259e0cfc7b8f" />

PhishGuard 360 is my attempt at building a phishing detector that behaves well in an actual Gmail workflow, not just in isolated notebook tests. The main idea is simple: phishing emails vary a lot in complexity, so a single detector is usually either too shallow or too expensive. I wanted a system where obvious attacks get handled quickly, and only uncertain emails get routed to heavier analysis. That led to a three-layer pipeline with increasing depth and compute cost.

The first layer is deterministic and fast. It checks for known phishing patterns, sender and domain risk signals, and high-risk linguistic combinations. You can view it as a weighted rule score over binary indicators:
$$
s_1(e)=\sum_{i=1}^{n} w_i \cdot \mathbf{1}[\text{pattern}_i(e)].
$$
If $s_1(e)\ge \tau_1$, the email is immediately flagged as a threat and no further model inference is needed. This layer is intentionally optimized for low latency and high recall on known attack templates.

If an email passes Layer 1, it moves to a DistilBERT classifier that estimates phishing likelihood from content. Formally:
$$
p_{\text{threat}}(e)=\sigma(f_\theta(e)),
$$
where $f_\theta(e)$ is the model logit and $\sigma$ is the sigmoid function. In practice, this stage serves as the statistical middle ground: it clears high-confidence benign emails and escalates borderline cases. I found this layer useful for catching patterns that are too varied for static rules but still don’t require full contextual reasoning.

The final layer uses Gemini with retrieval-augmented context for harder social-engineering cases. At this point the system is no longer just classifying text in isolation; it is reasoning with email content plus retrieved user/context memory:
$$
\hat{y}=\arg\max_{y\in\{\text{safe},\text{threat}\}} P(y\mid e, C_k),
$$
where $C_k$ is the retrieved context set. This is the expensive stage, so it is intentionally conditional. The whole architecture follows a conditional-compute objective: keep average latency low while preserving deeper analysis capacity for difficult samples. A simple expectation form is:
$$
\mathbb{E}[T]=T_1+(1-r_1)T_2+(1-r_1)(1-r_2)T_3,
$$
with $T_i$ as per-layer latency and $r_i$ as stop rates from earlier layers.

Implementation-wise, this repository includes a Chrome extension (Manifest V3) for Gmail-side extraction and scan triggering, a Flask backend that orchestrates multi-layer routing, and a containerized runtime with proxy, caching, and monitoring components. There is also retrieval-backed user context handling and model-training utilities for iterating on layer-2 behavior. The intent is to keep this as a practical research-style system artifact: inspectable, testable, and easy to extend.

The development dataset used here is `hackathon-resources/se_phishing_test_set.csv` (1000+ labeled emails). Current smoke tests confirm end-to-end operation across health checks, benign classification, threat classification, and user-context retrieval. I treat those as systems-validation signals, not final benchmark claims. Proper ablation and calibrated metric reporting across attack subtypes is still ongoing work.

If you want to run it quickly:
```bash
git clone https://github.com/ashworks1706/Cybersec-360-hackathon.git
cd Cybersec-360-hackathon
cp .env.template .env
./deploy.sh
```

## References

- Vaswani et al. (2017), *Attention Is All You Need*  
  https://arxiv.org/abs/1706.03762

- Sanh et al. (2019), *DistilBERT, a distilled version of BERT*  
  https://arxiv.org/abs/1910.01108

- Lewis et al. (2020), *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*  
  https://arxiv.org/abs/2005.11401

- Wei et al. (2022), *Chain-of-Thought Prompting Elicits Reasoning in Large Language Models*  
  https://arxiv.org/abs/2201.11903
