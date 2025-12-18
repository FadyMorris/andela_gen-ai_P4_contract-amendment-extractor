# Option 1

# Context & Objectives

You are an AI engineer at a legal technology company that processes
thousands of contract amendments monthly. The legal compliance team
currently spends 40+ hours per week manually comparing original
contracts with their amendments to identify what changed, assess impact,
and route documents for appropriate review. This manual process is
error-prone, slow, and prevents the team from scaling to handle the
growing contract volume.

Your team needs an Autonomous Contract Comparison and Change Extraction
Agent that can receive two scanned contract images (original and
amendment), automatically parse them using vision-capable LLMs,
intelligently extract changes through collaborative agents, and return
structured, validated outputs that downstream legal systems can consume.

## Objectives (what you will deliver and why it matters):

1. Implement multimodal document parsing using GPT-4o, Gemini Vision,
or Claude Vision to convert scanned contract images into structured text
preserving document hierarchy. This eliminates manual OCR preprocessing
and handles real-world document quality variations.

2. Build a two-agent collaborative system where Agent 1 contextualizes
both documents (identifying structure and corresponding sections) and
Agent 2 extracts specific changes using Agent 1's analysis. This mimics
how legal analysts work: first understanding context, then identifying
changes.

3. Return Pydantic-validated structured output with
"sections\_changed", "topics\_touched", and "summary\_of\_the\_change"
fields. This creates a stable contract for downstream systems (legal
databases, review queues, compliance dashboards) and prevents malformed
data from breaking integrations.

4. Instrument complete workflow tracing with Langfuse to capture every
step: image parsing, agent execution, handoffs, and validation. This
enables debugging misclassifications, performance bottlenecks, and cost
analysis for production deployment.

5. Document technical decisions explaining why you chose specific
multimodal models, agent collaboration patterns, and prompt engineering
strategies. This demonstrates engineering judgment beyond "making it
work" and prepares the system for team handoff.

**Why this matters:** This capstone integrates all course modules:
prompt engineering (M1), RAG principles for document understanding (M2),
multi-agent orchestration (M3), and multimodal LLMs with structured
outputs (M4). Real legal tech companies use systems like this to
automate contract analysis, compliance monitoring, and risk assessment.
Mastering multimodal document processing, agent collaboration,
structured validation, and production observability prepares you for
high-impact AI engineering roles.

# Assignment

Create an autonomous system that receives two scanned images. One of an
original contract and one of its amendment. The system first uses a
Multimodal LLM to parse both images into text. The workflow then employs
two collaborative agents: the first agent reads and contextualizes both
documents, and the second agent extracts and isolates the specific
changes introduced by the amendment. Finally, the system returns a
Pydantic-validated output detailing the sections\_changed,
topics\_touched, and a precise summary\_of\_the\_change. All agent
calling actions must be traced using a tracing tool e.g Langfuse.

# Project Deliverables and Submission Requirements

Submit via a public Git repository link of the repository. Ensure the
repository is self-contained and runnable.

## Expected repository structure

