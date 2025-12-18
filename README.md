# Contract Amendment Extractor

## Project Description

The Contract Amendment Extractor is an autonomous system designed to automatically analyze and extract changes between original contracts and their amendments. This solution addresses a critical pain point in legal operations where compliance teams spend 40+ hours per week manually comparing contract documents to identify what changed, assess impact, and route documents for review.

The system leverages multimodal LLMs (Claude Vision, GPT-4o, or Gemini Vision) to parse scanned contract images into structured text, employs a two-agent collaborative architecture to intelligently extract changes, and returns Pydantic-validated structured outputs that downstream legal systems can consume. By automating this workflow, legal teams can scale their contract processing capacity, reduce manual errors, and focus on high-value analysis and risk assessment.

The system integrates prompt engineering, multimodal document understanding, multi-agent orchestration, and production observability with Langfuse tracing to create an enterprise-ready contract analysis platform.

## Architecture & Workflow

### Architecture Diagrams
#### Contract Amendment Extractor Workflow:

```mermaid
---
config:
  flowchart:
    curve: linear
  theme: default
  look: classic
id: 9c7887ba-a979-4c1e-bf40-557121f95cf3
---
graph TD;
	__start__([<p>__start__</p>]):::first
	parse_contract_image_worker_original("parse_contract_image_worker(original)")
	parse_contract_image_worker_amended("parse_contract_image_worker(amended)")
	agent_contextualize(agent_contextualize)
	agent_extract_changes(agent_extract_changes)
	__end__([<p>__end__</p>]):::last
	__start__ -.-> parse_contract_image_worker_original;
	__start__ -.-> parse_contract_image_worker_amended;
	agent_contextualize --> agent_extract_changes;
	parse_contract_image_worker_original --> agent_contextualize;
	parse_contract_image_worker_amended --> agent_contextualize;
	agent_extract_changes --> __end__;
	classDef default fill:#f2f0ff,line-height:1.2
	classDef first fill-opacity:0
	classDef last fill:#bfb6fc
```

#### Testing Workflow with Text Extraction Accuracy Calculation:
The testing workflow extends the main extractor graph to include nodes for calculating text extraction accuracy using embedding cosine similarity. It includes the extractor graph as a subgraph of the test graph.


```mermaid
---
config:
  flowchart:
    curve: linear
  theme: default
  look: classic
id: 53110e16-ecdd-4d91-83af-0367832e2515
---
graph TD;
	__start__([<p>__start__</p>]):::first
    subgraph run_extractor_subgraph["run_extractor_subgraph"]
        direction TB
	    parse_contract_image_worker_original("parse_contract_image_worker(original)")
	    parse_contract_image_worker_amended("parse_contract_image_worker(amended)")
	    agent_contextualize(agent_contextualize)
	    agent_extract_changes(agent_extract_changes)
	    agent_contextualize --> agent_extract_changes;
	    parse_contract_image_worker_original --> agent_contextualize;
	    parse_contract_image_worker_amended --> agent_contextualize;
    end
	get_embeddings(get_embeddings)
	calculate_text_extraction_accuracy(calculate_text_extraction_accuracy)
	__end__([<p>__end__</p>]):::last
	__start__ --> run_extractor_subgraph;
	get_embeddings --> calculate_text_extraction_accuracy;
	run_extractor_subgraph --> get_embeddings;
	calculate_text_extraction_accuracy --> __end__;
	classDef default fill:#f2f0ff,line-height:1.2
	classDef first fill-opacity:0
	classDef last fill:#bfb6fc
```
### System Architecture Overview

The Contract Amendment Extractor uses a **two-agent collaborative pipeline** that mimics how legal analysts work in practice:

**Phase 1: Document Parsing**
- Both original and amended contract PDFs are processed in parallel
- Claude Vision (multimodal LLM) converts scanned images into structured markdown text
- Document hierarchy (sections, subsections, clauses) is preserved during extraction
- Base64 encoding is used for secure image transmission to the vision API

**Phase 2: Contextualization Agent (Agent 1)**
- Receives both extracted markdown documents as input
- Analyzes document structure and identifies corresponding sections between original and amendment
- Generates a comprehensive contextualization report explaining how sections align
- Output becomes the foundation for the extraction agent's analysis

