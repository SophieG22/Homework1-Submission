
# ARCHITECTURE DESIGN GUIDE

## Component Breakdown

Break your agent into logical components:

1. INPUT HANDLER
   - What: Receives and validates user input
   - How: Parse user query, validate parameters
   - Example: "Get topic and max_papers from user"

2. CORE LOGIC
   - What: The main workflow of your agent
   - How: Orchestrate the steps, make decisions
   - Example: "Loop: search ¡ú filter ¡ú extract ¡ú compare"

3. TOOL LAYER (if using tools)
   - What: Interfaces to external services
   - How: MCP servers, API calls, file operations
   - Example: "arXiv search tool, PDF parser tool"

4. LLM INTERACTION LAYER
   - What: All LLM calls with proper prompting
   - How: CO-STAR prompts, structured outputs, CoT
   - Example: "Extract methodology using structured JSON output"

5. OUTPUT GENERATOR
   - What: Format and present results
   - How: Generate markdown, tables, visualizations
   - Example: "Create comparison table in markdown"

## Data Flow Diagram

Create a flow diagram for your agent:

©°©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©´
©¦ User Input  ©¦
©¦ "Topic: X"  ©¦
©¸©¤©¤©¤©¤©¤©¤©Ð©¤©¤©¤©¤©¤©¤©¼
       ©¦
       ¨‹
©°©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©´
©¦  Search Papers  ©¦  ¡û Tool/API Call
©¦  (arXiv API)    ©¦
©¸©¤©¤©¤©¤©¤©¤©Ð©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¼
       ©¦
       ¨‹
©°©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©´
©¦  Filter Top N   ©¦  ¡û LLM: Relevance check
©¦  Most Relevant  ©¦
©¸©¤©¤©¤©¤©¤©¤©Ð©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¼
       ©¦
       ¨‹
©°©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©´
©¦ Extract Methods ©¦  ¡û LLM: Structured extraction
©¦  (JSON output)  ©¦
©¸©¤©¤©¤©¤©¤©¤©Ð©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¼
       ©¦
       ¨‹
©°©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©´
©¦ Compare Methods ©¦  ¡û LLM: CoT analysis
©¦  (find patterns)©¦
©¸©¤©¤©¤©¤©¤©¤©Ð©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¼
       ©¦
       ¨‹
©°©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©´
©¦ Generate Report ©¦  ¡û Format output
©¦  (Markdown)     ©¦
©¸©¤©¤©¤©¤©¤©¤©Ð©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¼
       ©¦
       ¨‹
©°©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©´
©¦ Return to User  ©¦
©¸©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¤©¼

## Module Structure

Organize your code into modules:

project/
©À©¤©¤ main.py (or main.ipynb)           # Entry point
©À©¤©¤ config.py                          # Configuration
©À©¤©¤ agents/
©¦   ©À©¤©¤ __init__.py
©¦   ©À©¤©¤ research_agent.py             # Main agent class
©¦   ©¸©¤©¤ workflow.py                   # Workflow logic
©À©¤©¤ tools/
©¦   ©À©¤©¤ __init__.py
©¦   ©À©¤©¤ paper_search.py               # Search functionality
©¦   ©À©¤©¤ pdf_parser.py                 # PDF processing
©¦   ©¸©¤©¤ formatter.py                  # Output formatting
©À©¤©¤ prompts/
©¦   ©À©¤©¤ __init__.py
©¦   ©À©¤©¤ extraction.py                 # Extraction prompts
©¦   ©¸©¤©¤ analysis.py                   # Analysis prompts
©¸©¤©¤ outputs/
    ©¸©¤©¤ [generated files]

## Prompt Library

Document your key prompts:

PROMPT_EXTRACT_METHOD = '''
{CO-STAR structure}
Extract the methodology from this paper...
'''

PROMPT_COMPARE = '''
{CO-STAR structure}
Compare these methodologies...
'''

## Error Handling Strategy

Plan for failures:

1. Input Validation
   - Check: Valid topic? Reasonable parameters?
   - Action: Return clear error message

2. API Failures
   - Check: Rate limits? Network issues?
   - Action: Retry with backoff, use cache, fail gracefully

3. LLM Failures
   - Check: Invalid JSON? Poor quality output?
   - Action: Retry with better prompt, use fallback

4. Partial Success
   - Check: Got some but not all results?
   - Action: Return partial results with warning
