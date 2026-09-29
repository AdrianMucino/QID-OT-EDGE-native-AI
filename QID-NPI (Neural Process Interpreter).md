
# QID-NPI: Edge-Native Cognitive Hypervisors for Deterministic Context Injection in Zero-Trust SCADA Environments
**Authors:** Adrian de Jesus Muciño Gomez$^{1}$ · Alain Javier Alvarado Barroso$^{1}$ · Hubert Anna Jo Kamps$^{1}$  
$^{1}$ QID Industrial Automation, Cuautitlán Izcalli, Estado de México, México  
**Correspondence:** QID Industrial Automation · qid-mex.com  
**Submitted to:** arXiv cs.SY (Systems and Control) — secondary: cs.AI  
---
## Abstract
Integrating Large Language Models (LLMs) into Operational Technology (OT) environments presents two fundamental challenges: strict data sovereignty requirements imposed by zero-trust industrial networks, and systematic degradation of LLM reasoning quality when processing raw continuous process data. We present **QID-NPI** (*QID Neural Process Intelligence*), a hybrid retrieval-augmented inference architecture we term an *edge-native cognitive hypervisor* — a system that enforces resource isolation between agents, schedules context deterministically, and dispatches structured prompts without any cloud dependency — deployed at a premium Aguascalientes Rincón de Romos cheese manufacturing facility. While the current production cadence averages 3 continuous batches per day, the system has been rigorously validated against the facility's relational SQL historian containing 26 years of legacy OT data, demonstrating scalability across decades of dense industrial telemetry.

//The system's robust preprocessing has been rigorously validated against a high-density 2-year rolling window of OT data (constrained by edge SQL Express licensing), though the underlying semantic control library boasts a 26-year operational pedigree//

