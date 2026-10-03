# Reproducible-Research
**question, how to reproduce, expected runtime/cost**

## Open Science and Reproducible Research with Open-Weight LLMs on Amazon Bedrock

**A guide for University of Arizona researchers and graduate students**'

### 1. Framing: what "open" means here

Three terms are often confused, and the distinction is the basis for the rest of this guide:

* **Open-source AI**, as defined by the Open Source Initiative, requires open weights, open code, and enough information about the training data that a skilled person could substantially rebuild the system.
* **Open-weight** means the trained parameters can be downloaded, inspected, and run independently, but the training data and full training pipeline are usually not released.
* **Hosted inference** (Bedrock) means a cloud provider runs those weights for you. That is convenient, but you do not control the serving stack.

All three of the models in scope are best described as **open-weight**, not fully open-source. The licenses also differ:

| Model (Bedrock) |	License | 	Key specifications on Bedrock	| Note for open science |
| :-- | :-- | :-- | :-- |
| Google Gemma 4 (31B dense, 26B-A4B MoE, E2B) |	Apache 2.0	| Gemma 4 31B is dense with a 256K context and suited to reasoning and coding; 26B-A4B is a mixture-of-experts variant optimized for cost and latency; E2B is the smallest, for low-latency interactive use	| Permissive license; supports 35+ languages and multimodal text and image input |
| OpenAI gpt-oss (120B, 20B)	| Apache 2.0	| A reasoning model with chain-of-thought and adjustable reasoning effort levels, and a 128K-token context window	| Text-only; training data not released |
| Meta Llama 4 Maverick	| Llama 4 Community License	| Mixture-of-experts: about 17B active and roughly 400B total parameters; image and text input	| Not OSI-approved. It carries use restrictions and attribution requirements, so read it before redistributing derivatives |



### 2. Possibilities for open, reproducible research

* **Annotation and coding at scale**. Examples include qualitative coding, content classification, entity extraction from public documents, and systematic-review screening. Any of these can be validated against human coders.
* **Research software assistance**. Generating analysis code, unit tests, and documentation, with every artifact versioned in Git.
* **Multilingual corpora**. Gemma 4 is useful for translating, normalizing, and analyzing non-English sources.
* **Cross-model triangulation**. Running the same pipeline across three independently trained model families. Agreement then becomes a robustness check rather than reliance on a single model.
* **Hosted-to-local verification.** This is the decisive advantage of open weights. Results produced on Bedrock can, in principle, be re-run by anyone on the same published weights, for example on UA HPC with vLLM. Closed models cannot offer this.

### 3. Advantages
1. **Independent verifiability**. Weights are pinned by a Hugging Face revision hash, which reviewers and future researchers can retrieve.
2. **Permissive licensing**. Gemma 4 and gpt-oss allow redistribution of fine-tuned derivatives, subject to the Llama caveat above.
3. **Data stewardship**. Bedrock does not use prompts for model training, and inference stays within the chosen AWS region.
4. **Programmatic access**. Gemma 4 is accessed through an OpenAI-compatible API, which simplifies migration from existing OpenAI SDK code, so a single code path can target all three families. 
daily
5. **Methodological transparency**. The model, its version, and its license can be cited precisely.

## 4. Caveats
1. **Hosted inference is not bitwise deterministic**. Even at temperature 0, batching, hardware, and floating-point effects can change outputs. Reproducibility therefore has to be defined statistically (distributional agreement), not as identical strings.
2. **Models are retired**. Bedrock model cards publish lifecycle dates. Gemma 4 E2B, for example, launched March 31, 2026, with end-of-life no sooner than March 31, 2027. A study that depends only on the hosted endpoint has a limited shelf life, which is why archiving raw outputs and pinning local weights matters. 
amazon
3. **Endpoints and model IDs differ**. Gemma 4 is served only on the bedrock-mantle endpoint, while on the bedrock-runtime endpoint, GPT OSS model IDs include a version suffix such as openai.gpt-oss-120b-1:0. The exact endpoint and ID must be recorded. 
4. **Hosted and local outputs may diverge**. Quantization, chat templates, and system defaults can differ between Bedrock and a local deployment. The local re-run is a check, not a guaranteed match.
5. **Training-data opacity**. For gpt-oss and Llama 4, contamination with benchmark or test material cannot be ruled out. Avoid evaluating on widely published datasets without acknowledging this.
Hallucination and bias. These remain fully present. LLM output is measurement with error, not ground truth.
Data governance. A public repository must never contain Restricted data, PII, PHI, or credentials. HIPAA-regulated data requires confirmation from UA Privacy and Research Computing before any use.