**Phase 3: Change Extraction Agent (Agent 2)**
- Receives the original markdown, amended markdown, AND Agent 1's contextualization report
- Uses the contextual understanding to identify specific changes with high precision
- Extracts three structured outputs: sections_changed, topics_touched, summary_of_changes
- Validates output against Pydantic schema before returning

**Phase 4: Validation & Observability**
- Pydantic model validates all three required fields (sections_changed, topics_touched, summary_of_changes)
- Langfuse traces capture complete workflow: parsing → Agent 1 → Agent 2 → validation
- Each trace includes input/output data, latency, token counts, and custom metadata

### Why Two Agents?

The two-agent architecture is intentional:
1. **Separation of Concerns**: Agent 1 focuses on understanding document structure; Agent 2 focuses on identifying changes
2. **Reduced Hallucination**: Agent 2 makes decisions based on Agent 1's analyzed context, not raw text comparison
3. **Mimics Legal Analysis**: Real legal analysts first understand document context, then identify changes
4. **Improved Accuracy**: This handoff pattern reduces false positives and false negatives in change detection
5. **Scalability**: Each agent can be optimized independently and reused in different workflows

## Setup Instructions

### 1. Installation & Environment Setup

```bash
# Clone the repository
git clone <repository-url>
cd contract-amendment-extractor

# Create a Python virtual environment
python -m venv venv

# Activate the virtual environment
# On macOS/Linux:
source venv/bin/activate
# On Windows:
venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### 2. Set API Keys & Configuration

Create a `.env` file in the project root by copying the template:

```bash
cp .env.example .env
```

Edit `.env` with your API keys:

```dotenv
# OpenAI API Key (for GPT-4o mini completion model)
OPENAI_API_KEY="sk-proj-your-actual-key-here"

# Anthropic API Key (for Claude Vision multimodal parsing)
ANTHROPIC_API_KEY="sk-ant-api03-your-actual-key-here"

# Langfuse API Keys (for tracing & observability)
LANGFUSE_SECRET_KEY="sk-lf-your-secret-key-here"
LANGFUSE_PUBLIC_KEY="pk-lf-your-public-key-here"
LANGFUSE_BASE_URL="https://us.cloud.langfuse.com"

# Model Configuration
VISION_MODEL="claude-haiku-4-5-20251001"
EMBEDDING_MODEL="text-embedding-3-small"
COMPLETION_MODEL="gpt-4.1-mini"
```

**Where to find your API keys:**
- **OpenAI**: https://platform.openai.com/api-keys
- **Anthropic**: https://console.anthropic.com/
- **Langfuse**: https://cloud.langfuse.com (create account, navigate to Settings → API Keys)

### 3. Prepare Test Data

Test contracts should be placed in `data/test_contracts/`:

```
data/test_contracts/
├── 01/
│   ├── 01_original_scan.pdf
│   ├── 01_amendment_scan.pdf
│   ├── 01_original.md
│   ├── 01_amendment.md
│   └── README.md
├── 02/
│   └── (similar structure)
└── 03/
    └── (similar structure)
```

Each pair should include:
- Scanned PDF files (5-10 pages typical, JPEG/PNG quality)
- Corresponding markdown files for the contents of the contract.
- README describing what changed in that pair

**Steps to generate test contracts and amendments:**  
1. The test data can be generated using the prompt provided in `.github/prompts/generate-contract-data.prompt.md` This prompt can be used to generate `.tex` files for original and amended contracts along with a `README.md` describing the amendments.
2. Convert the `.tex` files to Markdown using LaTeX to Markdown converters like `pandoc`.
   ```sh
   pandoc -s original.tex -t markdown_strict -o original.md
   pandoc -s amendment.tex -t markdown_strict -o amendment.md
   ```
3. Compile the `.tex` to PDF using `pdflatex`.
   ```sh
   pdflatex original.tex
   pdflatex amendment.tex
   ```
4. Convert each page to images using tools like `pdftoppm` or similar. 
    ```sh
    pdftoppm -jpeg original.pdf original
    pdftoppm -jpeg amendment.pdf amendment
    ```
5. Then, the images can be combined in a single PDF file representing the scanned contract using imagemagick `convert` command.
   ```sh
    convert original-*.jpg original_scan.pdf
    convert amendment-*.jpg amendment_scan.pdf
   ```

## Usage With Examples

### Running from Jupyter Notebook

The main entry point is `contract-amendment-extractor.ipynb`:

```bash
jupyter notebook contract-amendment-extractor.ipynb
```

Then run the cells in order:
1. **Setup & Imports**: Loads libraries and environment variables
2. **Dataset**: Defines paths to test contract pairs
3. **Agent Definitions**: Defines output schema and LangGraph workflow nodes.
4. **Graph Building**: Compiles the extractor multi-agent graph
5. **Running Tests**: Executes the test workflow on test contracts to output text extraction accuracy results using cosine similarity.

### Example Test Case Execution

```python
# Load test data
input_state = State(dataset[0])