QID-NPI addresses both challenges through a deterministic context preprocessing layer and a multi-agent inference architecture deploying specialized quantized foundation models (including Llama-3.3-70B and DeepSeek-R1-32B) at 4-bit precision on edge hardware. Operating entirely within the plant network boundary, the system generates differentiated outputs per production batch with *zero* data egress. A secondary air-gapped delivery mechanism encodes sanitized prompts into QR codes for local inference on operator mobile devices, addressing last-mile access without network dependency. Transitioning from dense tabular data to Sparse JSON serialization reduces context payload by up to 77%, significantly reducing Time to First Token (TTFT) while eliminating the *Lost-in-the-Middle* attention degradation. We further document the alignment of this architecture with the December 2025 CISA/NSA joint guidance on secure AI integration in OT environments.
---
## 1. Introduction
### 1.1 The Industrial AI Trilemma
The deployment of artificial intelligence in manufacturing Operational Technology (OT) environments exposes a structural trilemma that cloud-centric AI frameworks do not address: the simultaneous requirements of *data sovereignty*, *semantic data quality*, and *real-time operational accessibility*.
**Data sovereignty.** Modern Tier 1 manufacturing facilities operate under zero-trust OT network architectures mandated by IEC 62443. These architectures strictly prohibit the transmission of live process data to external cloud APIs. The December 2025 joint guidance published by CISA, NSA, and FBI explicitly recommends that AI systems in OT environments maintain human oversight and degrade gracefully without cloud connectivity [CISA2025].
**Semantic data quality.** LLMs are semantic reasoning engines. Raw industrial process data does not constitute semantic input. The academic consensus confirms that direct injection of raw tag streams produces unreliable outputs, including numerical hallucination [LLMTS2023]. The transformation from raw process values to semantically structured event records is a prerequisite for reliable LLM-based reasoning.
**Operational accessibility.** Industrial operators work in network-segmented plant floors requiring immediate access to structured information without relying on stable internet connectivity.
As elaborated in Section 2, QID-NPI resolves all three vertices of this trilemma simultaneously: data sovereignty is enforced by edge-only inference with zero egress (Layer 3); semantic quality is guaranteed by the deterministic preprocessing pipeline (Layer 2); and operational accessibility is achieved through the air-gapped QR delivery mechanism (Layer 4).
### 1.2 Existing Approaches and Their Limitations
Current industrial AI solutions address subsets of this trilemma but not all three constraints simultaneously.
**Rockwell Automation FactoryTalk DataReady & Copilot.** DataReady provides contextualization by mapping raw time-series data to unified models. However, it is fundamentally designed for IT/cloud egress — pushing payloads to central repositories (e.g., Azure) for external analytics. This introduces network latency, continuous API costs, and an expanded cybersecurity attack surface. Furthermore, FT Copilot utilizes SLMs (≤10B parameters), which lack the parametric capacity to deduce complex biochemical causality. QID-NPI diverges by feeding contextualized tensors directly into an air-gapped MoA on Apple Silicon.
**European OEM historian platforms** (e.g., GEA Codex, Proleit) provide SQL-based batch data management. Neither platform implements PLC-native semantic labeling nor integrates local LLM inference.
**Siemens Industrial Copilot** targets automation engineers performing PLC code generation. It relies on Microsoft Azure OpenAI, requiring cloud connectivity, and does not address operator-level shift documentation or multi-stream batch diagnostics [SIEMENSCOP2024].
**Unified Namespace (UNS) approaches** describe the goal of delivering full context to AI agents [HIVEMQ2025]. QID-NPI delivers this capability as a deployed production system, implemented through PLC-native semantic event emission.
### 1.3 Contributions
This paper presents QID-NPI and makes the following consolidated contributions:
1. **PLC-native semantic event emission.** Semantic labeling of process events occurs at the controller level (ISA-88 compliant), ensuring that every batch record carries structured meaning before any software layer is involved. This eliminates the ambiguity inherent in post-hoc tag renaming.
2. **Three-stream context assembly.** Deterministic fusion of process steps, critical equipment alarms, and operator interventions into a single context tensor enables causal attribution of process deviations without relying on LLM inference for temporal reasoning.
3. **Deterministic context preprocessing layer.** Sparse JSON serialization with a Drop-Zero rule and SQL Gaps-and-Islands temporal attribution reduces token payload by up to 77% and eliminates the *Lost-in-the-Middle* degradation, making the LLM's task strictly interpretive rather than reconstructive.
4. **Edge-native Mixture of Agents (MoA) inference.** A three-agent cascade — Extractor, Process Scientist, Executive — deployed on isolated Apple Silicon hardware with deterministic lifecycle management, producing operator-facing diagnostics with no cloud dependency.
5. **Air-gapped QR prompt delivery.** A failsafe last-mile access mechanism encoding sanitized context into QR codes for on-device mobile inference, decoupling operator access from SCADA network availability.
---
## 2. System Architecture
### 2.1 Overview
QID-NPI is structured as a five-layer architecture we term the **ProcessTensor Stack**. The term *cognitive hypervisor* is used deliberately: just as a compute hypervisor enforces resource isolation and schedules workloads across virtual machines, QID-NPI enforces context isolation between agents, schedules prompt dispatch deterministically via the Context Assembly Engine, and manages model lifecycle (memory allocation, cache flushing) to guarantee reproducible inference behavior. The five layers are:
- **Layer 0: Edge Control Fabric** (Rockwell ControlLogix)
- **Layer 1: Process Tensor Store** (SQL Server Historian)
- **Layer 2: Context Assembly Engine** (Ignition Perspective / FactoryTalk Optix)
- **Layer 3: Multi-Agent Inference** (Ollama + quantized LLMs)
- **Layer 4: Persistence & Output** (SQL write-back + QR delivery)
Together, these layers map directly to the trilemma introduced in §1.1: Layer 3 enforces *sovereignty*, Layer 2 guarantees *semantic quality*, and Layer 4 ensures *accessibility*.
### 2.2 Layer 0 — Edge Control Fabric
The control layer comprises four Rockwell ControlLogix controllers, one per horizontal agitator curd processing vessel.
#### 2.2.1 PLC-Native Semantic Event Emission (ISA-88 Compliance)
The critical architectural property is that semantic labeling of process events occurs at the controller level. The `PROJ_VATRecipeStepSetpoints01DType` data type encodes each ISA-88 recipe phase as a structured semantic object. When a batch step executes, the controller emits a report record containing the step's semantic fields alongside measured process values. This distinguishes QID-NPI from post-hoc contextualization approaches: the semantic ground truth is established at the source, not inferred downstream.
### 2.3 Layer 1 — Process Tensor Store
The `ProcessDataType1` table in SQL Server constitutes the Process Tensor Store: a multi-dimensional labeled batch event database. As of March 2026, the reference installation's Process Tensor Store incorporates structures validated against 26 years of accumulated legacy records, confirming that the preprocessing schema is stable across heterogeneous historical data densities.
### 2.4 Layer 2 — Context Assembly Engine
The Context Assembly Engine acts as the edge gateway. For this reference deployment, it is implemented via **Ignition Perspective 8.3** (Jython scripting), providing robust native SQL bridging. FactoryTalk Optix remains a highly capable alternative deployment target for future iterations.
#### 2.4.1 Three-Stream Context Assembly
Stream 1 (process steps) is retrieved via a named query. Streams 2 and 3 are retrieved through alarm journal queries spanning the same time window, classified into critical equipment alarms, analog PV warnings, and operator interventions. The three streams are merged into a single labeled context object before any LLM call is issued.
#### 2.4.2 Matrix Structure Comparison: Dense Tabular vs. Sparse JSON
Providing an LLM with excessive, unstructured data leads directly to context window saturation [BOUCHARD2024, LOSTMIDDLE]. To address this, QID-NPI abandons flat tabular SQL structures in favor of a nested **Sparse JSON** architecture utilizing a strict *Drop-Zero* serialization rule: any field carrying a zero, null, or default value is omitted from the serialized payload. As demonstrated in tabular data integration studies [JAITLY2024], this format acts as an "attention flare," directing the model's focus exclusively to anomalous keys and non-nominal values. The empirical impact of this decision is quantified in Section 3.2.
#### 2.4.3 Temporal Locality via Gaps-and-Islands SQL Pattern
To achieve strict Temporal Locality, QID-NPI attributes SCADA alarms and operator pauses deterministically using an advanced SQL *Gaps-and-Islands* pattern. The SQL performs the temporal attribution before the context reaches the LLM; the model simply reads a pre-attributed, causally labeled record. This design choice is architecturally significant: it confines non-determinism exclusively to the LLM's interpretive reasoning, not to its event-ordering logic.
### 2.5 Layer 3 — Multi-Agent Inference
#### 2.5.1 Inference Hardware and OS Bottleneck Prevention
To process Sparse JSON tensors without exceeding the 96 GB unified memory ceiling of the Mac Studio M3 Ultra, the orchestrator employs strict lifecycle management. Context caches are systematically flushed between inferences (`keep_alive: 0`), eliminating OS-level memory swapping and ensuring deterministic execution times across successive batch inferences.
#### 2.5.2 Mixture of Agents (MoA) Orchestration
The inference engine deploys a cascade of three specialized models:
1. **Agent 1 — The Extractor (Qwen-2.5-Coder-32B, T=0.0).** Performs structured extraction from the Sparse JSON tensor. Temperature 0.0 enforces fully deterministic output; no creative inference is desired at this stage.
2. **Agent 2 — The Process Scientist (DeepSeek-R1-32B, T=0.6).** Performs causal diagnostic reasoning via a hidden Chain-of-Thought (`<think>`) layer. The elevated temperature permits exploratory reasoning across plausible process failure modes.
3. **Agent 3 — The Executive (Llama-3.3-70B, T=0.1).** Generates operator-facing narrative output, constrained by strict semantic prompts to suppress hallucination in the final deliverable.
Temperature differentiation across agents is deliberate: each agent's stochasticity is calibrated to the epistemic demands of its role.
#### 2.5.3 In-Context Learning and Zero-Shot Domain Grounding
QID-NPI employs In-Context Learning (ICL) rather than model fine-tuning, injecting domain knowledge via the system prompt at inference time. This eliminates supervised re-training cycles and allows process knowledge to be updated by editing prompt templates — a maintenance model accessible to process engineers without ML expertise.
### 2.6 Layer 4 — Persistence and Air-Gapped QR Delivery
For operator contexts without SCADA access, QID-NPI compresses process step data into a flag-annotated format and encodes it as a QR code for local on-device mobile inference. This layer ensures that the system's advisory output remains accessible even when plant network segments are unavailable, satisfying the *graceful degradation* requirement of [CISA2025].
---
## 3. Deployment Reference and Operational Validation
### 3.1 Facility Description
The reference deployment is a premium pasta filata cheese manufacturing facility in Rincón de Romos, Aguascalientes, México, operating four horizontal agitator curd vessels at an average production cadence of 3 continuous batches per day.
### 3.2 Empirical Evaluation: Dense vs. Sparse Payload
We present a detailed case analysis of a representative complex batch (BatchID 644557334). This batch was selected for its elevated alarm density and multi-step operator interventions, representing a stress-test scenario for context assembly. Aggregate statistical evaluation across the full historian is reserved for future work as additional production batches are logged under the QID-NPI instrumentation schema.
**Table 1: Context Payload Performance (BatchID 644557334)**
| Metric | Dense CSV Format | Sparse JSON Format | Improvement |
| :--- | :--- | :--- | :--- |
| **Character Count** | ~11,850 chars | ~2,610 chars | 78% reduction |
| **Token Count** | ~2,950 tokens | ~650 tokens | 77% reduction |
| **Time to First Token (TTFT)** | 4.8 seconds | 1.2 seconds | 75% faster |
| **Diagnostic Accuracy** | Missed manual alarm | 100% deterministic attribution* | Critical |
*\*Footnote: Alarm-to-step attribution is performed by the SQL Gaps-and-Islands pattern prior to LLM ingestion, which is deterministic by construction. The LLM diagnostic reasoning based on this attribution remains probabilistic.*
---
## 4. Regulatory and Security Alignment
QID-NPI aligns directly with the December 2025 CISA/NSA/FBI joint guidance *Principles for the Secure Integration of Artificial Intelligence in Operational Technology* [CISA2025]:
- **Principle 1 – Understand AI:** Satisfied through edge-native architecture and PLC-level semantic event emission, which make the system's information inputs fully auditable.
- **Principle 2 – Consider AI Use in the OT Domain:** Operates entirely within the plant network boundary with zero data egress, eliminating the attack surface associated with cloud API calls.
- **Principle 3 – Establish AI Governance:** Implements complete SQL-based audit logging and human-initiated inference triggers, ensuring every diagnostic output is traceable to a specific batch and operator session.
- **Principle 4 – Embed Oversight:** Operates strictly in an advisory capacity. The QR delivery failsafe ensures that even in degraded network conditions, operators retain access to structured process context without automated actuation.
---
## 5. Roadmap — Vector Knowledge Layer
### 5.1 Retrieval-Augmented Generation (RAG) Architecture
Phase 2 introduces a vector knowledge layer using `pgvector` and `nomic-embed-text` to append exact manufacturer specification text to the system prompt, grounding model outputs in verified process documentation and reducing dependence on the LLM's parametric knowledge for equipment-specific reasoning.
### 5.2 Golden Batch Matching
Current batch contexts will be compared against a *golden batch* corpus using cosine similarity to identify and diagnose primary deviations from optimal production runs, enabling data-driven quality benchmarking without manual annotation.
### 5.3 ANE-Accelerated Fine-Tuning (Experimental)
We are exploring full transformer training on the Apple Neural Engine (ANE) using Metal APIs [MADERIX2026], which would enable domain-adapted model weights without requiring GPU cluster infrastructure — a meaningful capability for resource-constrained OT environments.