## 5. Good practices (consistent with FAIR, TOP Guidelines, and COPE)
* **Preregister the research question, model choices, prompts, parameters, and evaluation metrics on OSF before collecting outputs.
* **Treat prompts as code**. Store them as versioned files, never only in a chat interface. Interactive chat sessions are inherently weak for reproducibility.
* **Pin everything:** model ID and version, endpoint, region, temperature, top-p, max tokens, reasoning-effort setting, software lockfile, and the date and time of each run.
* **Log the raw request and response** for every call, in JSONL, with token counts and a content hash.
* **Quantify variability.** Use repeated runs (k ≥ 3 to 5 per item), report agreement statistics, and keep a human-validated subset scored with Cohen's κ or Krippendorff's α.
* **Separate exploration from confirmation**. Prompt iteration happens on a development split. The final evaluation runs once on a held-out split.
* **Disclose AI** use in the methods section. Major publishers and COPE agree that an LLM cannot be listed as an author.
* **Archive and cite**. Use a GitHub release, Zenodo DOI, and CITATION.cff for code. Use UA ReDATA for data and outputs, linked to your ORCID.
* **License deliberately**. Apache 2.0 or MIT for code, CC BY 4.0 for documentation and derived data, and respect each model's license terms.


## 6. Proposed workflow with an open GitHub repository
```
repo/
├── README.md              # question, how to reproduce, expected runtime/cost
├── LICENSE  CITATION.cff  AI_USE.md
├── environment/           # uv.lock or conda-lock; optional Dockerfile
├── config/models.yaml     # model IDs, endpoint, region, parameters, HF revision hash
├── prompts/               # versioned prompt templates (RACE/RISEN)
├── data/                  # public or synthetic only; pointer to ReDATA otherwise
├── src/                   # client wrapper, logging, evaluation
├── outputs/raw/           # JSONL logs of every request/response
├── analysis/              # notebooks → figures, agreement statistics
└── .github/workflows/     # CI: tests + replay from cached outputs (no live API keys)
```

## Stages:
1. **Preregister** (OSF) and create the repository from a template.
2. **Develop** prompts on a dev split through the Bedrock API, logging every call.
3. **Confirm** with a single run across all three models on the held-out split, k repetitions each.
4. **Validate** against a human-coded subset and report agreement and error analysis.
5. **Verify locally** by re-running a sample on pinned Hugging Face weights on UA HPC and reporting the hosted-versus-local concordance.
6. **Archive** with a GitHub release, Zenodo DOI, and ReDATA deposit of the outputs.
7. **Report** with a methods checklist covering models, versions, dates, parameters, prompts, variability, validation, and AI-use disclosure.

**On CI and credentials.** CI should replay analyses from the archived outputs rather than call Bedrock. That keeps credentials out of the public repository and makes the analysis reproducible even after a model is retired. If live calls are ever needed, use GitHub OIDC federation to AWS rather than stored keys.

## 7. Questions to shape the final, experiential version
1. **Deliverable format**. Should this become a 30-slide, 60-minute workshop deck like your previous sessions? Alternatively, it could be a written guide, or a public GitHub template repository that participants fork during the session, either alone or paired with the deck. My suggestion is the deck plus the template repository.
2. **Access path**. Will participants reach these models through the UA GenAI Platform's web interface, or programmatically through a UA-managed AWS account and the Bedrock API? Reproducibility practice depends heavily on programmatic access. If only the web interface is available, the guide should say so plainly and show what can still be documented.
3. **Running example**. Which research task should anchor the hands-on activity? Options include coding open-ended survey responses, screening abstracts for a systematic review, or extracting structured data from public documents. A task with a small, public, human-labeled dataset makes the validation step concrete.
4. **Technical level and local compute**. Can participants be assumed to have Python and Git skills? Is UA HPC access with GPU allocations realistic for the local-verification step, or should that step be a demonstration?

**Sources:**

* [Introducing Gemma 4 models on Amazon Bedrock (AWS)](https://aws.amazon.com/blogs/machine-learning/introducing-gemma-4-models-on-amazon-bedrock/)
* [Gemma 4 E2B model card (AWS Docs)](https://docs.aws.amazon.com/bedrock/latest/userguide/model-card-google-gemma-4-e2b.html)
* [Gemma 4 on Bedrock summary (daily.dev)](https://daily.dev/posts/introducing-gemma-4-models-on-amazon-bedrock-avkj1onbd)
* [Gemma 4 in AWS GovCloud (daily.dev)](https://daily.dev/posts/gemma-4-models-are-now-available-on-amazon-bedrock-in-aws-govcloud-us-west--r1xscvmkl)
* [Run NVIDIA Nemotron and OpenAI GPT OSS models on Amazon Bedrock (AWS)](https://aws.amazon.com/blogs/machine-learning/run-nvidia-nemotron-and-openai-gpt-oss-models-on-amazon-bedrock-in-aws-govcloud-us/)
* [Databricks supported models (gpt-oss license and context)](https://docs.databricks.com/aws/machine-learning/foundation-model-apis/supported-models)

***

Created: 10/03/2026 (C. Lizárraga) <br>
Upated: 10/03/2026 (C. Lizárraga)

