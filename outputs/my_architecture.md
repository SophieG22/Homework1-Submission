# My Agent Architecture
**Created:** 2026-04-12 16:14:55


# MY AGENT ARCHITECTURE

## Component Breakdown

### 1. INPUT HANDLER
Purpose: Capture and preprocess user natural language queries
Implementation:
- Simple CLI or API endpoint (FastAPI)
- Clean input (strip spaces, basic validation)
- Optionally detect intent (aggregation, filtering, grouping)

### 2. CORE WORKFLOW
Purpose: Convert natural language into database results

Steps:
  1. Receive user query
  2. Inject database schema into prompt
  3. Generate SQL query using LLM
  4. Validate SQL (SELECT-only, schema check)
  5. Execute SQL query on database
  6. Format results (table + optional summary)
  7. Return output to user

### 3. TOOL LAYER (if applicable)
Tools needed:
  - Database Connector (SQLAlchemy / sqlite3): Execute SQL queries
  - Schema Loader: Retrieve table/column metadata
  - SQL Validator: Ensure safe queries (block DELETE/UPDATE/DROP)
  - Pandas: Format query results into tables

### 4. LLM INTERACTION
LLM calls needed:

  Call 1: Natural Language ¡ú SQL generation  
    ¡ú Model: GPT-4.1 (or equivalent)  
    ¡ú Output format: JSON  
    Example:
    {
      "sql": "...",
      "explanation": "..."
    }

  Call 2: Result summarization (optional)  
    ¡ú Model: lighter model (GPT-4o-mini or similar)  
    ¡ú Output format: text summary  

### 5. OUTPUT GENERATOR
Output format: JSON + Markdown

Structure:
- SQL query used
- Table result (DataFrame ¡ú Markdown table)
- Optional summary
- Optional metadata (execution time, row count)

Example:
{
  "query": "...",
  "sql": "...",
  "result_table": "...",
  "summary": "..."
}

## Data Flow

User Input 
   ¡ý
Input Handler
   ¡ý
Schema Injection
   ¡ý
LLM (Generate SQL)
   ¡ý
SQL Validation
   ¡ý
Database Execution
   ¡ý
Result Formatting (pandas)
   ¡ý
LLM (Optional Summary)
   ¡ý
Final Output

## File Structure

nl_sql_agent/
  ©À©¤©¤ main.py
  ©À©¤©¤ config.py
  ©À©¤©¤ db/
  ©¦   ©À©¤©¤ connection.py
  ©¦   ©¸©¤©¤ schema.py
  ©À©¤©¤ agents/
  ©¦   ©¸©¤©¤ sql_agent.py
  ©À©¤©¤ prompts/
  ©¦   ©¸©¤©¤ sql_generation_prompt.py
  ©À©¤©¤ utils/
  ©¦   ©À©¤©¤ validator.py
  ©¦   ©¸©¤©¤ formatter.py
  ©¸©¤©¤ outputs/

## Key Prompts

EXTRACTION_PROMPT:
'''
You are a data analyst expert.

Context:
You are given a database schema and a user question.

Schema:
{schema}

Task:
Convert the user's question into a valid SQL query.

Constraints:
- Only generate SELECT queries
- Use only tables and columns from the schema
- Avoid unnecessary complexity
- Include aggregation if needed (SUM, COUNT, etc.)

Output format (JSON):
{
  "sql": "your SQL query",
  "explanation": "brief explanation"
}

User Question:
{user_query}
'''

ANALYSIS_PROMPT:
'''
You are a business analyst.

Given the following query result:
{table}

Summarize key insights in 2-3 sentences.
Focus on trends, comparisons, or anomalies.
'''

## Error Handling

1. Input errors:
   - Empty or unclear query ¡ú return clarification request

2. API failures:
   - Retry LLM call (max 2 retries)
   - Fallback to simpler prompt

3. LLM failures:
   - Validate JSON output
   - If invalid ¡ú re-prompt with stricter instructions

4. Partial results:
   - If SQL runs but result is empty ¡ú return message + possible reasons
   - If schema mismatch ¡ú suggest corrected columns
