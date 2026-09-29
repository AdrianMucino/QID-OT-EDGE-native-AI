# QID-NPI: Edge-Native Cognitive Hypervisors

[![Paper](https://img.shields.io/badge/Paper-Zenodo/arXiv-blue)](#) 
[![License](https://img.shields.io/badge/License-MIT-green.svg)](#)
[![Status](https://img.shields.io/badge/Status-Active_Deployment-orange)](#)

> **Deterministic Context Injection in Zero-Trust SCADA Environments**
> 
> *By Adrian de Jesus Muciño Gomez, Alain Javier Alvarado Barroso, and Hubert Anna Jo Kamps* | **QID Industrial Automation**

This repository contains the supplementary documentation and Markdown manuscript for **QID-NPI (QID Neural Process Intelligence)**, a hybrid retrieval-augmented inference architecture built specifically for industrial Operational Technology (OT) environments. 

QID-NPI operates as an *edge-native cognitive hypervisor*, enforcing resource isolation between AI agents, scheduling context deterministically, and dispatching structured prompts with **zero cloud dependency**. 

## 🏭 The Industrial AI Trilemma
Deploying AI in modern manufacturing exposes a structural trilemma that standard cloud-centric frameworks fail to address:
1. **Data Sovereignty:** Strict IEC 62443 zero-trust networks prohibit live process data from hitting external cloud APIs.
2. **Semantic Data Quality:** Raw, dense tag streams cause severe numerical hallucinations and *Lost-in-the-Middle* context degradation in LLMs.
3. **Operational Accessibility:** Operators require immediate, real-time insights without relying on stable internet connectivity.

**QID-NPI resolves all three simultaneously.**

## 🚀 Key Performance Metrics
Validated against a 26-year legacy operational pedigree and deployed at a premium cheese manufacturing facility, the system demonstrates massive improvements over dense tabular data ingestion:

* **📉 77% Reduction** in LLM context token payload via Sparse JSON serialization.
* **⏱️ 75% Faster** Time to First Token (TTFT).
* **🔒 100% Air-Gapped**, with zero data egress.
* **📱 Failsafe QR Delivery** for off-network mobile inference.

## 🏗️ The ProcessTensor Stack Architecture
QID-NPI is structured across a coherent five-layer architecture:

* **Layer 0 — Edge Control Fabric:** PLC-native semantic event emission (ISA-88 compliant) at the controller level (Rockwell ControlLogix).
* **Layer 1 — Process Tensor Store:** SQL Server Historian acting as a multi-dimensional labeled batch event database.
* **Layer 2 — Context Assembly Engine:** Deterministic temporal attribution using a SQL *Gaps-and-Islands* pattern and a strict *Drop-Zero* rule to generate Sparse JSON.
* **Layer 3 — Multi-Agent Inference:** An air-gapped Mixture of Agents (MoA) cascade on Apple Silicon running **Qwen-2.5-Coder-32B**, **DeepSeek-R1-32B**, and **Llama-3.3-70B**.
* **Layer 4 — Persistence & Output:** SQL write-back and compressed QR code delivery for operator-level access.

## 🗺️ Roadmap (Phase 2)
We are actively expanding the architecture to include a localized **Vector Knowledge Layer**:
- **Vector-Driven RAG:** Utilizing `pgvector` and `nomic-embed-text` to map manufacturer specifications into a 768-dimensional semantic phase space for context-aware alarm diagnostics.
- **Golden Batch Matching:** Creating an "Infinite Edge Historian" to benchmark real-time vectors against decades of optimal production runs.
- **ANE-Accelerated Fine-Tuning:** Experimental transformer training leveraging the Apple Neural Engine (ANE) via Metal APIs for resource-constrained OT environments.

## 📖 Read the Paper
The full manuscript detailing the architecture, empirical evaluations, and alignment with the CISA/NSA 2025 AI OT guidelines is available in this repository:
* [📄 Read the Full Markdown Paper](paper.md)

## 🤝 Citation
If you use this architecture or reference our findings in your own research, please cite our preprint:
```bibtex
@misc{mucino2026qidnpi,
  title={QID-NPI: Edge-Native Cognitive Hypervisors for Deterministic Context Injection in Zero-Trust SCADA Environments},
  author={Muciño Gomez, Adrian de Jesus and Alvarado Barroso, Alain Javier and Kamps, Hubert Anna Jo},
  year={2026},
  publisher={QID Industrial Automation}
}
