# Decoy

<img width="2100" height="782" alt="image" src="https://github.com/user-attachments/assets/28c46390-7340-4d69-9616-259e0cfc7b8f" />

[Decoy] and PhishGuard 360 were my attempt at building a NLP based security agent (Reverse Social Engineering) that behaves well in an actual Gmail workflow and Vapi workflow, not just in isolated not[...] 

The first layer is deterministic and fast. It checks for known phishing patterns, sender and domain risk signals, and high-risk linguistic combinations. You can view it as a weighted rule score over b[...] 

$$s_1(e)=\sum_{i=1}^{n} w_i \cdot \mathbf{1}[\text{pattern}_i(e)].$$

If $s_1(e)\ge \tau_1$, the email is immediately flagged as a threat and no further model inference is needed. This layer is intentionally optimized for low latency and high recall on known attack temp[...] 

If an email passes Layer 1, it moves to a DistilBERT classifier that estimates phishing likelihood from content. Formally:
$$
p_{\text{threat}}(e)=\sigma(f_\theta(e)),
$$
where $f_\theta(e)$ is the model logit and $\sigma$ is the sigmoid function. In practice, this stage serves as the statistical middle ground: it clears high-confidence benign emails and escalates bord[...] 

The final layer uses Gemini with retrieval-augmented context for harder social-engineering cases. At this point the system is no longer just classifying text in isolation; it is reasoning with email c[...] 

$$\hat{y}=\arg\max_{y\in\{\text{safe},\text{threat}\}} P(y\mid e, C_k),$$

where $C_k$ is the retrieved context set. This is the expensive stage, so it is intentionally conditional. The whole architecture follows a conditional-compute objective: keep average latency low whil[...] 

$$\mathbb{E}[T]=T_1+(1-r_1)T_2+(1-r_1)(1-r_2)T_3,$$

with $T_i$ as per-layer latency and $r_i$ as stop rates from earlier layers.

Implementation-wise, this repository includes a Chrome extension (Manifest V3) for Gmail-side extraction and scan triggering, a Flask backend that orchestrates multi-layer routing, and a containerized[...] 

The development dataset used here is `hackathon-resources/se_phishing_test_set.csv` (1000+ labeled emails). Current smoke tests confirm end-to-end operation across health checks, benign classification[...] 

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

- Devlin et al. (2019), *BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding*  
  https://arxiv.org/abs/1810.04805

- Sanh et al. (2019), *DistilBERT, a distilled version of BERT*  
  https://arxiv.org/abs/1910.01108

- Wang et al. (2020), *MiniLM: Deep Self-Attention Distillation for Task-Agnostic Compression of Pre-Trained Transformers*  
  https://arxiv.org/abs/2002.10957

- Lewis et al. (2020), *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*  
  https://arxiv.org/abs/2005.11401

- Wei et al. (2022), *Chain-of-Thought Prompting Elicits Reasoning in Large Language Models*  
  https://arxiv.org/abs/2201.11903

- Kojima et al. (2022), *Large Language Models are Zero-Shot Reasoners*  
  https://arxiv.org/abs/2205.11916

- Yao et al. (2023), *ReAct: Synergizing Reasoning and Acting in Language Models*  
  https://arxiv.org/abs/2210.03629

- Greshake et al. (2023), *Not What You’ve Signed Up For: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection*  
  https://arxiv.org/abs/2302.12173

- Park et al. (2023), *Generative Agents: Interactive Simulacra of Human Behavior*  
  https://arxiv.org/abs/2304.03442

- PhishGuard 360: cyber-agent workflow and project demo  
  https://youtu.be/l2kjR7IGRBA