<table>
<thead>
<tr>
<th>Deliverable</th>
<th>Filename/Format</th>
<th>Minimum Content</th>
</tr>
</thead>
<tbody>
<tr>
<td>Main application script</td>
<td>src/main.py or contract_agent.py or notebook.ipynb</td>
<td>Entry point that: (1) accepts two image file paths as arguments, (2)
calls multimodal LLM to parse both images, (3) executes Agent 1
(contextualization), (4) executes Agent 2 (change extraction), (5)
validates output with Pydantic, (6) returns structured JSON. Must be
runnable from command line.</td>
</tr>
<tr>
<td>Agent implementations</td>
<td>src/agents/contextualization_agent.py and
src/agents/extraction_agent.py OR sections in notebook</td>
<td>Two distinct agents with separate system prompts and
responsibilities. Agent 1: analyzes both documents, identifies structure
and corresponding sections. Agent 2: receives Agent 1's output, extracts
specific changes. Shows clear handoff mechanism.</td>
</tr>
<tr>
<td>Image processing utilities</td>
<td>src/image_parser.py or functions in main script</td>
<td>Functions for: image validation (format, size), encoding (base64 or
URL), multimodal API calls with proper message formatting,
vision-specific prompts for contract parsing.</td>
</tr>
<tr>
<td>Pydantic models</td>
<td>src/models.py or defined in main script</td>
<td>Pydantic model with exactly these three fields: sections_changed,
topics_touched, summary_of_the_change</td>
</tr>
<tr>
<td>Test contract images</td>
<td>data/test_contracts/</td>
<td>Minimum 2 contract pairs (4 images total): original contract +
amendment for each pair. Images should be realistic scanned documents
(JPEG or PNG, 5-10 pages typical). Include README describing what
changed in each pair.</td>
</tr>
<tr>
<td>Tests</td>
<td>tests/test_agents.py or tests/test_validation.py</td>
<td>At least 2 automated tests: (1) Pydantic validation test (valid and
invalid outputs), (2) Agent handoff test (verify Agent 2 receives Agent
1's output). Bonus: image parsing test, end-to-end integration test.
Include run instructions in README.</td>
</tr>
<tr>
<td>README</td>
<td>README.md</td>
<td>Required sections: (1) Project description (100+ words), (2)
Architecture diagram OR 150+ word explanation of agent workflow and
collaboration pattern, (3) Setup instructions (install, API keys for
OpenAI + Langfuse, test images), (4) Usage with example command, (5)
Expected output format with sample, (6) Technical decisions (why two
agents? why this model? 100+ words), (7) Langfuse tracing guide (how to
view dashboard, 50+ words).</td>
</tr>
<tr>
<td>Dependencies, Environment template</td>
<td>requirements.txt, .env.example</td>
<td>Minimum packages: openai, pydantic, langchain (if used), vector
store library (chromadb/faiss-cpu/pinecone-client), sqlalchemy (if SQL),
python-dotenv. All with version pins. Template showing:
OPENAI_API_KEY=your-key-here, LANGFUSE_PUBLIC_KEY=pk-lf-xxx,
LANGFUSE_SECRET_KEY=sk-lf-xxx, LANGFUSE_HOST=<a
href="https://cloud.langfuse.com" target="_blank"
rel="noopener noreferrer">https://cloud.langfuse.com</a>. No actual keys
included.</td>
</tr>
<tr>
<td>Langfuse integration</td>
<td>Integrated throughout codebase + src/tracing.py (optional
utilities)</td>
<td>Langfuse callbacks configured in all agent calls and LLM
invocations. Traces capture: image parsing (both documents), Agent 1
execution, Agent 2 execution, validation step. Each trace includes
input, output, latency, tokens, cost. Custom metadata: session_id,
contract_id, agent names.</td>
</tr>
</tbody>
</table>

------------------------------------------------------------------------

# Evaluation Rubric

<table>
<thead>
<tr>
<th>Criterion</th>
<th>Excellent</th>
<th>Satisfactory</th>
<th>Incomplete/Unsatisfactory</th>
</tr>
</thead>
<tbody>
<tr>
<td>1. FUNCTIONALITY &amp; CORE REQUIREMENTS</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>1.1 Image Parsing with Multimodal LLM</td>
<td>- Successfully parses both images (original contract + amendment)
with ≥95% text accuracy- Uses GPT-4o, Gemini Vision, or Claude with
vision capabilities- Extracts structured text preserving document
hierarchy (sections, subsections, clauses)- Handles various image
qualities (scanned, photographed, different resolutions)- Code shows
explicit multimodal API calls with image encoding (base64 or URL)- Test
with 2+ contract pairs: both documents fully parsed with minimal
errors</td>
<td>- Parses both images with ≥85% text accuracy- Uses a multimodal LLM
(GPT-4o, Gemini, or Claude)- Extracts readable text from both documents-
Works with standard quality scanned images- Visible API calls with image
input handling- Test with 1 contract pair: both documents successfully
processed</td>
<td>- Text accuracy &amp;lt;85% (missing sections, garbled text,
unreadable output)- Uses OCR library only (e.g. EasyOCR) without LLM
processing- Fails to parse one or both images- Cannot handle typical
scanned document quality- No visible multimodal LLM integration- Crashes
or returns empty output for test contracts</td>
</tr>
<tr>
<td>1.2 Two-Agent Collaborative Architecture</td>
<td>Two distinct agents clearly implemented in code: - Agent 1
(Contextualization): Reads both documents, understands context,
identifies corresponding sections - Agent 2 (Change Extraction):
Receives Agent 1's analysis, extracts specific changes- Agents have
explicit handoff mechanism (Agent 1 output &amp;gt; Agent 2 input)- Uses
LangChain agents OR custom agent implementation with tool calling- Each
agent has distinct system prompts and responsibilities (visible in
code)- Trace shows sequential agent execution: Image Parsing &gt; Agent
1 &gt; Agent 2 &gt; Output- Test shows Agent 1 provides context that
Agent 2 uses for extraction</td>
<td>- Two agents present with different responsibilities- Agent 1
handles document analysis, Agent 2 handles change extraction- Basic
handoff visible (passing data between agents)- Each agent has separate
function or class definition- Trace shows both agents executed- Test
demonstrates collaboration (Agent 2 uses Agent 1's results)</td>
<td>- Only one agent OR both "agents" are same function- No clear
handoff mechanism (agents don't communicate)- Agents have identical
prompts/logic (not truly specialized)- Cannot identify agent boundaries
in code- Trace shows only one agent or no agent pattern- Agent 2 doesn't
use Agent 1's output (independent processing)</td>
</tr>
<tr>
<td>1.3 Pydantic-Validated Output</td>
<td>Defines Pydantic model with exactly these three fields: -
sections_changed: List[str] with specific section identifiers -
topics_touched: List[str] with business/legal topic categories -
summary_of_the_change: str with detailed change description- Output
passes Pydantic validation (.model_validate() or equivalent succeeds)-
All three fields populated with relevant data for test contracts- Uses
type hints and field descriptions in Pydantic model- Handles validation
errors gracefully (try-except with meaningful error messages)</td>
<td>- Defines Pydantic model with all three required fields- Output
validates successfully (.model_validate() works)- All three fields
contain data (not empty)- Basic type annotations present (List, str)-
Validation attempted in code</td>
<td>- No Pydantic model defined (returns dict or JSON without
validation)- Missing one or more required fields- Validation fails
(Pydantic raises ValidationError)- Fields have incorrect types (e.g.,
sections_changed is str instead of List[str])- Output is raw LLM
response without structured parsing</td>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>2. MULTIMODAL &amp; LLM IMPLEMENTATION</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>2.1 Multimodal LLM API Integration</td>
<td>- Uses OpenAI GPT-4o, Google Gemini Vision, or Claude Vision API-
Correctly formats multimodal requests (images + text prompts in same
call)- Image handling: base64 encoding OR URL upload with proper
content-type- API calls include proper parameters: model, max_tokens,
temperature- Error handling for API failures (rate limits, invalid
images, timeouts)- Code shows explicit vision prompt: "Extract all text
from this contract image..."- Successfully processes images up to
typical contract size (5-10 pages)</td>
<td>- Uses a multimodal LLM API (GPT-4o, Gemini, or Claude)- Sends
images to API with text prompts- Basic image encoding present (base64 or
file upload)- Sets model parameter correctly- Handles at least one error
type (try-except block)- Processes standard 1-2 page contracts</td>
<td>- Uses text-only model (GPT-3.5, GPT-4 without vision)- No
multimodal API integration (only OCR library)- Image encoding broken
(cannot send images to API)- Missing model parameter or incorrect model
name- No error handling (crashes on API errors)- Cannot process typical
contract images</td>
</tr>
<tr>
<td>2.2 Text Extraction &amp; Comparison Quality</td>
<td>- Extracted text maintains document structure (sections, clauses,
paragraphs)- Comparison identifies &gt;90% of actual changes between
contracts- Correctly distinguishes: additions, deletions, modifications-
Handles complex changes (multi-paragraph edits, clause reordering)- No
hallucinated changes (false positives &lt; 10%)- Test with 2+ contract
pairs: accurate change detection for all</td>
<td>- Extracted text is readable and mostly accurate- Identifies &gt;
70% of actual changes- Recognizes basic change types (additions or
deletions)- Handles simple changes (single clause edits)- False
positives &lt; 30%- Test with 1 contract pair: identifies main
changes</td>
<td>- Extracted text is unstructured or garbled- Identifies &amp;lt;70%
of changes (misses major amendments)- Cannot distinguish change types-
Misses multi-line or complex edits entirely- Fails to detect obvious
changes in test contracts</td>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>3. TRACING &amp; OBSERVABILITY WITH LANGFUSE</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>3.1 Complete Workflow Tracing</td>
<td>All major steps traced in Langfuse/LangSmith: - Image parsing (2
traces: original + amendment) - Agent 1 execution (contextualization) -
Agent 2 execution (change extraction) - Pydantic validation- Trace shows
clear hierarchy: Parent span with 5+ child spans- Each span includes:
input data, output data, latency, token counts- Spans have descriptive
names: "parse_original_contract", "agent_contextualize",
"agent_extract_changes"- Can open Langfuse dashboard and view complete
workflow tree- Trace metadata includes: session_id, contract_pair_id,
timestamp</td>
<td>- Core workflow traced (image parsing + both agents)- Trace shows 3+
distinct spans- Most spans include input/output- Basic span names
present- Traces visible in Langfuse dashboard- At least one metadata
field (session_id or timestamp)</td>
<td>- Incomplete tracing (&amp;lt;3 steps traced)- All spans at same
level (no hierarchy)- Spans missing input/output data- Generic span
names ("span_1", "step_2")- Cannot access traces in Langfuse- No
metadata</td>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>4. CODE QUALITY &amp; DOCUMENTATION</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>4.1 Code Organization &amp; Structure</td>
<td>At least 5 functions with clear purposes: - parse_contract_image(),
agent_contextualize(), agent_extract_changes(), validate_output(),
main()- Clear separation: image processing &amp;gt; agent logic &gt;
output validation- Uses classes for agents (ContexualizationAgent,
ExtractionAgent) OR well-organized functions- Type hints on functions:
def parse_contract_image(image_path: str) &gt; str:- No code duplication
(DRY principle followed)</td>
<td>- &gt; 3 functions with descriptive names- Logical flow: input &gt;
processing &gt; output- Some type hints present- Limited code
duplication (&lt;2 repeated blocks)- Imports at top of file</td>
<td>- &lt; 2 functions (monolithic code block)- Unclear organization
(cannot identify workflow)- No type hints- Significant duplication (same
code 3+ times)- Poor function names (func1, process, do_thing)</td>
</tr>
<tr>
<td>4.2 Documentation - README &amp; Comments</td>
<td>README includes all sections about: - Project Description: 100+
words explaining the contract comparison system - Architecture: Diagram
OR 150+ word explanation of agent workflow - Setup: 4+ steps (install,
set API keys for OpenAI + Langfuse, prepare test images, run) - Usage:
Shows how to run with example: (python main.py --original contract1.jpg
--amendment contract2.jpg) - Expected Output: Example Pydantic model
output with sample data - Technical Decisions: Why two agents? Why this
multimodal model? (100+ words) - Tracing: How to view traces in
Langfuse- Code has docstrings for major functions- Inline comments for
complex logic (agent prompts, image encoding)</td>
<td>- README with 4+ sections- Project description (50+ words)- Setup
instructions with API keys- Usage example- Architecture explanation (50+
words)- Some docstrings or comments</td>
<td>- No README OR &lt;200 total words- Missing setup instructions- No
usage example- Cannot run project from README alone- No architecture
explanation- No code comments or docstrings</td>
</tr>
<tr>
<td>4.3 Dependencies &amp; Best Practices</td>
<td>- All keys from environment: os.getenv("OPENAI_API_KEY"), no "sk-"
in code- Settings in config file or environment (not hardcoded)- 3+
try-except blocks for API calls and errors- 3+ functions have type
hints, follows style guide, no security issues</td>
<td>- requirements.txt exists with main packages- API keys not hardcoded
(environment variables OR config file)- README mentions setting API
keys- Some error handling (2+ try-except blocks)</td>
<td>- No requirements.txt- Hardcoded API keys in code (CRITICAL SECURITY
ISSUE)- No instructions for API key setup- No error handling (crashes on
errors)- Missing critical dependencies</td>
</tr>
</tbody>
</table>

