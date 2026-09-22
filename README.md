# LLM Tool Calling System

A simple implementation of an **LLM tool-calling system** using a Qwen model.

The project demonstrates how an LLM can select and invoke deterministic tools, validate the generated arguments, execute the selected tool, and use the tool result to produce a final response.

---

##  Current Problems and Limitations

This project intentionally exposes several important limitations of LLM-based tool calling.

### 1. LLM tool-selection errors


```text
What is current population of india and china, and how many percentage india is ahead of china in population?
```

requires up-to-date external information and should use the `web_search` tool.

However, the LLM may incorrectly select the `calculator` tool because tool selection is generated probabilistically by the language model.

This demonstrates an important property of LLM tool-calling systems:

```text
LLM tool selection
        ↓
     probabilistic
        ↓
   can be incorrect
```

The system therefore cannot blindly assume that the LLM always chooses the correct tool.

---

### 2. Invalid tool arguments

The LLM can select the correct tool but still generate arguments that the tool cannot execute.

For example, the calculator expects a mathematical expression such as:

```text
125 * 48
```

but the LLM may generate multiple assignments:

```text
India_population = 1387000000;
China_population = 1426500000;
...
```

The tool schema may be valid because the argument is still a string, while the actual calculator implementation may reject the expression.

This demonstrates the difference between:

```text
Schema validation
        ↓
"Is the tool call structurally valid?"
```

and:

```text
Execution validation
        ↓
"Can the tool actually execute these arguments?"
```

---

### 3. Tool execution failures

Tools can fail even after argument validation.

Examples include:

* Invalid expressions
* Unsupported units
* Network failures
* Empty search results
* API errors
* Rate limits
* Invalid external API responses

A production agent therefore needs explicit tool-error handling rather than assuming every tool execution succeeds.

---

### 4. LLM hallucination after tool failure

A particularly important failure case occurs when a tool returns an error.

For example:

```text
Tool Result:
{
    "status": "error",
    "result": null
}
```

The LLM may still attempt to produce a final answer instead of recognizing that the tool failed.

This demonstrates why tool results must be validated before they are passed into the final-answer stage.

A robust system should distinguish:

```text
Successful tool result
        ↓
     continue
```

from:

```text
Tool error
        ↓
retry / recover / report failure
```

These failure modes are part of the motivation for the next stages of this project: improved orchestration, retries, tool-result validation, and agent loops.

---

# Project Overview

This project implements the fundamental mechanics of **LLM tool calling** using a locally loaded Qwen model.

The basic lifecycle is:

```text
User Question
      ↓
    LLM
      ↓
Tool Selection
      ↓
Argument Validation
      ↓
Tool Execution
      ↓
   Tool Result
      ↓
    LLM
      ↓
 Final Answer
```

The LLM does not directly execute tools.

Instead:

```text
LLM
 ↓
generates tool call
 ↓
application validates it
 ↓
application executes the tool
 ↓
tool result is returned
 ↓
LLM generates final response
```

This separation allows deterministic application code to control tool execution instead of blindly trusting generated model output.

---

# Features

* Local LLM inference using **Qwen2.5-1.5B-Instruct**
* 4-bit quantization using **BitsAndBytes NF4**
* Automatic tool selection by the LLM
* Tool argument generation
* Tool argument validation
* Tool execution
* Real web search using the **Tavily Search API**
* Passing tool results back to the LLM
* Final natural-language response generation
* Basic handling of invalid tool arguments and tool execution failures

---

# Available Tools

## 1. Calculator

Used when the user requires an exact mathematical calculation.

```json
{
  "tools_required": true,
  "tool_name": "calculator",
  "arguments": {
    "expression": "125 * 48"
  }
}
```

The application validates the generated arguments and executes the calculator.

---

## 2. Unit Converter

Supports basic conversions such as:

Example tool arguments:

```json
{
  "tools_required": true,
  "tool_name": "unit_converter",
  "arguments": {
    "value": 5,
    "from_unit": "km",
    "to_unit": "miles"
  }
}
```

The generated arguments are validated before execution.

---

## 3. Web Search

The project now uses the **Tavily Search API** to perform real web searches.

The web-search flow is:

```text
LLM
 ↓
web_search(query)
 ↓
Tavily Search API
 ↓
Search results
 ↓
Result parsing
 ↓
LLM
 ↓
Final answer
```

Example:

```text
User: What is agentic AI?
```

The LLM can generate:

```json
{
  "tools_required": true,
  "tool_name": "web_search",
  "arguments": {
    "query": "what is agentic AI"
  }
}
```

The application sends the query to Tavily and returns relevant search results to the LLM.

The raw Tavily response is converted into a simpler internal structure before being passed to the model.

Conceptually:

```text
Tavily API response
        ↓
Result parser
        ↓
Compact internal format
        ↓
LLM
```

This prevents the rest of the application from depending directly on the complete external API response structure.

---

# Tool Calling Flow

The system first asks the LLM whether a tool is required.

If no tool is required:

```text
User
 ↓
LLM
 ↓
Final Answer
```

If a tool is required:

```text
User
 ↓
LLM
 ↓
Tool Selection
 ↓
Argument Validation
 ↓
Tool Execution
 ↓
Tool Result
 ↓
LLM
 ↓
Final Answer
```

For web search:

```text
User
 ↓
LLM
 ↓
web_search
 ↓
Tavily API
 ↓
Search Results
 ↓
LLM
 ↓
Final Answer
```

---

# Validation

Before executing a tool, the generated arguments are validated.

Validation checks include:

* Required fields
* Argument data types
* Empty values
* Supported units
* Valid tool names
* Tool-specific argument requirements

For example, an invalid unit conversion is rejected instead of being executed.

The basic safety boundary is:

```text
LLM-generated arguments
        ↓
     Validation
        ↓
   Tool execution
```

rather than:

```text
LLM
 ↓
direct execution
```

The project therefore treats the LLM as a **tool caller**, not as a trusted execution environment.

---

# Project Structure

```text
.
├── ToolCalling.ipynb
├── README.md
└── .gitignore
```

---

# What This Project Demonstrates

This project focuses on the fundamental mechanics of LLM tool calling:

1. Defining tools
2. Describing tools to an LLM
3. LLM-based tool selection
4. Generating structured tool arguments
5. Validating LLM-generated arguments
6. Executing deterministic tools
7. Integrating an external web-search API
8. Passing tool results back to the LLM
9. Generating a final response
10. Identifying common failure modes in LLM tool-calling systems

---