# Run the complete workflow with Langfuse tracing
output_state = graph.invoke(
    input_state,
    config={"callbacks": [langfuse_handler]}
)

# View results
print_highlighted_json(output_state["output"])
```

### Running Extraction Accuracy Tests

The notebook includes a test graph that calculates text extraction accuracy using embedding cosine similarity:

```python
# Create test state
input_test_state = TestState(dataset[0])

# Run extraction + accuracy calculation
output_test_state = test_graph.invoke(
    input_test_state,
    config={"callbacks": [langfuse_handler]}
)

# View accuracy metrics
print(f"Original Accuracy: {output_test_state['original_text_extraction_accuracy']:.3f}")
print(f"Amendment Accuracy: {output_test_state['amendment_text_extraction_accuracy']:.3f}")
```

## Expected Output Format

### Sample Pydantic Model Output

```json
{
  "sections_changed": [
    "Section 2.1: Term and Termination",
    "Section 3.4: Payment Terms",
    "Schedule A: Pricing"
  ],
  "topics_touched": [
    "Contract Duration",
    "Financial Obligations",
    "Renewal Terms",
    "Price Adjustments"
  ],
  "summary_of_changes": "The amendment extends the contract term from 3 years to 5 years, increases the base annual fee from $50,000 to $65,000, adds a 3% annual price adjustment clause, and modifies the automatic renewal conditions to require 60-day notice instead of 30-day notice for non-renewal. Payment terms remain net-30 but now include late payment penalties of 1.5% per month."
}
```

### Output Schema Definition

```python
class OutputSchema(BaseModel):
    sections_changed: list[str] = Field(
        description="List of sections that have changed between the original and amended contract."
    )
    topics_touched: list[str] = Field(
        description="List of business/legal topic categories affected by the changes."
    )
    summary_of_changes: str = Field(
        description="Detailed description of the changes made in the amended contract."
    )
```

## Technical Decisions

### 1. Why Claude Haiku 4.5 for Vision/Parsing?

- **Cost-Effective**: Haiku is significantly cheaper than GPT-4o Vision while maintaining high accuracy for document parsing
- **Multimodal Capability**: Native PDF handling without requiring image conversion
- **Document Hierarchy Preservation**: Claude excels at maintaining structural relationships in formatted documents
- **Trade-off**: Speed vs. accuracy is well-balanced for contract documents (not real-time but very accurate)

### 2. Why GPT-4o Mini for Agent Logic?

- **Cost Optimization**: Mini version sufficient for structured reasoning tasks (not creative generation)
- **Structured Output**: Reliable JSON/schema compliance for downstream systems
- **Proven Performance**: GPT-4o family has excellent instruction-following for multi-step agent workflows
- **OpenAI Ecosystem**: Better integration with LangChain agents and function calling

### 3. Why Two Agents Instead of One?

**Single-Agent Approach (rejected):**
- Would attempt to understand context AND extract changes in one pass
- Higher hallucination rate (agent invents changes that don't exist)
- Harder to debug: unclear which step failed
- Less interpretable for legal compliance teams

**Two-Agent Approach (chosen):**
- Agent 1 focuses solely on structural analysis (narrower scope = fewer errors)
- Agent 2 makes extraction decisions with explicit context (more grounded reasoning)
- Clear handoff mechanism creates explainability (legal teams can audit Agent 1's understanding)
- Parallel inference possible for Agent 1 on both documents (future optimization)

### 4. Why LangGraph Over LangChain?

- **Explicit State Management**: State flows through clearly defined nodes (visualization possible)
- **Streaming Support**: Better for large documents that may take time to process
- **Conditional Routing**: `Send()` allows parallel document parsing (original + amendment simultaneously)
- **Production Ready**: More control over error handling and custom logic

### 5. Why Pydantic for Output Validation?

- **Type Safety**: Ensures downstream systems receive well-formed data
- **Self-Documenting**: Field descriptions appear in generated schemas
- **Error Clarity**: ValidationError messages pinpoint exactly what's wrong
- **Serialization**: Automatic JSON serialization for APIs/databases
- **Prevents Malformed Integration**: Bad data cannot break downstream compliance systems

### 6. Why Langfuse for Tracing?

- **Fine-Grained Observability**: Captures every LLM call, agent step, and handoff
- **Cost Tracking**: Token counts and costs per step (enables ROI analysis)
- **Production Debugging**: Identify which agent/step introduced errors in real usage
- **Team Handoff**: Dashboard allows non-technical stakeholders to understand workflow
- **Performance Bottleneck Identification**: Latency insights guide optimization efforts

## Langfuse Tracing Guide

### Accessing Your Traces

1. **Navigate to Langfuse Dashboard**: https://us.cloud.langfuse.com
2. **Login** with your credentials
3. **Select your project** from the dashboard
4. **View Traces** tab shows all recorded workflow executions

### Trace Hierarchy & Structure

Each workflow execution creates a **parent trace** with **child spans**:

```
LangGraph
28.78s ∑ 0.01371
    parse_contract_image_worker
        ChatAnthropic
    agent_contextualize
        ChatOpenAI
    agent_extract_changes
        ChatOpenAI
