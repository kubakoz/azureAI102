# Azure AI-102 Complete Certification Cheat Sheet

**This is a comprehensive reference for the AI-102: Designing and Implementing a Microsoft Azure AI Solution exam, current as of the December 23, 2025 skills update.** The exam tests your ability to plan, build, manage, and deploy AI solutions on Azure across six skill domains. It carries a **700/1000 scaled passing score**, runs **100 minutes** with **40–60 questions**, and costs **$165 USD**. Question types include multiple choice, drag-and-drop, code completion (C# or Python—chosen at exam start and locked in), and at least one case study section that cannot be revisited after submission.

---

## Exam domains and their weightings

The April 30, 2025 restructuring fundamentally changed this exam. A new **agentic solutions** domain was added, **generative AI** became its own major section, **Bot Framework** was replaced by Agent Service, and the platform shifted from Cognitive Services to **Microsoft Foundry** (formerly Azure AI Foundry, renamed December 2025).

| # | Skill Domain | Weight |
|---|---|---|
| 1 | Plan and manage an Azure AI solution | **20–25%** |
| 2 | Implement generative AI solutions | **15–20%** |
| 3 | Implement an agentic solution | **5–10%** |
| 4 | Implement computer vision solutions | **10–15%** |
| 5 | Implement natural language processing solutions | **15–20%** |
| 6 | Implement knowledge mining and information extraction solutions | **15–20%** |

---

## Domain 1: Plan and manage an Azure AI solution (20–25%)

This is the highest-weighted domain, covering service selection, deployment, security, monitoring, and Responsible AI governance.

### 1.1 Select the appropriate Microsoft Foundry Services

You must match scenarios to the correct Azure AI service:

| Scenario | Service |
|----------|---------|
| Generate text, code, or images from prompts | Azure OpenAI Service |
| Analyze/classify/detect objects in images | Azure AI Vision |
| Extract text, sentiment, entities from text | Azure AI Language |
| Convert speech ↔ text, translate speech | Azure AI Speech |
| Extract structured data from documents | Azure AI Document Intelligence |
| Search and enrich large document collections | Azure AI Search |

### 1.2 Plan, create, and deploy a Microsoft Foundry Service

**Resource types.** A **multi-service resource** provides a single endpoint and key for Vision, Language, Speech, and more. A **single-service resource** is dedicated to one service (required for Azure OpenAI, AI Search). Every resource has **two API keys** for zero-downtime rotation.

**Deployment options include**: cloud endpoints, Docker containers (on-premises/edge), and CI/CD pipelines via ARM templates, Bicep, or Terraform. Container deployment requires three parameters: `Eula=accept`, `Billing=<endpoint>`, `ApiKey=<key>`.

**Key CLI commands:**
```bash
# Create AI Services resource
az cognitiveservices account create --name <name> --resource-group <rg> \
  --kind AIServices --sku S0 --location eastus

# List keys
az cognitiveservices account keys list --name <name> --resource-group <rg>

# Regenerate a key
az cognitiveservices account keys regenerate --name <name> --resource-group <rg> --key-name key1

# Deploy an OpenAI model
az cognitiveservices account deployment create -g <rg> -n <account> \
  --deployment-name gpt4-deploy --model-name gpt-4 --model-version "0613" \
  --model-format OpenAI --sku-name "Standard" --sku-capacity 1
```

**Endpoint URL formats (memorize these):**

| Service | Endpoint Pattern |
|---------|-----------------|
| AI Services (multi) | `https://<name>.cognitiveservices.azure.com/` |
| Azure OpenAI | `https://<name>.openai.azure.com/` |
| AI Search | `https://<name>.search.windows.net/` |
| Speech (regional) | `https://<region>.api.cognitive.microsoft.com/` |
| Translator (global) | `https://api.cognitive.microsofttranslator.com/` |

### 1.3 Manage, monitor, and secure

**Authentication hierarchy** (from simplest to most secure):

1. **API Key**: Passed via `Ocp-Apim-Subscription-Key` header. Simplest but least secure.
2. **Token Exchange**: POST to `/sts/v1.0/issueToken` → Bearer token valid **10 minutes**.
3. **Microsoft Entra ID**: Uses `DefaultAzureCredential`. **Requires custom subdomain** on the resource. Recommended for production.
4. **Managed Identity**: System-assigned or user-assigned. No credentials stored. Uses Entra ID under the hood.

**Critical RBAC roles:**

| Role | Permissions |
|------|-------------|
| **Cognitive Services User** | Call AI APIs (read-only access) |
| **Cognitive Services Contributor** | Full resource management |
| **Cognitive Services OpenAI User** | Access Azure OpenAI endpoints |
| **Cognitive Services OpenAI Contributor** | Deploy models + manage OpenAI resources |

**Key Vault integration**: Store API keys in Key Vault, access via managed identity. For customer-managed keys (CMK), Key Vault must have **soft-delete and purge protection** enabled, and must be in the **same Azure region** as the AI resource.

**Network security options:**
- **All networks** (default): Allows all traffic
- **Selected Networks + Private Endpoints**: Restricts to specific VNets/IPs
- **Disabled**: Private endpoints only (strongest isolation)

Private endpoints provide a private IP via Azure Private Link. Service endpoints route traffic through Azure backbone. **Speech service requires separate endpoint configuration** with custom domain names when using private endpoints.

**Monitoring**: Enable diagnostic settings in Azure Monitor → route to Log Analytics, Storage, or Event Hubs. Log categories: **Audit**, **RequestResponse**, **AllMetrics**. Data may take up to **2 hours** to appear. Query with KQL in Log Analytics.

### 1.4 Implement AI solutions responsibly

**Microsoft's 6 Responsible AI Principles:**

1. **Fairness** — AI treats all people equitably; avoid bias against similar groups
2. **Reliability & Safety** — Perform consistently under normal and unexpected conditions
3. **Privacy & Security** — Comply with privacy laws; protect data
4. **Inclusiveness** — Empower everyone, including people with disabilities
5. **Transparency** — People understand how the AI works and its limitations
6. **Accountability** — Humans maintain meaningful control; AI is never the final authority on life-affecting decisions

**Content filtering in Azure OpenAI** detects 4 harm categories (**Hate, Sexual, Violence, Self-harm**) at 4 severity levels (**Safe, Low, Medium, High**). Default threshold is **Medium** for all categories on both input and output. Modified filters require Microsoft approval.

**Additional safety features**: Prompt shields (jailbreak detection), protected material detection (copyrighted text/code), groundedness detection (hallucination checking), and custom blocklists.

**Limited Access features** require application: Face API identification/verification, Custom Neural Voice, disconnected containers. **Face detection does NOT require registration** — only identification and verification do. **US police departments are explicitly prohibited** from using facial recognition features.

**Transparency Notes** are published for each AI service documenting intended uses, limitations, and potential harms.

---

## Domain 2: Implement generative AI solutions (15–20%)

### 2.1 Build with Microsoft Foundry

Deploy a **hub** (shared infrastructure), **project** (workspace for a specific solution), and resources. Use **prompt flow** for orchestrating LLM workflows. Implement **RAG (Retrieval Augmented Generation)** by grounding models in your data via Azure AI Search + Azure Blob Storage. Evaluate models with built-in metrics. Integrate via the **Microsoft Foundry SDK**.

### 2.2 Azure OpenAI in Foundry Models

**Key models to know:**

| Model | Use Case |
|-------|----------|
| GPT-4o / GPT-4o-mini | Multimodal chat (text + vision) |
| GPT-4 / GPT-4 Turbo | Text generation, reasoning |
| GPT-3.5 Turbo | Cost-effective chat |
| text-embedding-3-large/small | Semantic search, RAG embeddings |
| DALL-E 3 | Image generation |
| Whisper | Speech-to-text |
| o3 / o4-mini | Enhanced reasoning |

**Deployment types**: Standard (pay-per-token, shared), Global Standard (routes globally for throughput), Data Zone Standard (geographic data residency), Provisioned (reserved PTUs).

**Critical API detail**: Azure OpenAI uses **deployment name** (not model name) in API calls.

```python
from openai import AzureOpenAI

client = AzureOpenAI(
    api_key=os.environ["AZURE_OPENAI_API_KEY"],
    api_version="2024-02-01",
    azure_endpoint="https://<resource>.openai.azure.com"
)

response = client.chat.completions.create(
    model="<deployment-name>",  # DEPLOYMENT name, not model name!
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is Azure?"}
    ],
    temperature=0.7,
    max_tokens=800
)
```

### 2.3 Optimize and operationalize

**Key generation parameters:**
- `temperature` (0–2): Higher = more creative/random
- `top_p`: Nucleus sampling; alternative to temperature
- `max_tokens` / `max_completion_tokens`: Response length cap
- `frequency_penalty` / `presence_penalty`: Control repetition
- `stop`: Stop sequences to end generation

**Prompt engineering techniques**: system messages for persona, few-shot learning with examples, chain-of-thought reasoning, grounding with retrieved context (RAG).

**Fine-tuning vs prompt engineering**: Always try prompt engineering first (cheaper, faster, no training data needed). Fine-tune only when prompt engineering is insufficient for specialized behaviors.

**Token math**: ~4 characters per English token. Billing = input tokens + output tokens.

---

## Domain 3: Implement an agentic solution (5–10%)

This is the newest domain (added April 2025). Expect **5–8 questions**.

### 3.1 Create custom agents

Build agents with **Microsoft Foundry Agent Service** for simpler scenarios or **Microsoft Agent Framework** (Semantic Kernel) for complex orchestration. Key concepts include:

- **Agent role and use cases**: Autonomous task completion, tool use, multi-step reasoning
- **Multi-agent orchestration**: Multiple specialized agents coordinating on complex workflows
- **Tools and skills integration**: Agents can call external APIs, search, code execution
- **Conversation memory**: Maintaining context across turns
- **Testing, optimization, and deployment**: Evaluate agent performance, deploy to production

---

## Domain 4: Implement computer vision solutions (10–15%)

### 4.1 Image analysis with Azure AI Vision

**Visual features available in a single API call**: Caption, DenseCaptions, Tags, Objects, SmartCrops, People, Read (OCR). Image limits: **50×50 to 16,000×16,000 pixels**, max **20MB**, formats JPEG/PNG/GIF/BMP.

```python
from azure.ai.vision.imageanalysis import ImageAnalysisClient
from azure.ai.vision.imageanalysis.models import VisualFeatures

client = ImageAnalysisClient(endpoint=endpoint, credential=AzureKeyCredential(key))
result = client.analyze_from_url(
    image_url="https://example.com/image.jpg",
    visual_features=[VisualFeatures.CAPTION, VisualFeatures.READ, 
                     VisualFeatures.TAGS, VisualFeatures.OBJECTS]
)
```

**OCR (Read API)** extracts printed text in **164+ languages** and handwritten text in **9 languages**. Returns pages → lines → words with bounding boxes and confidence scores. Available as Docker container.

### 4.2 Custom Vision models

Choose between **image classification** (multi-class or multi-label) and **object detection**. Train with labeled images, evaluate with precision/recall/F1, and export to ONNX/TensorFlow/CoreML for edge deployment. Pricing: F0 (Free) and S0 (Standard).

**Key metrics to know:**
- **Precision** = True Positives / (True Positives + False Positives) — "Of predictions, how many correct?"
- **Recall** = True Positives / (True Positives + False Negatives) — "Of actual positives, how many found?"
- **F1 Score** = 2 × (P × R) / (P + R) — Harmonic mean of both

### 4.3 Video analysis

**Azure AI Video Indexer** extracts insights from video/live streams: transcripts, faces, topics, sentiments, brands, scenes. **Spatial Analysis** (Docker container) detects people presence and movement in real-time video with operations: personcount, personcrossingline, personcrossingpolygon.

---

## Domain 5: Implement NLP solutions (15–20%)

### 5.1 Text analysis and translation

**Pre-configured features (no training needed):**

| Feature | Output | Key Detail |
|---------|--------|------------|
| Language Detection | Language code + confidence score | Supports 100+ languages |
| Sentiment Analysis | Positive/Negative/Neutral/Mixed per sentence and document | **Mixed** = opposing sentiments in different sentences |
| Opinion Mining | Aspect-level sentiment linked to specific text spans | Enabled via `show_opinion_mining=True` |
| Key Phrase Extraction | Main concepts from text | Unsupervised, no configuration |
| Named Entity Recognition (NER) | Person, Location, Organization, Event, Product entities | Category + confidence score |
| Entity Linking | Entities linked to Wikipedia articles | Provides disambiguation URL |
| PII Detection | Phone, email, SSN, etc. with redaction | Returns redacted text + entity details |
| Summarization | Extractive and abstractive | Document and conversation summarization |

```python
from azure.ai.textanalytics import TextAnalyticsClient

client = TextAnalyticsClient(endpoint=endpoint, credential=AzureKeyCredential(key))
result = client.analyze_sentiment(documents=["I love this product!"])[0]
# result.sentiment → "positive"
# result.confidence_scores.positive → 0.99
```

**Azure AI Translator** supports **100+ languages** via REST API. **Critical detail**: When using a multi-service key, you MUST include the `Ocp-Apim-Subscription-Region` header. Document translation requires **S1 or D3 tier** (not free), Azure Blob Storage with SAS tokens, and a system-assigned managed identity with **Storage Blob Data Contributor** role.

### 5.2 Speech processing

**Speech-to-Text modes**: Real-time (streaming), Fast (synchronous, faster than real-time), Batch (async for large volumes). **Custom Speech** trains domain-specific acoustic and language models.

**Text-to-Speech** uses neural voices controlled by **SSML (Speech Synthesis Markup Language)** for pitch, rate, volume, speaking styles (cheerful, sad, newscast, etc.), and pronunciation.

```python
import azure.cognitiveservices.speech as speechsdk

# Note: Speech SDK uses (subscription, region) — different from other SDKs!
speech_config = speechsdk.SpeechConfig(subscription=key, region=region)

# Speech-to-Text
recognizer = speechsdk.SpeechRecognizer(speech_config=speech_config)
result = recognizer.recognize_once()

# Text-to-Speech
speech_config.speech_synthesis_voice_name = "en-US-AriaNeural"
synthesizer = speechsdk.SpeechSynthesizer(speech_config=speech_config)
result = synthesizer.speak_text_async("Hello world").get()

# SSML for fine control
ssml = '''<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">
  <voice name="en-US-AriaNeural">
    <prosody rate="slow" pitch="+5%">Hello world</prosody>
  </voice>
</speak>'''
result = synthesizer.speak_ssml_async(ssml).get()
```

**Speech Translation** provides real-time speech-to-speech and speech-to-text translation using `SpeechTranslationConfig`.

### 5.3 Custom language models

**Conversational Language Understanding (CLU)** replaces LUIS. Workflow: create intents → add entities → add utterances → train → evaluate → deploy → test → consume from client app. Supports orchestration workflows connecting CLU, Question Answering, and LUIS projects.

**Custom Question Answering** replaces QnA Maker. Create a knowledge base from URLs, documents, or manual Q&A pairs. Supports **multi-turn conversations**, **active learning** (suggestions from user queries), alternate phrasings, chit-chat personality, and multi-language projects. Export/import knowledge bases for backup.

---

## Domain 6: Knowledge mining and information extraction (15–20%)

### 6.1 Azure AI Search pipeline

**This is one of the most heavily tested topics.** The enrichment pipeline flows:

**Data Source → Indexer → Skillset → Index → Knowledge Store**

| Component | Purpose | Key Details |
|-----------|---------|-------------|
| **Data Source** | Where documents live | Blob Storage, Cosmos DB, SQL, Table Storage, SharePoint, OneLake |
| **Indexer** | Pulls data, runs skills, pushes to index | Scheduled (min 5 min) or on-demand; handles document cracking |
| **Skillset** | AI enrichment pipeline | Built-in + custom skills; requires attached AI resource for billing |
| **Index** | Searchable schema | Fields are searchable, filterable, sortable, facetable, retrievable |
| **Knowledge Store** | Enriched data for analytics | Table projections (Power BI), Object projections (JSON), File projections (images) |

**Built-in skills**: OCR, Image Analysis, Entity Recognition, Key Phrase Extraction, Language Detection, Sentiment, Text Split (chunking), Translation, PII Detection, AzureOpenAIEmbedding.

**Custom skills**: Call Azure Functions or any web API conforming to the skill JSON interface. Use when built-in skills are insufficient.

**Query syntax**: Supports Lucene syntax. `$filter` for OData expressions (`category eq 'Books' and price lt 20`), `$orderby` for sorting, wildcards (`*`, `?`), faceting for category counts.

**Semantic ranking** uses deep learning from Bing to re-rank initial BM25/RRF results. Returns **semantic captions** (highlighted passages) and **semantic answers** (direct answers). Premium add-on feature.

**Vector search** indexes numeric embeddings with HNSW or exhaustive KNN algorithms. **Hybrid search** combines keyword + vector with Reciprocal Rank Fusion (RRF). **Integrated vectorization** handles automatic chunking and embedding during both indexing and query time.

**Search pricing tiers:**

| Tier | Storage | Key Limits |
|------|---------|------------|
| Free | 50 MB | 3 indexes, no SLA |
| Basic | 15 GB/partition | 15 indexes, 1 partition |
| Standard S1–S3 | Scalable | Full feature set |
| S3 HD | Multi-tenant optimized | 3,000 indexes, no indexers |
| Storage Optimized L1/L2 | Maximum storage | Large, slow-changing indexes |

SLA requires **2+ replicas** for queries or **3+ replicas** for queries + indexing.

### 6.2 Azure AI Document Intelligence

**Prebuilt models** (no training needed): Invoice, Receipt, ID Document, Business Card, W-2, Tax Documents, Health Insurance Card, Bank Statement, Check, Contract, Mortgage Documents.

**Custom models**: **Template** (consistent layouts, faster training) vs **Neural** (varied layouts, deep learning). Minimum **5 labeled documents** for training. Training data stored in Azure Blob Storage. **Composed models** combine multiple custom models to auto-route documents.

```python
from azure.ai.formrecognizer import DocumentAnalysisClient

client = DocumentAnalysisClient(endpoint=endpoint, credential=AzureKeyCredential(key))

# Analyze operations are async (Long Running Operations)
poller = client.begin_analyze_document_from_url("prebuilt-invoice", document_url=url)
result = poller.result()  # Must call .result() on the poller!

for doc in result.documents:
    for name, field in doc.fields.items():
        print(f"{name}: {field.value} (confidence: {field.confidence})")
```

Supports: JPEG, PNG, PDF, TIFF, DOCX, XLSX, PPTX, HTML. Max file size **50 MB**, max **2,000 pages**.

### 6.3 Azure Content Understanding in Foundry Tools

A newer service for extracting information from multimodal content. Create OCR pipelines for text extraction, summarize/classify documents, extract entities and tables, and process documents, images, video, and audio through unified pipelines.

---

## Container deployment reference

**Services available as containers**: Speech (STT, TTS), Language (Sentiment, Key Phrases, Language Detection, NER), Vision (Read/OCR), Document Intelligence, Translator.

**Required Docker parameters** (tested heavily):
```bash
docker run --rm -it -p 5000:5000 \
  -e Eula=accept \
  -e Billing=<endpoint_uri> \
  -e ApiKey=<key> \
  mcr.microsoft.com/azure-cognitive-services/<service>/<feature>
```

**Connected containers** must maintain internet connectivity for metering. **Disconnected containers** run fully offline but are a **Limited Access feature** requiring enterprise agreement and Microsoft approval. Disconnected containers use license files and commitment plan pricing.

**Key container facts for the exam:**
- Containers expose the same API as cloud service
- No compliance certifications specific to containers
- No default security on container API — add Nginx/Istio for production
- Customers control update timing (not auto-updated)
- Images from `mcr.microsoft.com` (Microsoft Container Registry)

---

## SDK patterns quick reference

**All Azure AI Python SDKs follow the same initialization pattern:**
```python
from azure.core.credentials import AzureKeyCredential
# API Key auth:
client = ServiceClient(endpoint=endpoint, credential=AzureKeyCredential(key))
# Entra ID auth:
from azure.identity import DefaultAzureCredential
client = ServiceClient(endpoint=endpoint, credential=DefaultAzureCredential())
```

**Exception**: Speech SDK uses `SpeechConfig(subscription=key, region=region)` — different from all other SDKs.

**Long Running Operations (LROs)**: Document Intelligence and some Language operations use `begin_*` methods. You must call `.result()` on the returned poller to get results.

**Key Python packages:**

| Package | Service |
|---------|---------|
| `azure-ai-textanalytics` | Language (sentiment, NER, key phrases, PII) |
| `azure-ai-vision-imageanalysis` | Vision (Image Analysis 4.0) |
| `azure-ai-formrecognizer` / `azure-ai-documentintelligence` | Document Intelligence |
| `azure-cognitiveservices-speech` | Speech (STT, TTS, translation) |
| `azure-search-documents` | AI Search (query, index, indexer management) |
| `openai` (AzureOpenAI class) | Azure OpenAI |

**AI Search has three client types**: `SearchClient` (query documents), `SearchIndexClient` (manage indexes), `SearchIndexerClient` (manage indexers, data sources, skillsets).

---

## Infrastructure as code

**Bicep template for AI Services:**
```bicep
resource account 'Microsoft.CognitiveServices/accounts@2023-05-01' = {
  name: resourceName
  location: location
  kind: 'AIServices'    // Also: OpenAI, TextAnalytics, ComputerVision, SpeechServices, FormRecognizer
  sku: { name: 'S0' }
  properties: { publicNetworkAccess: 'Enabled' }
  identity: { type: 'SystemAssigned' }
}
```

ARM resource type is `Microsoft.CognitiveServices/accounts`. The `kind` property determines the service type. For AI Search, use `Microsoft.Search/searchServices`.

---

## Critical name changes to memorize

| Old Name | Current Name | Exam Impact |
|----------|-------------|-------------|
| LUIS | **Conversational Language Understanding (CLU)** | Different architecture; LUIS deprecated |
| QnA Maker | **Custom Question Answering** | Now under Azure AI Language |
| Form Recognizer | **Azure AI Document Intelligence** | Same APIs, renamed |
| Cognitive Search | **Azure AI Search** | Same service, renamed |
| Cognitive Services | **Azure AI Services** | Umbrella rebrand (July 2023) |
| Azure AI Studio / Azure AI Foundry | **Microsoft Foundry** | Latest rename (Dec 2025) |
| Content Moderator | **Azure AI Content Safety** | Content Moderator deprecated Mar 2024 |
| Computer Vision | **Azure AI Vision** | Part of Foundry Tools |

**⚠ Exam terminology may lag behind the latest announcements.** The exam uses terminology from its listed version date.

---

## Tricky concepts and common exam pitfalls

**Scoring ranges vary by service.** Some services return confidence scores on a **0–1 scale** (Language, Vision), while others may use different ranges. Know which is which.

**Sentiment "Mixed" vs "Neutral"**: Mixed means the document contains opposing sentiments in different sentences. Neutral means no strong sentiment detected.

**Endpoint format trap**: `<name>.cognitiveservices.azure.com` uses resource NAME. `<region>.api.cognitive.microsoft.com` uses resource REGION. Entra ID auth requires the name-based custom subdomain format.

**RAG grounding requirements**: Requires **Azure Blob Storage + Azure AI Search** — not Azure SQL, not Bot Service.

**Knowledge Store projections**: "Analyze enriched data with Power BI" → **Table projections**. "Store enriched JSON documents" → **Object projections**. "Store extracted images" → **File projections**.

**Custom skill answer**: Any scenario requiring "custom logic enrichment" in an AI Search pipeline → the answer is a **Custom Skill** (Azure Function or web API).

**Waterfall dialog**: For "collecting structured input step by step" → **Waterfall dialog with Prompts**.

**Key rotation**: Two keys exist specifically for zero-downtime rotation. Update apps to Key 2 → regenerate Key 1 → update apps to Key 1 → regenerate Key 2.

**Document Intelligence operations are asynchronous**: Always use `begin_analyze_document` → `poller.result()` pattern.

---

## Pricing tiers summary

| Service | Free Tier (F0) | Standard Tier | Key Billing Unit |
|---------|---------------|---------------|-----------------|
| AI Services (multi) | Limited transactions | S0 | Per transaction |
| Azure OpenAI | N/A (requires approval) | S0 | Per token (Standard) or per hour (Provisioned/PTU) |
| AI Vision | Limited calls | S0 | Per transaction |
| AI Language | 5,000 text records/month | S0 | Per 1,000 text records |
| AI Speech | Limited hours | S0 | Per audio hour (STT), per million chars (TTS) |
| Document Intelligence | 2 pages max per request | S0 | Per page analyzed |
| AI Search | 50 MB, 3 indexes, no SLA | Basic/S1/S2/S3 | Per hour per Search Unit |
| Translator | 2M chars/month (text only) | S1 | Per million characters |
| Custom Translator | N/A | Per hour training + hosting + chars | Training + hosting + translation |
| Bot Service | Standard channels free | Premium per-message | Per message (premium channels) |

**Commitment tiers** are available for high-volume usage at discounted rates across most services.

---

## Exam-day strategy and practical advice

The exam provides **access to Microsoft Learn documentation** during the test, but searching consumes valuable time. Reserve approximately **30 minutes for the case study** section. You select C# or Python at the start and **cannot switch languages** after. There is **no penalty for guessing** — always answer every question. Flag uncertain questions and return to them if time permits, but remember case study sections cannot be revisited.

The most commonly tested topics based on community reports are: **Azure AI Search pipelines** (indexers, skillsets, knowledge stores), **Azure OpenAI** (prompt engineering, RAG, content filtering), **NLP services** (CLU, sentiment, NER, PII), **Document Intelligence** (prebuilt vs custom models), **Responsible AI principles and content safety**, **container deployment parameters**, and **security** (keys, RBAC, networking). The new agentic solutions domain is relatively small at 5–10% but expect direct questions on Foundry Agent Service, multi-agent orchestration, and Semantic Kernel concepts.

## Conclusion

The AI-102 exam shifted dramatically with the April 2025 update. The three most important changes to internalize are: **generative AI and agentic solutions now command ~25% of the exam combined**, the platform has moved from Cognitive Services to **Microsoft Foundry** as the unifying framework, and **Responsible AI** is no longer a standalone concept but woven throughout every domain. Study the AI Search enrichment pipeline end-to-end, practice writing prompts with content filtering configurations in Azure OpenAI, and ensure you can read both Python and C# SDK code for Document Intelligence and Speech. The exam rewards understanding of *when to use which service* over memorization of individual API details — focus on scenario-based decision-making alongside the technical specifics in this reference.