//
## 5. Roadmap — Vector Knowledge Layer

### 5.1 Vector-Driven RAG for Equipment Documentation
Phase 2 introduces a vector knowledge layer utilizing a local PostgreSQL instance equipped with the `pgvector` extension and the `nomic-embed-text` topological mapping model. Unlike traditional keyword-based SQL queries, this architecture maps manufacturer specifications into a 768-dimensional semantic phase space. When the MoA pipeline detects a process anomaly (e.g., "Steam Valve in Manual Mode"), `nomic-embed-text` projects this operational state into a floating-point vector. `pgvector` then calculates the cosine distance between the anomaly vector and the embedded manual paragraphs. By identifying the nearest geometric neighbors, the system appends the exact, contextually relevant manufacturer specification text to the system prompt. This grounds the model's diagnostic output in verified documentation without requiring exact string matches between the PLC tag nomenclature and the manufacturer's terminology.

### 5.2 Golden Batch Matching and the "Infinite Edge Historian"
A critical limitation of the Layer 1 SQL Express deployment is its strict 10 GB storage cap, necessitating a 2-year rolling deletion script that permanently purges historical "golden batches" (optimal production runs). Phase 2 strategically resolves this licensing constraint by deploying a PostgreSQL container directly on the Mac Studio M3 Ultra edge hardware, acting as an isolated "Sidecar" vector vault.