```

### Key Metrics to Monitor

| Metric | What It Tells You | Action if High |
|--------|-------------------|-----------------|
| **Parse Latency** | Vision model speed | Consider using faster region or batch processing |
| **Token Count** | Document size/complexity | May need to chunk very long contracts |
| **Agent Latency** | Reasoning time | Refine prompts to reduce unnecessary reasoning |
| **Cost per Document** | Budget impact | Experiment with smaller models for Agent 2 |
| **Error Rate** | System stability | Review failed traces for common issues |

### Example: Debugging a Failed Trace

1. Open a failed trace in the dashboard
2. Expand the failing span (usually "agent_extract_changes")
3. Review **Input** section: Was Agent 1's context properly passed?
4. Review **Output** section: Did the LLM return valid JSON?
5. Check **Model Parameters**: Was temperature set too high (causing inconsistent output)?
6. Compare with a successful trace to identify differences

### Custom Metadata in Traces

The notebook attaches metadata to each trace for easier filtering:

```python
config={
    "callbacks": [langfuse_handler],
    "metadata": {
        "session_id": "session_20250115_001",
        "contract_pair_id": "pair_002",
        "environment": "production",
    }
}
```

This allows you to:
- Filter traces by session or contract pair
- Calculate cost per contract type
- Identify performance trends over time

### Cost Analysis Example

```
Dashboard: Analytics → Cost per Session

Session 1: $0.23 (3 documents analyzed, 45 seconds total)
Session 2: $0.19 (2 documents analyzed, 38 seconds total)
Average: $0.21 per contract pair

At scale (1,000 pairs/month): ~$210 operational cost
Legal team: 40 hours/week = $3,200/week = $13,600/month
ROI: 65x cost savings
```

## Testing
### Test Coverage
- **Pydantic Validation**: Valid/invalid outputs, missing fields, wrong types
- **Agent Handoff**: LangGraph ensures that Agent 1 output correctly passed to Agent 2
- **Text Extraction Accuracy**: Cosine similarity checks against ground truth markdown
- **Integration**: Full workflow end-to-end

### Text Extraction Accuracy Testing
The testing is implemented in the Jupyter notebook `contract-amendment-extractor.ipynb`. There is a `TestGraph` that extends the main extractor graph to add two extra nodes for calculating text extraction accuracy using embedding cosine similarity.


## Requirements

See `requirements.txt` for complete dependency list with version pins. Key dependencies:

- **langchain** (0.1+): Agent orchestration framework
- **langgraph** (0.1+): State graph implementation
- **pydantic** (2.0+): Data validation
- **langchain-anthropic**: Claude Vision integration
- **langchain-openai**: GPT-4o integration
- **langfuse** (3.0+): Production tracing
- **python-dotenv**: Environment variable management

