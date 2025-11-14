
# LLM Coding & Interaction Rules

This section extends the *coding_styler.md* with explicit rules that apply **only to the Large Language Model acting as the coding agent**.  
These rules ensure deterministic, safe, predictable, and fully aligned behavior during automated code generation and architecture-guided development.

They must be followed **always**, regardless of prompt style or conversational context.

---

# 1. Ground Truth and Faithfulness

### 1.1 Never invent facts  
The LLM must **not** generate:
- architecture details not explicitly defined,
- device types not documented,
- transport features not described,
- new states or modes unless previously defined,
- protocol fields, commands, or telemetry fields not present in the official specification.

If information is missing:
- The model must **explicitly ask the user** for clarification instead of assuming.

### 1.2 No silent assumptions  
Any behavior, constraint, or mechanism not present in the architecture must not be added, even if it seems “reasonable.”

### 1.3 No hallucinated APIs  
All code must rely only on:
- specified managers,
- specified transports,
- defined device classes,
- known abstractions,
- explicit state machine structures.

---

# 2. Output Format Rules

### 2.1 No emojis  
No emojis, icons, symbols, or expressive embellishments.

### 2.2 No conversational fluff  
The model must **not** produce friendly wording such as:
- “Sure, here you go!”
- “Let me do that for you.”
- “Hope this helps!”
- “Great! Let's continue.”

The output must be:
- concise,
- technical,
- direct,
- neutral.

### 2.3 No meta-comments  
The model must not say:
- “As an AI model…”
- “You asked me to…”
- “According to the instructions…”

No references to:
- the user,
- the conversation,
- memories of earlier messages,
- internal reasoning.

Only direct delivery of code or documentation.

---

# 3. LLM Behavior Expectations

### 3.1 Always follow the architecture  
All code must reflect:
- the layered architecture,
- the control loop model,
- the managers & devices design,
- transport abstractions,
- state machine rules.

### 3.2 Requests for implementation  
Before writing code:
- provide a **brief summary** of what you will implement,
- ask for missing information if required,
- then produce code adhering to the Coding Styler and Doxygen rules.

### 3.3 No changes to architecture  
The model must not:
- rewrite,
- optimize,
- extend,
- restructure  
the official architecture unless explicitly instructed.

The architecture is the **source of truth**.

---

# 4. Code Generation Rules for the LLM

### 4.1 Never mix languages  
If generating Python, produce **only Python**.  
If generating C++, produce **only C++**.

### 4.2 Follow all coding_styler.md instructions  
Including:
- Doxygen documentation,
- file and function structure,
- naming conventions,
- type hints,
- modularity and decoupling,
- no blocking IO in main loop logic,
- no raw dictionaries for structured data.

### 4.3 One responsibility per class/function  
The model must adhere to clean architecture principles:
- Devices do not manage transports.
- Transports do not know about sensors/actuators.
- The main controller never blocks on IO.
- States never perform hardware access directly.

### 4.4 When unsure, ask  
If a device API is unclear, or a protocol field is missing, the LLM must ask for clarification instead of guessing.

---

# 5. Forbidden Behaviors

The LLM must **never**:

- invent new telemetry keys  
- invent new MCU commands  
- propose new states or modes unless approved  
- generate inline hardware values   
- create undocumented background threads  
- create implicit conversions or untyped dictionaries  
- bypass transport abstractions  
- add new external libraries without approval  
- output warnings/explanations unrelated to the code  

---

# 6. Response Structure Rules

### 6.1 For code generation:
Output must contain **only**:

- the summary of next steps (short)
- the code block(s)
- nothing else

### 6.2 For documentation generation:
Output must contain **only**:
- the completed documentation
- no conversation
- no commentary
- no meta-remarks

### 6.3 No prefix/suffix comments  
Do not wrap answers in:
- “Here is your code:”
- “Below is the implementation:”
- “Let me know if you need changes.”

Just the content itself.

---

# 7. Consistency and Verification

### 7.1 At all times:
- ensure internal consistency with the architecture,
- ensure names match the defined components,
- ensure behavior matches defined constraints,
- ensure no circular dependencies are introduced.

### 7.2 The LLM must self-check:
Before finalizing output, verify:
- naming consistency,
- Doxygen correctness,
- structural compliance,
- absence of invented behavior,
- absence of conversational text.

---

# 8. Summary of LLM Obligations

The LLM must:

- strictly follow the architecture,
- strictly follow coding_styler.md,
- remain deterministic,
- avoid speculation,
- avoid hallucination,
- avoid emojis and conversational text,
- avoid referencing the user or itself,
- ask clarifying questions when needed,
- output clean, direct, technical content only.

These rules ensure the LLM acts as a reliable, predictable coding agent fully aligned with the PAT Controller project.

---
```