As each batch completes, the extracted Sparse JSON tensor is processed by `nomic-embed-text` to generate its semantic embedding. Both the dense JSON payload and its corresponding 768-dimensional vector are permanently archived in the local PostgreSQL instance. Because the edge hardware possesses ample solid-state storage decoupled from SQL Server licensing, this architecture effectively transforms the edge compute node into an infinite historian.

At inference time, current batch contexts are dynamically compared against this expanding golden batch corpus. Utilizing the Hierarchical Navigable Small World (HNSW) indexing algorithm native to `pgvector`, the system calculates geometric similarities across thousands of historical records in milliseconds. This enables deterministic, data-driven quality benchmarking—diagnosing primary process deviations by mathematically mapping how far the current batch vector has drifted from the historical optimum.

### 5.3 ANE-Accelerated Fine-Tuning (Experimental)
We are exploring full transformer training on the Apple Neural Engine (ANE) using Metal APIs [MADERIX2026], which would enable domain-adapted model weights without requiring GPU cluster infrastructure — a meaningful capability for resource-constrained OT environments.


---
## 6. Deployment Scalability Considerations
QID-NPI's architecture is designed to accommodate facilities at varying levels of data infrastructure maturity, as defined by the AI Maturity and Readiness Index for Manufacturing (AIMRI) [INCIT2024].
**Track 01 — Accelerated Edge-Native Deployment.** For facilities with Level 4/5 semantic data readiness, where PLCs already emit structured, labeled events. Deployment bypasses the data re-engineering phase and proceeds directly to Context Assembly Engine configuration and MoA orchestration.
**Track 02 — Phased Integration for Legacy Historians.** A three-tier integration path (Assessment → Design → Deployment) for facilities whose historians contain unstructured or sparsely labeled legacy data. The Assessment phase maps existing tag taxonomies to ISA-88 semantic structures; Design configures the Sparse JSON schema; Deployment validates the full pipeline against historical batches before live operation.
---
## 7. Conclusion
We have presented QID-NPI, demonstrating that reliable, offline, domain-specific AI-assisted diagnostics are achievable in production OT environments without cloud infrastructure. The system resolves the Industrial AI Trilemma — data sovereignty, semantic quality, and operational accessibility — through a coherent five-layer architecture in which each layer addresses a specific constraint: edge-only inference eliminates egress, Sparse JSON preprocessing guarantees semantic clarity, and QR delivery ensures last-mile access.
A 77% token reduction translates directly into a 75% reduction in TTFT. The SQL-driven Gaps-and-Islands temporal attribution layer eliminates alarm misclassification that affects dense tabular baselines. The MoA cascade, with calibrated per-agent temperature, separates deterministic extraction from probabilistic causal reasoning and constrained narrative generation.
We consider this work an operational proof-of-concept and a reference architecture for the broader class of air-gapped, neuro-symbolic industrial AI systems — where the *symbolic* component resides in the SQL preprocessing layer and the *neural* component in the quantized LLM cascade — and invite community engagement on evaluation methodology, historian schema standards, and edge inference benchmarking.
---
## Acknowledgments
The authors thank Ricardo Muciño Gómez (Facultad de Ciencias, UNAM) for arXiv submission preparation and LaTeX typesetting assistance. The authors thank the premium Aguascalientes facility for permission to reference the deployment.
---
## References
    [BOUCHARD2024] Bouchard, L.-F., & Chaudhary, A. "Building LLMs for Production." 2024.
    [CISA2025]     CISA, NSA, FBI et al. "Principles for the Secure Integration of
                   Artificial Intelligence in Operational Technology." December 3, 2025.
    [HIVEMQ2025]   HiveMQ. "Establishing Real-Time Data Flow for Agentic AI in Manufacturing."
                   December 2025.
    [INCIT2024]    INCIT. "AI Maturity and Readiness Index for Manufacturing (AIMRI)." 2024.
    [JAITLY2024]   Jaitly, N. et al. "Better Serialization of Tabular Data for LLMs."
                   arXiv:2402.17944, 2024.
    [LLMTS2023]    Jin, M. et al. "Time-LLM: Time Series Forecasting by Reprogramming
                   Large Language Models." arXiv:2310.01728, 2023.
    [LOSTMIDDLE]   Liu, N. F. et al. "Lost in the Middle: How Language Models Use Long Contexts."
                   Transactions of the Association for Computational Linguistics, 2023.
                   arXiv:2307.03172
    [MADERIX2026]  maderix (M. Singh). "Inside the M4 Apple Neural Engine, Part 3:
                   Training." March 2026.
    [SIEMENSCOP2024] Siemens AG. "Industrial Copilot." Product documentation, 2024.
