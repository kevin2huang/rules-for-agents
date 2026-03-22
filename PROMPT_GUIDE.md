# Prompt Engineering Guide

_Use this guide to take any rough prompt and systematically improve it before feeding it to an LLM._

---

## My Prompt

```
[Paste your prompt here]
```

When a prompt is provided above, apply the Prompt Improvement Checklist to it. Return the improved prompt inside a single fenced code block (```markdown) so it can be copied in one click. After the code block, provide a brief bullet list of what changed.

---

## Prompt Improvement Checklist

### Step 1: Structure Your Prompt

- [ ] **Assign a role/persona** — Who should the model be? _(see Role / Persona Prompting)_
- [ ] **Add clear instructions** — Use action verbs, specify format, length, audience _(see Strategies for Writing Better Prompts)_
- [ ] **Provide context** — Include all relevant facts, data, or reference material _(see Core Components §2)_
- [ ] **Add examples** if the task is non-obvious — Show 1-3 input/output pairs _(see Core Components §3)_
- [ ] **Use positive instructions** — Say what to do, not what to avoid _(see Controlling Output Format)_

### Step 2: Choose the Right Prompting Technique

- [ ] **Simple task?** → Zero-shot (direct instruction)
- [ ] **Need consistent format?** → Few-shot examples
- [ ] **Complex reasoning?** → Chain-of-thought ("Think step by step")
- [ ] **Multi-step with tools?** → ReAct pattern (think-act-observe)
- [ ] **Need to explore alternatives?** → Tree-of-thought or Branch-Solve-Merge
- [ ] **Iterative refinement?** → Reflexion (validate → retry with feedback)

### Step 3: Format for Your Model

- [ ] **Claude** → Use XML tags (`<instructions>`, `<context>`, `<examples>`) _(see Model-Specific: Claude)_
- [ ] **OpenAI GPT** → Use Markdown headers (`# Identity`, `# Instructions`) _(see Model-Specific: OpenAI)_
- [ ] **Reasoning models (o1/o3/Opus)** → Give high-level goals, do NOT say "think step by step" _(see Choosing a Model)_
- [ ] **Place system instructions** in the system/developer message _(see Message Roles)_

### Step 4: Assemble and Optimize

- [ ] **Important content at the edges** — Not buried in the middle _(see Valley of Meh)_
- [ ] **Restate the question near the end** — Sandwich Technique _(see Prompt Assembly)_
- [ ] **Remove irrelevant context** — Bad context is worse than no context _(see Chekhov's Gun Fallacy)_
- [ ] **Specify output format** — JSON, markdown, prose, XML tags _(see Structured Output)_
- [ ] **Set temperature** — Low for precision, high for creativity _(see Key Principles)_

### Step 5: Validate

- [ ] Does the prompt contain everything the model needs to answer?
- [ ] Would a human expert understand exactly what's being asked?
- [ ] Is the prompt formatted in a style the model has seen in training data?
- [ ] Are few-shot examples representative (no accidental ordering bias)?

---

## Key Principles

Before diving into techniques, keep these fundamentals in mind:

- **LLMs predict tokens, not words.** Character-level tasks (counting letters, anagrams) are surprisingly hard. Break these down explicitly.
- **Early tokens shape everything.** The model builds on its own output — a wrong start cascades. Structure your prompt to lead toward the right first tokens.
- **LLMs hallucinate.** They cannot fact-check themselves. Provide context (RAG), ask them to quote sources, and instruct them to say "I don't know" when uncertain.
- **Sloppy input → sloppy output.** Use proper grammar, standard formatting, and familiar document structures (the Little Red Riding Hood principle).
- **Bad context is worse than no context.** LLMs feel compelled to use everything you give them (the Chekhov's Gun Fallacy). Only include relevant information.
- **Temperature controls creativity.** 0 = deterministic (code, classification); 0.5–1.0 = balanced; 1.0+ = creative. Top-p is a complementary control. OpenAI reasoning models ignore temperature.

---

## Core Components of a Good Prompt

### 1. Prompt Format

The structure and style of your prompt significantly influence how the AI interprets your request. Consider:

- **Natural language questions** — conversational queries the model answers directly.
- **Direct commands** — imperative instructions telling the model exactly what to do.
- **Structured inputs** — prompts with labeled fields, sections, or templates that clearly delineate each part of the request.

Understanding a model's capabilities and preferred format is essential for crafting effective prompts.

### 2. Context and Background Information

Providing context helps the model understand the task and generate more accurate output. Tactics include:

- **Including relevant facts and data** — e.g., "Given that global temperatures have risen by 1°C since the pre-industrial era, discuss the consequences for sea level rise."
- **Referencing specific sources** — e.g., "Based on the attached financial report, analyze profitability over the past five years."
- **Defining key terms** — e.g., "Explain quantum computing in simple terms suitable for a non-technical audience."

### 3. Examples (Few-Shot Learning)

Providing input-output examples within the prompt helps the model implicitly learn the pattern you want. Show a diverse range of possible inputs with desired outputs to steer tone, format, and detail level.

**Drawbacks to watch for:**

- **Scales poorly with context** — If your main question requires extensive context (user history, documents), replicating all that context in each example quickly exceeds the context window. Overly simplistic examples may nudge the model away from deeper reasoning. Few-shot prompting is especially suited to demonstrating output format rather than full task complexity.
- **Anchoring bias** — Examples create preconceived expectations. If all example ratings are 4s and 5s, the model will skew toward high ratings. Use a representatively drawn sample to maintain a realistic distribution.
- **Spurious patterns** — LLMs extrapolate aggressively. If examples happen to be in ascending order, the model may continue that sequence. Shuffle examples to break accidental ordering and include all major classes.

> **Rule of thumb:** Use few-shot examples if they illustrate something otherwise unobvious. If the problem is already clear to the model, don't feel compelled to add examples — they lengthen the prompt and introduce the biases above.

### 4. Identity and Instructions (OpenAI Pattern)

OpenAI recommends structuring developer messages with these sections, roughly in this order:

1. **Identity** — The assistant's purpose, communication style, and high-level goals.
2. **Instructions** — Rules the model should follow, what to do and what to avoid, how to call tools, etc.
3. **Examples** — Sample inputs and desired outputs.
4. **Context** — Additional data (proprietary information, retrieved documents) the model needs. Best positioned near the end, since it may vary per request.

---

## Types of Prompts

### Zero-Shot (Direct) Prompts

Provide a direct instruction or question with no examples. Useful for idea generation, summarization, and translation.

> _"Summarize the main points of the following news article on climate change."_

### One-Shot, Few-Shot, and Multi-Shot Prompts

Provide one or more examples of the desired input-output pairs before the actual prompt. This helps the model understand the task pattern and produce more accurate responses.

> **Example (sentiment classification):**
>
> - Review: "I absolutely love these headphones — sound quality is amazing!" → **Positive**
> - Review: "Battery life is okay, but the ear pads feel cheap." → **Neutral**
> - Review: "Terrible customer service, I'll never buy from them again." → **Negative**
>
> Now classify: "The packaging was beautiful but the product broke on day one."

### Chain-of-Thought (CoT) Prompts

Encourage the model to break down complex reasoning into intermediate steps. Structure few-shot exemplars as `⟨input, chain of thought, output⟩` triples — the model sees the reasoning _before_ the answer.

> **Example:**
>
> Q: Roger has 5 tennis balls. He buys 2 more cans of tennis balls. Each can has 3 tennis balls. How many tennis balls does he have now?
> A: Roger started with 5 balls. 2 cans of 3 tennis balls each is 6 tennis balls. 5 + 6 = 11. **The answer is 11.**

**When to use CoT:**

- Complex multi-step problems (math, logic, planning)
- Commonsense reasoning requiring multiple inferences
- Symbolic reasoning and generalization tasks
- **Not helpful** for simple, single-step problems or very small models

**Key insight:** The model must reason in natural language _before_ the answer — not after. Filler tokens or equations alone don't help. You don't need perfect chains of thought — just include intermediate reasoning steps.

**Self-consistency:** Sample multiple reasoning paths and take the majority answer. Trades compute for accuracy.

### Tree-of-Thought (ToT) Prompts

Extends CoT by exploring multiple reasoning branches, evaluating each, and pruning unpromising paths. Useful for problems requiring search or backtracking.

> _"Imagine three experts are collaborating to answer this question. Each expert will write one step of their thinking, then share it with the group. If any expert realizes their reasoning is flawed, they leave. Continue until a consensus is reached."_

### Zero-Shot CoT Prompts

Combine chain-of-thought with zero-shot by asking the model to reason step-by-step without providing examples. Simply adding "Let's think step by step" triggers chain-of-thought reasoning. Often improves output on reasoning tasks without the cost of crafting exemplars.

### ReAct (Reasoning + Acting)

**ReAct** (Yao et al., 2022) combines reasoning with tool usage in a **think-act-observe** loop:

1. **Thought** — The model reasons about what to do next
2. **Action** — The model invokes a tool (e.g., `Search[entity]`)
3. **Observation** — The tool result is injected into the prompt
4. Repeat until the model calls `Finish[answer]`

ReAct is particularly effective when tasks require gathering external information and reasoning about it iteratively. In decision-making benchmarks, ReAct achieved 71% success vs. 45% for acting without reasoning — the thinking step helps with goal decomposition, state tracking, and exception handling.

> **Key finding:** After fine-tuning with just 3,000 ReAct examples, an 8B model outperformed standard prompting on the original 62B model. Proper reasoning + fine-tuning can substitute for a much larger model.

### Plan-and-Solve Prompting

Adds preemptive planning before step-by-step execution:

> _"Let's first understand the problem and devise a plan to solve it. Then, let's carry out the plan step-by-step."_

This outperforms basic zero-shot CoT on complex tasks because the model commits to a strategy before diving into details.

### Reflexion

After completing a task, if the result fails validation (unit tests don't pass, output doesn't match constraints), the failure messages are fed back into the prompt for a retry. The model learns from its own mistakes on subsequent attempts.

> **Pattern:** Generate solution → Validate → If failed, feed failure report back → Retry with context of what went wrong.

This is especially powerful for code generation, where automated tests provide clear pass/fail signals.

### Branch-Solve-Merge

Fork a problem to N independent LLM conversations (possibly with different perspectives or instructions), then merge their outputs with a final agent that synthesizes the best solution. Useful when:

- Multiple valid approaches exist and you want the best one
- The problem benefits from diverse perspectives
- You can trade compute cost for quality

---

## Strategies for Writing Better Prompts

### 1. Set Clear Goals and Objectives

| Tactic                      | Example                                                                                    |
| --------------------------- | ------------------------------------------------------------------------------------------ |
| Use action verbs            | "Write a bulleted list that summarizes the key findings of the attached research paper."   |
| Define length and format    | "Compose a 500-word essay discussing the impact of climate change on coastal communities." |
| Specify the target audience | "Write a product description targeting young adults concerned with sustainability."        |

### 2. Provide Context and Background

Supply the model with the relevant facts, data, and reference material it needs to generate an informed response.

### 3. Use Few-Shot Examples

Show diverse examples covering the desired style, tone, and detail level so the model can generalize the pattern.

### 4. Be Specific

| Tactic                                | Example                                                                                                                           |
| ------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Use precise language, avoid ambiguity | Instead of "Write something about climate change," → "Write a persuasive essay arguing for stricter carbon emission regulations." |
| Quantify requests                     | Instead of "Write a long poem," → "Write a sonnet with 14 lines exploring themes of love and loss."                               |
| Break down complex tasks              | Instead of "Create a marketing plan," → "1. Identify the target audience. 2. Develop key messages. 3. Choose marketing channels." |

### 5. Iterate and Experiment

- Try different phrasings and synonyms.
- Adjust the level of detail and specificity.
- Test shorter and longer prompts to find the optimal balance.

### 6. Leverage Chain-of-Thought Reasoning

- Ask the model to explain its reasoning process.
- Guide the model through a logical sequence of thought.

---

## Message Roles

Most LLM APIs organize conversation history into typed messages. The exact naming differs, but the pattern is universal: one role carries system-level instructions, one carries user input, and one carries model output.

| Concept                 | OpenAI Term                     | Anthropic (Claude) Term | Purpose                                                                                                               |
| ----------------------- | ------------------------------- | ----------------------- | --------------------------------------------------------------------------------------------------------------------- |
| **System instructions** | `developer` (formerly `system`) | `system`                | Application-level instructions (highest priority). Sets the assistant's rules, identity, and business logic.          |
| **User input**          | `user`                          | `user`                  | End-user instructions and queries. Think of it like function arguments — inputs that the system rules are applied to. |
| **Model output**        | `assistant`                     | `assistant`             | Messages generated by the model. Can also be injected to steer conversation history.                                  |

**Key insight:** System-level messages set the rules; user messages provide the inputs. This separation keeps system logic secure and user input flexible. Both OpenAI and Claude treat system instructions with the highest priority, so place your most important constraints there.

---

## Formatting Prompts with Markdown and XML

Use **Markdown** (headers, lists) to mark distinct sections and communicate hierarchy. Use **XML tags** to delineate content boundaries (e.g., where a reference document starts and ends). XML attributes can define metadata about content.

```markdown
# Identity

You are a coding assistant that enforces snake_case variables in JavaScript.

# Instructions

- Use snake_case names (e.g. my_variable) instead of camelCase.
- Declare variables using "var" for old browser support.
- Return only code, no Markdown formatting.

# Examples

<user_query>
How do I declare a string variable for a first name?
</user_query>

<assistant_response>
var first_name = "Anna";
</assistant_response>
```

---

## Prompt Assembly

Gathering content is only half the job — assembling it into an effective prompt is an engineering challenge in itself. A well-assembled prompt has a strong introduction, strategically ordered content, and a clean transition into the completion.

### The Valley of Meh (Lost in the Middle)

Research shows that LLMs pay strong attention to the **beginning** and **end** of the prompt but lose focus in the middle. Information buried in the center is more likely to be ignored or misremembered.

**Mitigations:**

- Place the most important context near the **edges** of the prompt (beginning and end).
- Order middle content by relevance, with the most important items closest to the edges.
- Use the **Sandwich Technique**: state the main question at the beginning of the prompt and **again near the end**, so the model has the question fresh when it begins generating.

### Snippet Formatting Principles

When inserting context snippets (retrieved documents, user data, examples) into the prompt, follow four principles:

- **Modularity** — Each snippet should be self-contained. Removing it shouldn't break the prompt; adding it shouldn't conflict with other content.
- **Naturalness** — Format snippets in ways the model has seen frequently (markdown headers, bullet points, code blocks).
- **Brevity** — Every token counts. Verbose snippets dilute the model's attention and waste context window budget.
- **Inertness** — A snippet should convey information without being misinterpreted as an instruction or contradicting other parts of the prompt.

### The Chekhov's Gun Fallacy

> The playwright Anton Chekhov advised: "If in the first act you have hung a pistol on the wall, then in the following one, it should be fired." LLMs follow this principle — they feel compelled to use every piece of context they receive, even irrelevant ones.

**Including bad context is worse than including no context.** The only mitigation is to retrieve the right snippets. When using RAG or dynamic context, prioritize precision over recall.

### Elastic Snippets

An **elastic snippet** has multiple versions of varying length — from a short summary to a detailed version. When assembling the prompt:

- If space is abundant, use the long version for richer context.
- If space is tight, use the short version to fit more snippets.
- The assembly algorithm dynamically selects which version based on the remaining token budget.

This allows prompts to gracefully degrade as context grows, rather than hard-cutting snippets entirely.

### Priority Tiers

Assign each prompt element a **priority tier** (integer) and a **relevance score** (within tiers):

| Tier            | Content                                                     | Policy                                    |
| --------------- | ----------------------------------------------------------- | ----------------------------------------- |
| **0 (highest)** | System message, user query, core instructions               | Always include                            |
| **1**           | Directly relevant context (matching snippets, key examples) | Include if space permits                  |
| **2**           | Background information, additional examples                 | Include only after tier 1 is satisfied    |
| **3+**          | Nice-to-have context                                        | Include only if significant space remains |

Higher-priority elements are always included before lower-priority ones. Within a tier, higher-scored elements take precedence. This turns prompt assembly into a **knapsack problem** — maximizing total value within a fixed token budget.

### Prompt Assembly Algorithms

- **Additive greedy** — Start with mandatory elements (tier 0). Repeatedly add the highest-value remaining element that fits. Most common in practice.
- **Subtractive greedy** — Start with all elements. If total exceeds budget, remove the lowest-value element. Repeat until everything fits. Tends to include more context.
- **Elastic selection** — Combine with elastic snippets: start with long versions, downgrade to short versions before removing elements entirely.

---

## Choosing a Model

### General Model Tiers

| Tier                     | Characteristics                                                                                                                            | Prompting Style                                                                  |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------- |
| **Reasoning / Frontier** | Excel at complex tasks, multi-step planning, and long-horizon reasoning. Generate an internal chain of thought. Slower and more expensive. | Give high-level goals; let the model plan details. Avoid over-prescribing steps. |
| **Standard / Mid-tier**  | Fast, cost-efficient, highly intelligent. Benefit from explicit, precise instructions.                                                     | Provide step-by-step guidance and detailed constraints.                          |
| **Small / Fast**         | Optimized for speed and cost. Best for simple, well-scoped tasks.                                                                          | Keep prompts short and direct; avoid complex reasoning demands.                  |

### Model Families

| Provider      | Frontier / Reasoning | Standard         | Small / Fast              |
| ------------- | -------------------- | ---------------- | ------------------------- |
| **OpenAI**    | o1, o3, o4-mini      | GPT-4o, GPT-4.1  | GPT-4o-mini, GPT-4.1-nano |
| **Anthropic** | Claude Opus 4        | Claude Sonnet 4  | Claude Haiku 3.5          |
| **Google**    | Gemini 2.5 Pro       | Gemini 2.0 Flash | Gemini 2.0 Flash-Lite     |

> **Note:** Model names change frequently. Check each provider's documentation for the latest model identifiers and capabilities.

---

## Context Window and Prompt Caching

- **Context window** — Models have a token limit for how much data they can consider per request (ranging from ~100k to 1M+ tokens depending on the model).
- **Prompt caching** — Place reusable, static content at the _beginning_ of your prompt to maximize cost and latency savings from caching.

---

## Retrieval-Augmented Generation (RAG)

Adding relevant external context to a prompt — from vector database queries, uploaded documents, or other sources — is known as RAG. This is useful for:

- Giving the model access to proprietary or up-to-date data outside its training set.
- Constraining responses to a specific set of trusted resources.

### Retrieval Strategies

| Strategy                  | How It Works                                                      | Strengths                                               | Weaknesses                                            |
| ------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------- | ----------------------------------------------------- |
| **Lexical (BM25/TF-IDF)** | Matches based on word overlap, weighted by rarity                 | Fast, debuggable, tunable, no infrastructure            | Fails on synonyms, typos, cross-language              |
| **Neural (embeddings)**   | Converts text to vectors; finds nearest neighbors in vector space | Matches on meaning, works across synonyms and languages | Opaque, requires indexing pipeline and vector storage |
| **Hybrid**                | Combines lexical and neural retrieval                             | Best of both worlds                                     | More complex to implement                             |

### Snippetizing Documents for RAG

Break documents into chunks appropriate for embedding and prompt inclusion:

- **Moving window** — Fixed-size chunks with overlap to avoid splitting ideas at boundaries.
- **Natural boundaries** — Split at paragraphs, sections, or headings.
- **Augmented snippets** — Add surrounding context (e.g., section header, parent class definition) to each chunk for better comprehension.

### Summarization as an Alternative to Retrieval

When you need the model to consider a large document holistically (not just relevant snippets), use **hierarchical summarization**:

1. Split into semantic units (chapters, sections).
2. Summarize each unit.
3. Summarize the summaries.
4. Recurse if needed.

**Caution — the rumor problem:** Each level of summarization introduces a small chance of misunderstanding that compounds, like a game of Telephone. Keep summaries generous enough to avoid excessive information loss.

**General vs. specific summaries:** General summaries are reusable but may lose critical details. Specific summaries tailored to your question ("What in this document helps me answer X?") are much more powerful but must be regenerated if the question changes.

---

## Additional Universal Techniques

### Role / Persona Prompting

Assigning a role or persona focuses the model's tone, vocabulary, and depth. All major models respond well to this technique.

> _"You are a senior financial analyst with 15 years of experience in equity research. Analyze the following earnings report…"_

Best practices:

- Be specific about expertise level, communication style, and domain.
- Combine with instructions that constrain scope (e.g., "Do not provide investment advice").
- Place the role in the system message when using the API, or at the very top of a single-turn prompt.

### Structured Output (JSON, CSV, Tables)

When you need machine-readable output, explicitly request the format and provide a schema or example.

> _"Return a JSON array of objects with keys: title (string), author (string), year (integer)."_

Tips:

- OpenAI offers a **Structured Outputs** mode that guarantees valid JSON against a JSON Schema.
- Claude responds well to XML-wrapped output instructions and will follow a provided schema reliably.
- For all models: providing an example of the desired output shape is more reliable than describing it abstractly.

### Prompt Chaining

Break complex tasks into sequential steps where the output of one prompt becomes the input of the next. This is useful when:

- You need to inspect or validate intermediate results.
- The full task exceeds what a single prompt can handle reliably.
- You want to combine different models (e.g., a fast model for drafting, a reasoning model for review).

> **Pattern:** Generate draft → Review against criteria → Refine based on review.

Both Claude and OpenAI recommend this pattern for production pipelines where quality control matters.

### Controlling Output Format

Steer formatting with positive instructions rather than negative ones:

| Less Effective            | More Effective                                                        |
| ------------------------- | --------------------------------------------------------------------- |
| "Do not use markdown"     | "Write in flowing prose paragraphs with no special formatting."       |
| "Don't be verbose"        | "Respond in 2–3 sentences maximum."                                   |
| "Don't use bullet points" | "Incorporate items naturally into sentences instead of listing them." |

Additional tactics:

- **Match your prompt's style to your desired output.** If your prompt uses markdown, the model will likely respond with markdown.
- **Use XML format indicators** (especially effective with Claude): "Write your analysis inside `<analysis>` tags."
- **Provide a concrete example** of the exact format you want.

### Stop Sequences

A **stop sequence** is a specific string that, when generated by the model, halts further generation. This gives precise control over where the completion ends:

- Saves tokens and cost by preventing unnecessary generation.
- Prevents the model from "running on" into tangential content or unwanted postscripts.
- Can be set to structural markers like `\n#`, `</answer>`, closing code fences, or any delimiter.

> **Tip:** Combine stop sequences with structured prompts. If your prompt uses markdown sections, set `\n##` as a stop sequence to halt generation when the model starts a new section.

### Constrained Sampling (Grammar-Based Output)

**Constrained sampling** restricts the model's token selection at each step to only tokens valid according to a predefined grammar. Rather than hoping the model voluntarily produces well-formatted output, this guarantees format compliance by construction.

| Method                                 | Guarantee                          | Cost                            |
| -------------------------------------- | ---------------------------------- | ------------------------------- |
| **Few-shot examples**                  | Not guaranteed — model may deviate | Cheap, simple                   |
| **Constrained sampling / JSON Schema** | Guaranteed format compliance       | Requires tooling or API support |
| **Fine-tuning**                        | Most reliable across all inputs    | Most expensive                  |

- OpenAI's **Structured Outputs** mode enforces JSON Schema compliance via constrained sampling.
- Tools like `llama-cpp-python` support GBNF grammars for open-source models.
- Especially powerful for producing JSON, SQL queries, function calls, or any format where compliance is critical.

### Tool Definition Best Practices

When giving an LLM access to tools (functions, APIs), the quality of your tool definitions directly impacts the model's ability to use them correctly. These guidelines apply across all providers:

**Selecting tools:**

- Limit the number of tools available at once — more tools = more confusion.
- Tools should partition the domain (cover it broadly, avoid overlap).
- Simpler tools are better — do not copy complex web APIs directly.

**Naming:**

- Use meaningful, self-documenting names (`searchFlights`, not `sf`).
- Avoid concatenated lowercase names like `retrieveemail` — they are hard to parse.

**Arguments:**

- Keep arguments few and simple.
- Remove arguments the model doesn't need; provide defaults for ambiguous values.
- Watch for **argument hallucination** — if the model hasn't been told a value, it will guess (e.g., "my-org", "my-repo").

**Tool outputs:**

- Return only what the model needs — extra "just-in-case" content distracts.
- For errors, provide actionable information so the model can self-correct.

**Dangerous tools:**

- Never rely solely on prompt instructions to prevent dangerous actions ("Don't do X without asking" — the model will sometimes ignore this).
- **Intercept dangerous calls in the application layer** and require explicit user authorization before execution.

### Logprobs for Quality and Classification

**Logprobs** (log-probabilities) reveal the model's confidence in each token prediction. Not all APIs expose them, but when available they unlock powerful techniques:

- **Flag low-confidence completions** for human review.
- **Classification calibration** — Get the full probability distribution across options, not just the top choice. Adjust decision thresholds with calibration constants.
- **Anomaly detection** — Unusually low logprobs on prompt tokens can flag typos, errors, or surprising content.

> **The unique first token problem:** When classification options share a prefix (e.g., "North America" and "Northeast Asia"), the model combines probabilities at the shared token. Use unique labels (A, B, C or short codes) to avoid this.

---

## Model-Specific Guidelines

Different model families have distinct strengths, API patterns, and prompting quirks. This section captures provider-specific best practices. When building a system that may target multiple models, start with the universal techniques above, then layer in model-specific adjustments based on which model is active.

### Anthropic Claude

_Based on [Anthropic's Prompting Best Practices](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/claude-4-best-practices)._

#### Prompt Structure

Claude strongly prefers **XML tags** for structuring prompts. Wrap each section of your prompt in descriptive tags to reduce ambiguity:

```xml
<instructions>
Analyze the document and return the top 3 findings.
</instructions>

<context>
[document content here]
</context>

<output_format>
Return findings as a numbered list with one sentence each.
</output_format>
```

Best practices:

- Use consistent, descriptive tag names (e.g., `<instructions>`, `<context>`, `<examples>`, `<input>`).
- Nest tags for hierarchical content: `<documents>` → `<document index="1">` → `<document_content>`.
- Wrap few-shot examples in `<example>` tags (multiple in `<examples>`) so Claude distinguishes them from instructions.

#### System Prompt

Claude's system prompt is set via the `system` parameter in the API (separate from the messages array). Use it for:

- Role and identity assignment.
- Behavioral rules and constraints.
- Communication style preferences.

```python
message = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    system="You are a helpful coding assistant specializing in Python.",
    messages=[{"role": "user", "content": "How do I sort a list of dicts?"}],
)
```

#### Long Context Prompting

When working with large inputs (20K+ tokens):

- **Place documents at the top** of the prompt, above your query and instructions. Queries at the end can improve response quality by up to 30%.
- **Structure multi-document inputs** with XML metadata:
  ```xml
  <documents>
    <document index="1">
      <source>annual_report_2025.pdf</source>
      <document_content>…</document_content>
    </document>
  </documents>
  ```
- **Ground responses in quotes:** Ask Claude to quote relevant passages before answering. This improves accuracy on long-document tasks.

#### Extended Thinking

Claude offers an extended thinking mode for complex reasoning:

- **Adaptive thinking** (`thinking: {type: "adaptive"}`) — Claude dynamically decides when and how much to think. Control depth with the `effort` parameter (`low`, `medium`, `high`, `max`).
- Use `<thinking>` tags in few-shot examples to demonstrate reasoning patterns.
- **Manual CoT fallback:** When thinking mode is off, you can still prompt "Think through this step by step" and use `<thinking>` / `<answer>` tags to separate reasoning from output.

```python
client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=16000,
    thinking={"type": "adaptive"},
    output_config={"effort": "high"},
    messages=[{"role": "user", "content": "…"}],
)
```

#### Output Formatting

- **Tell Claude what to do, not what to avoid.** Instead of "Don't use markdown," say "Write in flowing prose paragraphs."
- Claude's latest models default to **LaTeX** for math. If you prefer plain text, explicitly request it.
- The formatting style of your prompt influences the response — remove markdown from your prompt to reduce markdown in the output.

#### Tool Use

- Be explicit about when to use tools. "Can you suggest changes?" may produce text suggestions instead of tool calls. Say "Implement the changes using the edit tool."
- Claude's latest models are highly responsive to system prompt instructions about tools — avoid aggressive language like "YOU MUST use this tool" which can cause over-triggering. Use natural phrasing like "Use this tool when…"

#### Agentic Systems

For long-running, multi-step agent workflows:

- **Minimize hallucinations:** Add `<investigate_before_answering>` instructions telling Claude to read files before making claims about them.
- **Control autonomy:** Explicitly state which actions require user confirmation (destructive operations, external API calls).
- **Avoid over-engineering:** Claude's latest models tend to add unnecessary abstractions. Add guidance like "Only make changes that are directly requested. Keep solutions minimal."
- **Subagent orchestration:** Claude can delegate to subagents for parallel workstreams. Guide when subagents are vs. aren't warranted to prevent overuse.

#### Key Claude Differences

| Aspect                  | Claude-Specific Behavior                                                                                     |
| ----------------------- | ------------------------------------------------------------------------------------------------------------ |
| **Formatting**          | Strongly prefers XML tags for prompt structure. Markdown works but XML is more reliable for complex prompts. |
| **Thinking**            | Has built-in extended/adaptive thinking. No need for external CoT scaffolding on supported models.           |
| **Prefill (legacy)**    | Older models support prefilling the assistant response to force a format. Deprecated in Claude 4.6+.         |
| **Verbosity**           | Latest models are more concise by default. Request detailed summaries explicitly if needed.                  |
| **Tool responsiveness** | Latest models are very responsive to tool instructions — dial back aggressive prompting from older models.   |

---

### OpenAI (GPT & Reasoning Models)

_Based on [OpenAI's Prompt Engineering Guide](https://platform.openai.com/docs/guides/prompt-engineering)._

#### Prompt Structure

OpenAI models respond well to **Markdown** for structuring prompts. Use headers, lists, and code blocks to organize sections:

```markdown
# Identity

You are a data analyst specializing in e-commerce metrics.

# Instructions

- Analyze the provided dataset.
- Return the top 3 insights as a numbered list.
- Include supporting data points.

# Context

[dataset here]
```

XML tags also work but are not as strongly emphasized as with Claude.

#### Developer Messages

OpenAI uses `developer` messages (formerly `system`) as the highest-priority instructions. Structure them with these sections in order:

1. **Identity** — Purpose, style, goals.
2. **Instructions** — Rules, constraints, tool usage.
3. **Examples** — Sample I/O pairs.
4. **Context** — Reference data (place near the end since it varies per request).

#### Reasoning Models (o1, o3, o4-mini)

OpenAI's reasoning models generate an internal chain of thought before responding. Key differences from standard GPT models:

- **Do NOT add "think step by step"** — reasoning models already do this internally. Adding it can actually hurt performance.
- **Give high-level goals, not step-by-step instructions.** Let the model plan its own approach.
- **Use `developer` messages** (not `system`) — reasoning models only support `developer` role starting with o1-2024-12-17.
- These models are best for complex multi-step problems, code generation, mathematical proofs, and strategic planning.

#### Standard GPT Models (GPT-4o, GPT-4.1)

- **Be explicit and detailed.** These models perform best with step-by-step instructions.
- **Use few-shot examples liberally** — they significantly improve consistency.
- **Structured Outputs mode** guarantees valid JSON conforming to a provided JSON Schema. Use this for any pipeline that parses model output programmatically.

#### Structured Outputs

OpenAI offers a dedicated Structured Outputs feature that constrains the model to produce valid JSON matching a provided schema:

```python
response = client.chat.completions.create(
    model="gpt-4o",
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "analysis",
            "schema": {
                "type": "object",
                "properties": {
                    "findings": {"type": "array", "items": {"type": "string"}},
                    "confidence": {"type": "number"}
                },
                "required": ["findings", "confidence"]
            }
        }
    },
    messages=[{"role": "user", "content": "…"}],
)
```

#### Key OpenAI Differences

| Aspect                  | OpenAI-Specific Behavior                                                                 |
| ----------------------- | ---------------------------------------------------------------------------------------- |
| **Formatting**          | Markdown is the preferred structuring format. XML works but isn't emphasized.            |
| **Reasoning**           | Reasoning models (o1/o3) have built-in CoT — do not prompt them to "think step by step." |
| **Structured Outputs**  | Native JSON Schema enforcement available. Use for reliable programmatic parsing.         |
| **Developer vs System** | `developer` role replaces `system` for newer models. Both work on GPT-4o.                |
| **Temperature**         | Reasoning models ignore temperature settings. Standard models use temperature 0–2.       |

---

## Model Detection and Adaptation

When building systems that work across multiple LLM providers, the agent or application should detect which model is active and adapt its prompting strategy. Here is a recommended approach:

### 1. Check the Model Identifier

Most APIs return or accept a model identifier string. Use it to determine the provider and tier:

```python
def get_model_config(model_id: str) -> dict:
    """Return prompting configuration based on the active model."""
    model_lower = model_id.lower()

    if "claude" in model_lower:
        return {
            "provider": "anthropic",
            "prefer_xml": True,
            "prefer_markdown": False,
            "has_extended_thinking": True,
            "system_role": "system",
            "supports_structured_output": False,  # use XML or prompt-based
            "is_reasoning_model": "opus" in model_lower,
        }
    elif any(x in model_lower for x in ["o1", "o3", "o4"]):
        return {
            "provider": "openai",
            "prefer_xml": False,
            "prefer_markdown": True,
            "has_extended_thinking": True,  # built-in CoT
            "system_role": "developer",
            "supports_structured_output": True,
            "is_reasoning_model": True,
        }
    elif "gpt" in model_lower:
        return {
            "provider": "openai",
            "prefer_xml": False,
            "prefer_markdown": True,
            "has_extended_thinking": False,
            "system_role": "developer",
            "supports_structured_output": True,
            "is_reasoning_model": False,
        }
    elif "gemini" in model_lower:
        return {
            "provider": "google",
            "prefer_xml": False,
            "prefer_markdown": True,
            "has_extended_thinking": "thinking" in model_lower or "pro" in model_lower,
            "system_role": "system",
            "supports_structured_output": True,
            "is_reasoning_model": "pro" in model_lower,
        }
    else:
        # Safe defaults for unknown models
        return {
            "provider": "unknown",
            "prefer_xml": False,
            "prefer_markdown": True,
            "has_extended_thinking": False,
            "system_role": "system",
            "supports_structured_output": False,
            "is_reasoning_model": False,
        }
```

### 2. Adapt Prompt Formatting

Based on the detected model, adjust how you structure the prompt:

| Decision                            | Claude                                       | OpenAI GPT                              | OpenAI Reasoning (o1/o3)         |
| ----------------------------------- | -------------------------------------------- | --------------------------------------- | -------------------------------- |
| **Structure format**                | XML tags                                     | Markdown headers                        | Markdown headers                 |
| **CoT prompting**                   | Use extended thinking API or prompt manually | Prompt "think step by step"             | Do NOT prompt CoT — built-in     |
| **System message role**             | `system`                                     | `developer`                             | `developer`                      |
| **JSON output**                     | Prompt-based with XML wrapper                | Structured Outputs (JSON Schema)        | Structured Outputs (JSON Schema) |
| **Instruction detail level**        | Clear and direct; moderate detail            | Highly detailed, step-by-step           | High-level goals only            |
| **Long context document placement** | Documents at top, query at bottom            | Documents near end (in context section) | Documents near end               |

### 3. Runtime Model Check (for Agents)

If you are building an agent or assistant that should adapt its behavior at runtime, include a model-check step at the start of execution:

```
Before generating your response:
1. Identify which model you are (check your model identifier or known capabilities).
2. If you are a Claude model: prefer XML tags for structure, use extended thinking for complex reasoning.
3. If you are an OpenAI reasoning model (o1/o3): do not add "think step by step" — you already reason internally.
4. If you are a standard GPT model: use detailed step-by-step instructions and Markdown formatting.
5. Apply the appropriate formatting and reasoning strategy for your model family.
```

---

## Quick Reference

1. **Structure** — Role → Instructions → Examples → Context. System message for rules, user message for inputs.
2. **Be specific** — Action verbs, quantified requirements, defined formats. Positive instructions only.
3. **Show, don't tell** — Few-shot examples beat abstract descriptions. Shuffle to avoid bias.
4. **Match the model** — Reasoning models: high-level goals. Standard models: step-by-step. Claude: XML. OpenAI: Markdown.
5. **Assemble strategically** — Important content at the edges. Sandwich Technique. Only relevant context.
6. **Control output** — Stop sequences, constrained sampling, structured output modes.
7. **Iterate** — Test, review, refine. Try different phrasings. Adjust detail level.

---
