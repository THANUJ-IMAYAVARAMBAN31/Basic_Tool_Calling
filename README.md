# LLM Tool Calling System

A simple implementation of an **LLM tool-calling system** using a locally loaded Qwen model.

The project demonstrates the basic agent/tool-calling lifecycle:

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

## Features

* Local LLM inference using **Qwen2.5-1.5B-Instruct**
* 4-bit quantization using **BitsAndBytes NF4**
* Automatic tool selection by the LLM
* Tool argument generation
* Tool argument validation
* Tool execution
* Passing tool results back to the LLM
* Final natural-language response generation

## Available Tools

### 1. Calculator

Used when the user requires an exact mathematical calculation.

Example:

```text
User: What is 125 * 48?
```

The LLM can generate:

```json
{
  "tools_required": true,
  "tool_name": "calculator",
  "arguments": {
    "expression": "125 * 48"
  }
}
```

The calculator executes the expression and returns the result.

### 2. Unit Converter

Supports basic conversions such as:

* km ↔ miles
* kg ↔ g
* m ↔ cm

Example:

```text
User: Convert 5 km to miles
```

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

### 3. Web Search

The notebook currently contains a **mock web-search implementation** for demonstrating the tool-calling flow.

It does not perform real internet search yet.

## Tool Calling Flow

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

## Validation

Before executing a tool, the generated arguments are validated.

Validation checks include:

* Required fields
* Argument data types
* Empty values
* Supported units
* Valid tool names
* Tool-specific argument requirements

For example, an invalid unit conversion is rejected instead of being executed.

This separates:

```text
LLM-generated arguments
        ↓
     Validation
        ↓
   Tool execution
```

rather than blindly trusting the LLM output.

## Model

The project uses:

```text
Qwen/Qwen2.5-1.5B-Instruct
```

The model is loaded using Hugging Face Transformers with 4-bit quantization:

```python
BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_use_double_quant=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16
)
```

The notebook also saves the model and tokenizer locally so they can be reused without downloading them again.

## Installation

Install the required packages:

```bash
pip install -U transformers torch bitsandbytes
```

Depending on your environment, you may also need the appropriate PyTorch installation for your CUDA version.

## Running the Project

Open:

```text
ToolCalling.ipynb
```

Run the notebook cells in order.

At the end, the notebook accepts a user question:

```python
question = input("\nAsk: ")

run_agent(question)
```

Example:

```text
Ask: Calculate 25 * 40
```

The system determines that the calculator is required, generates the arguments, validates them, executes the calculator, and sends the result back to the LLM.

## Project Structure

```text
.
├── ToolCalling.ipynb
└── README.md
```

## What This Project Demonstrates

This project focuses on the fundamental mechanics of tool calling:

1. Defining tools
2. Describing tools to an LLM
3. LLM-based tool selectio
