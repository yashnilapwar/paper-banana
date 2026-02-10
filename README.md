# Paper Banana: Technical System Report

---

## 1. Project Overview

**Problem Solved:**  
Paper Banana automates the generation of publication-quality academic diagrams and statistical plots from text descriptions, targeting AI scientists and researchers. It streamlines the process of creating complex illustrations for papers, reducing manual effort and improving visual consistency.

**High-Level Idea:**  
The system uses a multi-agent pipeline, leveraging large vision-language models (VLMs) and image generation models (primarily Google Gemini), to convert textual descriptions or data into high-quality diagrams and plots. It retrieves reference examples, plans the diagram, styles it, generates the image, and iteratively refines the output using a critic agent.

**Key Assumptions & Constraints:**
- Assumes access to VLMs (e.g., Gemini) and image generation APIs.
- Requires curated reference sets for retrieval.
- Designed for local execution; not intended for remote or untrusted environments.
- Focused on methodology diagrams and statistical plots, not arbitrary images.

---

## 2. Repository Structure

**Major Directories & Files:**

- `paperbanana/`
  - `core/`: Pipeline orchestration, types, config, utilities.
  - `agents/`: Implements the five agents (Retriever, Planner, Stylist, Visualizer, Critic).
  - `providers/`: VLM and image generation provider interfaces and implementations.
    - `vlm/`: Gemini and other VLM provider code.
    - `image_gen/`: Image generation provider code.
  - `reference/`: Reference set management (curated diagram examples).
  - `guidelines/`: Style guidelines loader (e.g., NeurIPS style).
  - `evaluation/`: VLM-as-Judge evaluation system.
- `configs/`: YAML configuration files (default and user overrides).
- `prompts/`: Prompt templates for all agents and evaluation.
- `scripts/`: Utility scripts for building, curating, and evaluating reference sets.
- `examples/`: Example scripts for generating diagrams and plots.
- `tests/`: Unit and integration tests.
- `mcp_server/`: MCP server for IDE integration.
- `README.md`, `CONTRIBUTING.md`, `SECURITY.md`: Documentation and policies.

**Entry Points & Execution Flow:**
- CLI: `paperbanana/cli.py` (main Typer app for command-line usage).
- Python API: `PaperBananaPipeline` in `core/pipeline.py`.
- MCP Server: `mcp_server/server.py` (for IDE integration).

---

## 3. System Architecture

**Text Diagram:**

```
[User Input]
    |
    v
[CLI/API/MCP Server]
    |
    v
[PaperBananaPipeline]
    |
    v
+-------------------+
|  Multi-Agent Flow |
+-------------------+
| 1. Retriever      |<--[Reference Set]
| 2. Planner        |
| 3. Stylist        |<--[Guidelines]
| 4. Visualizer     |<--[VLM/Image Gen]
| 5. Critic         |
+-------------------+
    |
    v
[Iterative Refinement Loop]
    |
    v
[Final Output (Image + Metadata)]
```

**Core Components:**
- **RetrieverAgent:** Finds relevant reference diagrams.
- **PlannerAgent:** Generates a detailed diagram plan.
- **StylistAgent:** Refines the plan for aesthetics.
- **VisualizerAgent:** Renders the image (via VLM or code).
- **CriticAgent:** Evaluates and suggests improvements.
- **ReferenceStore:** Manages curated example diagrams.
- **ProviderRegistry:** Instantiates VLM and image providers.
- **Settings:** Centralized configuration management.

**Data Flow:**
- Input (text/data) → Reference retrieval → Plan generation → Styling → Image generation → Critique → (repeat) → Output.

**Model Invocation Flow:**
- Inputs (text/data) → Preprocessing (reference/context extraction) → Model call (VLM/image gen) → Postprocessing (image, metadata, evaluation).

---

## 4. Model Details

**What is "Nano Banana"?**
- *Ambiguity:* The codebase and documentation do not mention a model called "Nano Banana". The primary model is Google Gemini (VLM and image gen). If "Nano Banana" refers to a lightweight or internal model, it is not present or is a placeholder.

**Model Loading & Usage:**
- Models are loaded via provider classes (`VLMProvider`, `ImageGenProvider`).
- Instantiated in `ProviderRegistry` based on config.
- Used by agents through a common interface.

**Model-Specific Logic:**
- Resides in `providers/vlm/` and `providers/image_gen/`.
- Abstracted via `VLMProvider` interface (`generate()` method, etc.).

**Interfaces/Abstractions:**
- All VLMs must implement `VLMProvider` (see `providers/base.py`).
- Image generators implement a similar interface.

---

## 5. Execution Flow (Step-by-Step)

1. **User Command/API Call:**  
   CLI, Python API, or MCP server receives input (text or data file).

2. **Configuration Loading:**  
   Loads settings from YAML, environment, or CLI flags.

3. **Pipeline Initialization:**  
   `PaperBananaPipeline` is instantiated with settings.

4. **Reference & Guidelines Loading:**  
   Loads reference diagrams and style guidelines.

5. **Agentic Pipeline:**
   - **RetrieverAgent:** Selects relevant examples.
   - **PlannerAgent:** Generates a textual plan.
   - **StylistAgent:** Refines for style.
   - **VisualizerAgent:** Generates image (calls VLM/image gen).
   - **CriticAgent:** Evaluates and suggests improvements.

6. **Iterative Refinement:**  
   Steps 4-5 repeat for a configured number of iterations.

7. **Output:**  
   Final image and metadata are saved to the output directory.

---

## 6. Configuration & Dependencies

**Environment Variables:**
- `GOOGLE_API_KEY`, `OPENROUTER_API_KEY` (for Gemini/OpenRouter access).
- Loaded from `.env` (never committed).

**Config Files:**
- `configs/config.yaml` (default settings).
- User can override via CLI or custom YAML.

**External Services/APIs:**
- Google Gemini (VLM and image generation).
- OpenRouter (optional VLM provider).

**Hardware Assumptions:**
- No explicit GPU requirement (relies on cloud APIs).
- Sufficient memory for image processing and data handling.

**Dependencies:**
- Python 3.10+
- `httpx`, `pydantic`, `typer`, `matplotlib`, `dotenv`, etc.

---

## 7. Can We Replace Nano Banana with Qwen3-VL-32B?

**YES/NO:**  
**NO** direct replacement, because:
- The codebase is tightly coupled to Gemini's API, prompt format, and multimodal capabilities.
- All VLM logic is abstracted, but prompt engineering, input/output handling, and image generation are Gemini-specific.

**Coupling Points:**
- `providers/vlm/gemini.py` and related provider code.
- Prompt templates in `prompts/` are Gemini-oriented.
- Tokenization, image encoding, and multimodal handling are tailored to Gemini.

**Assumptions:**
- Input: Text + (optionally) images.
- Output: Text (for planning/styling/critique), image (for visualization).
- Gemini-specific API calls, response formats, and error handling.

**Format Differences:**
- Qwen3-VL-32B may use different tokenization, image encoding, and prompt structure.
- Vision encoder and multimodal fusion may differ, requiring new adapters.

---

## 8. How to Replicate the Architecture Using Qwen3-VL-32B

**Step-by-Step Plan:**

1. **Implement a Qwen3-VL-32B Provider:**
   - Create `providers/vlm/qwen3.py` implementing `VLMProvider`.
   - Handle Qwen3-VL-32B API calls, authentication, and response parsing.

2. **Update Provider Registry:**
   - Register the new provider in `ProviderRegistry`.

3. **Adapt Prompts:**
   - Create Qwen3-specific prompt templates in `prompts/`.
   - Adjust for any differences in input formatting.

4. **Handle Multimodal I/O:**
   - Implement image encoding/decoding as required by Qwen3.
   - Ensure input/output formats match Qwen3 expectations.

5. **Test Agent Integration:**
   - Validate that Retriever, Planner, Stylist, Critic agents work with Qwen3 outputs.

6. **Update Config:**
   - Add Qwen3 as a selectable `vlm_provider` in config.

**What Can Stay the Same:**
- Agent logic (Retriever, Planner, etc.) if the interface is respected.
- Pipeline orchestration and config management.

**Required Adapters/Wrappers:**
- Input/output adapters for Qwen3's API.
- Possibly a wrapper for vision encoder differences.

**Performance/Memory Trade-offs:**
- Qwen3-VL-32B may have higher memory/compute requirements.
- API latency and throughput may differ.

**Potential Blockers:**
- Incompatible prompt/response formats.
- Missing features (e.g., image generation) in Qwen3.
- Multimodal fusion differences.

---

## 9. Recommended Refactor (If Needed)

**Redesign for Model-Agnostic Interface:**

- **Abstract Model Interface:**
  - Define a strict `VLMProvider` interface for all VLMs.
  - Use dependency injection for provider selection.

- **Prompt/Response Adapters:**
  - Separate prompt templates per model.
  - Adapter classes for input/output normalization.

- **Suggested Class Structure:**
  ```
  class VLMProvider(ABC):
      def generate(self, prompt, images=None, ...): ...
      def is_available(self): ...
  
  class GeminiProvider(VLMProvider): ...
  class Qwen3Provider(VLMProvider): ...
  ```

- **Pseudocode Example:**
  ```python
  class VLMProvider(ABC):
      @abstractmethod
      def generate(self, prompt, images=None, **kwargs): ...
  
  class ProviderRegistry:
      @staticmethod
      def create_vlm(settings):
          if settings.vlm_provider == "gemini":
              return GeminiProvider(...)
          elif settings.vlm_provider == "qwen3":
              return Qwen3Provider(...)
          else:
              raise NotImplementedError()
  ```

---

## 10. Limitations & Open Questions

- **Ambiguity:** "Nano Banana" is not present in the codebase; all logic is Gemini-centric.
- **Model Coupling:** Prompt engineering and multimodal handling are Gemini-specific.
- **Reference Set:** Only 13 curated methodology diagrams; may limit generalization.
- **Evaluation:** VLM-as-Judge assumes VLM can reliably evaluate images.
- **Migration Risks:** Qwen3-VL-32B may not support all Gemini features (e.g., image generation, prompt structure).
- **Open Questions:**
  - How are vision encoders and multimodal fusion handled in Qwen3?
  - Are there licensing or API access constraints for Qwen3?
  - What are the performance implications of using a much larger model?

---

**If you need further details on any section, or want code-level migration steps, please specify.**
