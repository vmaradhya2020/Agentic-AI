# GenAI Investment Advisor - Complete Project Documentation

## Table of Contents
1. [Project Overview](#project-overview)
2. [Business Value](#business-value)
3. [Architecture Overview](#architecture-overview)
4. [Technical Components](#technical-components)
5. [How It Works - Step by Step](#how-it-works---step-by-step)
6. [Code Modules Explained](#code-modules-explained)
7. [Flow Diagram](#flow-diagram)
8. [Setup Guide](#setup-guide)
9. [API Reference](#api-reference)

---

## Project Overview

**GenAI Investment Advisor** is an intelligent financial analysis application that automatically fetches stock market data and generates comprehensive investment reports using Artificial Intelligence.

### What Does This App Do?
- **Input**: You enter a stock ticker symbol (e.g., "AAPL" for Apple, "TSLA" for Tesla)
- **Process**: The app automatically discovers available data sources, fetches all relevant information, and analyzes it
- **Output**: You get visual charts and an AI-generated report with insights and outlook

### Target Users
- **Individual Investors**: Want quick analysis of stocks before making decisions
- **Financial Advisors**: Need rapid insights on multiple stocks
- **Students/Learners**: Learning about financial analysis and AI applications
- **Anyone**: Curious about stock market trends without manual research

---

## Business Value

### Problem It Solves
Traditional stock analysis requires:
- Visiting multiple websites (Yahoo Finance, news sites, analyst reports)
- Manually compiling data from different sources
- Spending hours reading and synthesizing information
- Technical knowledge to interpret financial metrics

### Solution Provided
- **One-Click Analysis**: Enter a ticker, get complete report in seconds
- **AI-Powered Insights**: Artificial Intelligence reads all data and provides human-readable summary
- **Visual Representation**: Charts make trends easy to understand at a glance
- **Automated Data Collection**: No need to visit multiple websites

### Market Applications
1. **Personal Finance Apps**: Integrate as a feature for investment recommendations
2. **Trading Platforms**: Enhance user experience with AI insights
3. **Financial Advisory Services**: Scale analyst capabilities
4. **Educational Platforms**: Teach stock analysis with real-time examples

---

## Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER INTERFACE                          │
│                    (Gradio Web Application)                     │
│                                                                 │
│  [Text Input: "AAPL"] → [Analyze Button] → [Results Display]   │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│                      MCP CLIENT (Brain)                         │
│                      (mcp_client.py)                            │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  PHASE 1: Data Aggregation                              │   │
│  │  - Connect to MCP Server                                │   │
│  │  - Discover available tools                             │   │
│  │  - Fetch all data in parallel                           │   │
│  └─────────────────────────────────────────────────────────┘   │
│                             ↓                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  PHASE 2: AI Agent Processing                           │   │
│  │  - Analyze fetched data                                 │   │
│  │  - Generate visualizations                              │   │
│  │  - Create AI summary with Gemini                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                             ↓                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  PHASE 3: Result Rendering                              │   │
│  │  - Combine charts                                       │   │
│  │  - Format final report                                  │   │
│  └─────────────────────────────────────────────────────────┘   │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│                    MCP SERVER (Data Layer)                      │
│                      (mcp_server.py)                            │
│                                                                 │
│  [Tool 1]  [Tool 2]  [Tool 3]  [Tool 4]  [Tool 5]             │
│     ↓         ↓         ↓         ↓         ↓                  │
└─────┬─────────┬─────────┬─────────┬─────────┬──────────────────┘
      │         │         │         │         │
      ↓         ↓         ↓         ↓         ↓
┌─────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│ Yahoo   │ │Finnhub │ │Finnhub │ │Finnhub │ │Finnhub │
│ Finance │ │  API   │ │  API   │ │  API   │ │  API   │
│  (yf)   │ │ Quote  │ │  News  │ │ Trends │ │Earnings│
└─────────┘ └────────┘ └────────┘ └────────┘ └────────┘
```

### What is MCP (Model Context Protocol)?
**MCP** is a standard way for AI applications to discover and use tools/data sources.

**Simple Analogy**:
- Think of MCP Server as a **restaurant menu** - it lists all available dishes (tools)
- MCP Client is the **customer** - it reads the menu and orders multiple dishes
- The waiter (protocol) ensures orders are taken and food is delivered correctly

**Why Use MCP?**
- **Extensibility**: Add new data sources without changing client code
- **Discoverability**: Client automatically finds all available tools
- **Standardization**: Common protocol for AI tool integration

---

## Technical Components

### 1. **MCP Server** (`mcp_server.py`) - The Data Provider

**Purpose**: Exposes financial data as "tools" that can be called by the client

**Technology Stack**:
- `FastMCP`: Framework to create MCP-compliant servers
- `finnhub-python`: SDK for Finnhub financial API
- `yfinance`: Yahoo Finance data library

**Available Tools** (5 total):

#### Tool 1: `get_stock_history(ticker: str)`
```python
Location: mcp_server.py:21-41
Input: Stock ticker (e.g., "AAPL")
Output: Last 7 days of closing prices with dates
Example: {"dates": ["2025-10-17", "2025-10-18", ...], "prices": [150.25, 152.30, ...]}
```
**What it does**: Gets historical price trend to see if stock is going up or down

#### Tool 2: `get_latest_quote(ticker: str)`
```python
Location: mcp_server.py:45-53
Input: Stock ticker
Output: Real-time price data
Example: {"c": 150.25, "h": 151.00, "l": 149.50, "o": 150.00, "pc": 149.75}
         (c=current, h=high, l=low, o=open, pc=previous close)
```
**What it does**: Gets the current stock price and today's trading range

#### Tool 3: `get_company_news(ticker: str)`
```python
Location: mcp_server.py:56-67
Input: Stock ticker
Output: Top 3 recent news articles (last 30 days)
Example: [{"headline": "Apple launches new iPhone", "summary": "...", "url": "..."}]
```
**What it does**: Fetches recent news that might affect stock price

#### Tool 4: `get_recommendation_trends(ticker: str)`
```python
Location: mcp_server.py:70-79
Input: Stock ticker
Output: Analyst recommendations breakdown
Example: {"buy": 15, "hold": 8, "sell": 2, "strongBuy": 10, "strongSell": 0}
```
**What it does**: Shows what professional analysts recommend (buy/sell/hold)

#### Tool 5: `get_earnings_reports(ticker: str)`
```python
Location: mcp_server.py:82-101
Input: Stock ticker
Output: Recent earnings calendar data
Example: [{"date": "2025-01-15", "epsActual": 1.25, "epsEstimate": 1.20}]
```
**What it does**: Gets company's financial performance reports

---

### 2. **MCP Client** (`mcp_client.py`) - The Intelligence Layer

**Purpose**: Orchestrates data fetching, AI analysis, and report generation

**Technology Stack**:
- `google-generativeai`: Google's Gemini AI for text generation
- `mcp`: Client library to communicate with MCP servers
- `gradio`: Web UI framework
- `matplotlib`: Charting library
- `asyncio`: Asynchronous programming for parallel execution

#### Component 2.1: Data Aggregation Engine

```python
Location: mcp_client.py:139-158
```

**What it does**:
1. Connects to MCP Server
2. Asks: "What tools do you have?" (discovers tools dynamically)
3. Calls ALL tools at once in parallel (saves time)
4. Collects all results into one big data package

**Technical Implementation**:
```python
# Discover tools
available_tools = await session.list_tools()

# Parallel execution
tasks = [session.call_tool(name=name, arguments={"ticker": ticker})
         for name in tool_names]
results = await asyncio.gather(*tasks)
```

**Why Parallel?**: Instead of waiting 5 seconds × 5 tools = 25 seconds, all run together in ~5 seconds!

---

#### Component 2.2: Local Rendering Tools

**Tool A: Price Chart Generator**
```python
Location: mcp_client.py:54-72
Function: plot_historical_price_chart(data_json: str)
```
**What it does**: Takes price data and creates a line graph showing 7-day trend

**Technical Details**:
- Uses matplotlib for visualization
- Converts date strings to datetime objects for proper X-axis formatting
- Blue line with circular markers at each day's closing price
- Automatically formats dates as "Oct 17", "Oct 18", etc.

**Business Value**: Visual trends are easier to understand than raw numbers

---

**Tool B: Analyst Recommendations Chart**
```python
Location: mcp_client.py:74-99
Function: plot_analyst_recommendations_chart(data_json: str)
```
**What it does**: Creates a colorful bar chart showing analyst opinions

**Visual Design**:
- Dark Red = Strong Sell (very negative)
- Red = Sell (negative)
- Orange = Hold (neutral)
- Sky Blue = Buy (positive)
- Dark Green = Strong Buy (very positive)

**Business Value**: Instantly see if experts are bullish or bearish on the stock

---

**Tool C: AI Summary Generator**
```python
Location: mcp_client.py:102-132
Function: generate_financial_summary(full_context_json: str)
```
**What it does**: Sends all collected data to Google's Gemini AI with instructions to write a financial report

**AI Prompt Engineering**:
The prompt tells the AI to:
1. Act as a financial analyst
2. Create a "Summary" section (what's happening now)
3. Create an "Outlook" section (what might happen next)
4. Only use provided data, don't make up information
5. Format output in markdown for readability

**Technical Implementation**:
```python
model = genai.GenerativeModel('gemini-2.5-flash-lite-preview-06-17')
response = await model.generate_content_async(prompt)
```

**Business Value**: AI reads complex data and explains it in simple English

---

#### Component 2.3: AI Agent Orchestration

```python
Location: mcp_client.py:162-224
```

**What is an AI Agent?**
An AI agent is a program that:
- Receives a goal ("analyze this stock")
- Has access to tools (chart generators, summary writer)
- Decides which tools to use based on available data
- Executes tools autonomously
- Returns results

**How This Agent Works**:

**Step 1: Agent Configuration**
```python
agent = genai.GenerativeModel(
    'gemini-2.5-flash-lite-preview-06-17',
    tools=[plot_historical_price_chart,
           plot_analyst_recommendations_chart,
           generate_financial_summary],
    generation_config=GenerationConfig(temperature=0.0)
)
```
- `temperature=0.0`: Makes AI deterministic (same input = same output)

**Step 2: Agent Prompt**
```python
prompt = f"""
You are an expert financial analyst.
Here is data for ticker '{ticker}':
{full_context_json_str}

Your task:
1. Call plot_historical_price_chart if history data exists
2. Call plot_analyst_recommendations_chart if trends data exists
3. Call generate_financial_summary with complete context
"""
```

**Step 3: Agent Execution**
```python
response = await agent.generate_content_async(prompt)
```
The AI analyzes the data and decides which tools to call

**Step 4: Tool Execution**
```python
for call in tool_calls:
    if is_async(tool):
        async_tasks.append(tool(**args))
    else:
        sync_results.append(tool(**args))
```
Executes the tools the AI decided to use

**Step 5: Fallback Safety**
```python
if "generate_financial_summary" not in called_tool_names:
    async_tasks.append(generate_financial_summary(full_context_json_str))
```
If AI forgets to call summary tool, we force it to ensure users always get a report

**Business Value**: Agent adapts to available data - if news is missing, it still works with price data

---

#### Component 2.4: Result Renderer

```python
Location: mcp_client.py:226-274
```

**What it does**: Combines multiple charts into one image and formats the final report

**Technical Process**:

1. **Separate Results by Type**:
```python
figures = [res for res in tool_results if isinstance(res, plt.Figure)]
summaries = [res for res in tool_results if isinstance(res, str)]
```

2. **Combine Multiple Charts**:
```python
combined_fig, axes = plt.subplots(len(figures), 1, figsize=(10, 5*len(figures)))
```
Creates a vertical stack of charts

3. **Graceful Degradation**:
```python
final_summary = summaries if summaries else "### No data available"
```
If no data, shows friendly message instead of crashing

**Business Value**: Professional-looking output that handles edge cases

---

### 3. **User Interface** (`mcp_client.py` lines 281-297)

**Technology**: Gradio - Python library for creating web UIs

**Components**:
```python
ticker_input = gr.Textbox(label="Ticker Symbol")  # Text input
analyze_button = gr.Button("Analyze")              # Click button
status_output = gr.Textbox(label="Status")         # Progress updates
plot_output = gr.Plot(label="Visual Report")       # Charts display
summary_output = gr.Markdown(label="Summary")      # AI text report
```

**User Flow**:
1. User types "AAPL" in text box
2. Clicks "Analyze" button
3. Status shows "Analyzing..."
4. Charts appear
5. AI summary appears below charts

**Technical**: Event-driven programming
```python
analyze_button.click(
    analyze_and_plot,           # Function to call
    inputs=[ticker_input],      # What data to send
    outputs=[plot, status, summary]  # What to update
)
```

---

## How It Works - Step by Step

### Scenario: User Analyzes Apple Stock (AAPL)

#### **Step 1: User Input** (0 seconds)
```
User types: "AAPL"
User clicks: "Analyze" button
```

#### **Step 2: Client Startup** (0.5 seconds)
```python
# Location: mcp_client.py:142-144
server_params = StdioServerParameters(command="python", args=["mcp_server.py"])
read_pipe, write_pipe = await stack.enter_async_context(stdio_client(server_params))
session = await stack.enter_async_context(ClientSession(read_pipe, write_pipe))
```
**What happens**:
- Client starts MCP server as a subprocess
- Creates communication pipes (like phone lines between processes)
- Establishes session (handshake)

**Analogy**: Like calling a restaurant and establishing the phone connection

---

#### **Step 3: Tool Discovery** (0.5 seconds)
```python
# Location: mcp_client.py:147-148
available_tools = await session.list_tools()
tool_names = [tool.name for tool in available_tools.tools]
```
**What happens**:
- Client asks server: "What can you do?"
- Server responds with list of 5 tools
- Client now knows what functions it can call

**Analogy**: Getting the restaurant menu to see what's available

**Result**:
```
tool_names = [
    "get_stock_history",
    "get_latest_quote",
    "get_company_news",
    "get_recommendation_trends",
    "get_earnings_reports"
]
```

---

#### **Step 4: Parallel Data Fetching** (3-5 seconds)
```python
# Location: mcp_client.py:150-151
tasks = [session.call_tool(name=name, arguments={"ticker": ticker})
         for name in tool_names]
results = await asyncio.gather(*tasks, return_exceptions=True)
```

**What happens**:
All 5 API calls execute simultaneously:

```
Timeline:
0.0s → call_tool("get_stock_history", {"ticker": "AAPL"})    → yfinance API
0.0s → call_tool("get_latest_quote", {"ticker": "AAPL"})     → Finnhub API
0.0s → call_tool("get_company_news", {"ticker": "AAPL"})     → Finnhub API
0.0s → call_tool("get_recommendation_trends", ...)           → Finnhub API
0.0s → call_tool("get_earnings_reports", ...)                → Finnhub API

3.5s ← All responses received
```

**Why Fast?**: Parallel execution instead of sequential (5 tools × 3s = 15s → 3.5s)

**Result Package**:
```json
{
  "get_stock_history": {"dates": [...], "prices": [...]},
  "get_latest_quote": {"c": 150.25, "h": 151, ...},
  "get_company_news": [{"headline": "Apple announces...", ...}],
  "get_recommendation_trends": {"buy": 15, "hold": 8, ...},
  "get_earnings_reports": [{"date": "2025-01-15", ...}]
}
```

---

#### **Step 5: Agent Initialization** (0.1 seconds)
```python
# Location: mcp_client.py:167-171
agent = genai.GenerativeModel(
    'gemini-2.5-flash-lite-preview-06-17',
    tools=local_tools,
    generation_config=GenerationConfig(temperature=0.0)
)
```
**What happens**: Creates AI agent with access to 3 rendering tools

---

#### **Step 6: Agent Analysis** (2-3 seconds)
```python
# Location: mcp_client.py:189
response = await agent.generate_content_async(prompt)
```

**What happens**:
1. Agent receives all data + instructions
2. Agent reads the data context
3. Agent decides:
   - "History data exists → call plot_historical_price_chart"
   - "Trends data exists → call plot_analyst_recommendations_chart"
   - "Need summary → call generate_financial_summary"
4. Returns list of function calls to make

**AI Decision Making Example**:
```json
[
  {
    "function_call": {
      "name": "plot_historical_price_chart",
      "args": {"data_json": "{\"dates\": [...], \"prices\": [...]}"}
    }
  },
  {
    "function_call": {
      "name": "plot_analyst_recommendations_chart",
      "args": {"data_json": "{\"buy\": 15, ...}"}
    }
  },
  {
    "function_call": {
      "name": "generate_financial_summary",
      "args": {"full_context_json": "{...all data...}"}
    }
  }
]
```

---

#### **Step 7: Tool Execution** (1-2 seconds)
```python
# Location: mcp_client.py:199-223
for call in tool_calls:
    if is_async(tool):
        async_tasks.append(tool(**args))
    else:
        sync_results.append(tool(**args))

async_results = await asyncio.gather(*async_tasks)
```

**Parallel Execution**:
```
0.0s → plot_historical_price_chart()    → Creates line chart (sync, 0.5s)
0.5s → plot_analyst_recommendations()   → Creates bar chart (sync, 0.5s)
1.0s → generate_financial_summary()     → Calls Gemini AI (async, 2s)

3.0s ← All results ready
```

**Chart 1 Output** (Price History):
```
📈 7-Day Price History
150 |                    ●
    |               ●
145 |          ●
    |     ●
140 | ●
    +------------------
    Oct 17 ... Oct 23
```

**Chart 2 Output** (Analyst Recommendations):
```
📊 Analyst Recommendations
15 |     ████
10 | ████████████
 5 | ████████████████
 0 +────────────────
   SS  S   H   B  SB
```

**Summary Output**:
```markdown
### Summary
Apple (AAPL) closed at $150.25, showing a 3.2% increase over the past week.
Recent news highlights the launch of new AI features...

### Overall Outlook
Based on strong analyst recommendations (15 buy, 10 strong buy)
and positive news sentiment, the outlook remains bullish...
```

---

#### **Step 8: Result Rendering** (0.5 seconds)
```python
# Location: mcp_client.py:232-263
if len(figures) > 1:
    combined_fig, axes = plt.subplots(len(figures), 1)
    # Copy charts into combined figure
```

**What happens**:
- Combines 2 charts vertically into one image
- Formats summary text as markdown
- Prepares final output package

---

#### **Step 9: Display to User** (0.1 seconds)
```python
# Location: mcp_client.py:274
return final_plot, f"Analysis complete for {ticker.upper()}.", final_summary
```

**What user sees**:
```
Status: ✅ Analysis complete for AAPL.

[CHARTS DISPLAYED HERE]

### Summary
Apple (AAPL) closed at $150.25...

### Overall Outlook
Based on strong analyst recommendations...
```

**Total Time**: ~7-8 seconds from button click to results

---

## Code Modules Explained

### Module 1: Configuration & Setup

#### File: `.env` (Environment Variables)
```bash
FINNHUB_API_KEY=xxxxx
GEMINI_API_KEY=xxxxx
```
**Purpose**: Stores secret API keys securely (not in code)
**Why Important**: Prevents exposing keys in version control (GitHub)

#### File: `pyproject.toml` (Dependency Configuration)
```toml
[project]
name = "app"
requires-python = ">=3.12"
dependencies = [
    "finnhub-python>=2.4.24",
    "google-generativeai>=0.8.5",
    "mcp[cli]==1.10.1",
    "gradio==5.36.2",
    ...
]
```
**Purpose**: Defines project metadata and required libraries
**Why Important**: Ensures everyone uses compatible library versions

---

### Module 2: Error Handling & Resilience

#### Error Handling Pattern (mcp_server.py)
```python
try:
    return finnhub_client.quote(ticker)
except Exception as e:
    return {"error": str(e)}
```
**What it does**: If API fails, return error message instead of crashing
**Business Value**: App continues working even if one data source fails

#### Error Handling Pattern (mcp_client.py)
```python
for name, res in zip(tool_names, results):
    if isinstance(res, Exception):
        full_context[name] = json.dumps({"error": f"Failed to call tool: {res}"})
    else:
        full_context[name] = parse_json_response(res)
```
**What it does**: Collects errors but continues processing other results
**Business Value**: Partial data is better than no data

---

### Module 3: JSON Parsing Utility

```python
# Location: mcp_client.py:29-48
def parse_json_response(res: CallToolResult) -> str:
    """Parse MCP tool response and convert to JSON string"""
    try:
        content = res.content
        if len(content) == 1:
            data = json.loads(content[0].text)
        else:
            data = [json.loads(item.text) for item in content]
        return json.dumps(data)
    except json.JSONDecodeError as e:
        return json.dumps({"error": f"JSON decode error: {e}"})
```

**What it does**: Safely converts MCP responses to JSON format
**Why Important**: MCP protocol uses specific response structure that needs parsing
**Error Handling**: Returns error JSON instead of crashing on bad data

---

### Module 4: Asynchronous Programming

**What is Async/Await?**

**Synchronous (Sequential)**:
```python
# Total time: 15 seconds
result1 = fetch_data_1()  # 5 seconds
result2 = fetch_data_2()  # 5 seconds
result3 = fetch_data_3()  # 5 seconds
```

**Asynchronous (Parallel)**:
```python
# Total time: 5 seconds
results = await asyncio.gather(
    fetch_data_1(),  # All start together
    fetch_data_2(),
    fetch_data_3()
)
```

**Implementation in This Project**:
```python
# Location: mcp_client.py:150-151
tasks = [session.call_tool(name=name, arguments={"ticker": ticker})
         for name in tool_names]
results = await asyncio.gather(*tasks, return_exceptions=True)
```

**Business Value**: 3x faster response time = better user experience

---

## Flow Diagram

### High-Level User Journey Flow

```
┌─────────────┐
│   START     │
│  User opens │
│     app     │
└──────┬──────┘
       │
       ↓
┌─────────────────────┐
│  USER INTERFACE     │
│  ┌───────────────┐  │
│  │ Enter Ticker: │  │
│  │   [AAPL]      │  │
│  └───────────────┘  │
│  ┌───────────────┐  │
│  │ [  Analyze  ] │  │
│  └───────────────┘  │
└──────┬──────────────┘
       │
       │ Button Click Event
       ↓
┌──────────────────────────────────────────────┐
│  PHASE 1: DATA AGGREGATION                   │
│  ┌────────────────────────────────────────┐  │
│  │ 1. Start MCP Server Process            │  │
│  │ 2. Initialize Client-Server Session    │  │
│  │ 3. Discover Available Tools             │  │
│  └────────────────────────────────────────┘  │
│                    ↓                          │
│  ┌────────────────────────────────────────┐  │
│  │ 4. Parallel Tool Execution              │  │
│  │                                          │  │
│  │    ┌──────────────────┐                 │  │
│  │    │ Tool 1: History  │───┐             │  │
│  │    └──────────────────┘   │             │  │
│  │    ┌──────────────────┐   │             │  │
│  │    │ Tool 2: Quote    │───┤             │  │
│  │    └──────────────────┘   │             │  │
│  │    ┌──────────────────┐   ├─→ gather()  │  │
│  │    │ Tool 3: News     │───┤   results   │  │
│  │    └──────────────────┘   │             │  │
│  │    ┌──────────────────┐   │             │  │
│  │    │ Tool 4: Trends   │───┤             │  │
│  │    └──────────────────┘   │             │  │
│  │    ┌──────────────────┐   │             │  │
│  │    │ Tool 5: Earnings │───┘             │  │
│  │    └──────────────────┘                 │  │
│  └────────────────────────────────────────┘  │
│                    ↓                          │
│  ┌────────────────────────────────────────┐  │
│  │ 5. Aggregate All Results into Context  │  │
│  │    full_context = {                    │  │
│  │      "get_stock_history": {...},       │  │
│  │      "get_latest_quote": {...},        │  │
│  │      ...                                │  │
│  │    }                                    │  │
│  └────────────────────────────────────────┘  │
└──────┬───────────────────────────────────────┘
       │
       │ Data Ready
       ↓
┌──────────────────────────────────────────────┐
│  PHASE 2: AI AGENT PROCESSING                │
│  ┌────────────────────────────────────────┐  │
│  │ 1. Initialize Gemini Agent             │  │
│  │    - Register 3 local tools            │  │
│  │    - Set temperature = 0.0             │  │
│  └────────────────────────────────────────┘  │
│                    ↓                          │
│  ┌────────────────────────────────────────┐  │
│  │ 2. Send Prompt to Agent                │  │
│  │    "You are a financial analyst..."    │  │
│  │    + full_context_json                 │  │
│  └────────────────────────────────────────┘  │
│                    ↓                          │
│  ┌────────────────────────────────────────┐  │
│  │ 3. Agent Analyzes Data                 │  │
│  │    ┌──────────────────────────────┐    │  │
│  │    │ Gemini AI Brain              │    │  │
│  │    │                              │    │  │
│  │    │ Reads: History? ✓            │    │  │
│  │    │ Reads: Trends? ✓             │    │  │
│  │    │ Reads: News? ✓               │    │  │
│  │    │                              │    │  │
│  │    │ Decision:                    │    │  │
│  │    │ - Create price chart         │    │  │
│  │    │ - Create recommendations bar │    │  │
│  │    │ - Generate summary           │    │  │
│  │    └──────────────────────────────┘    │  │
│  └────────────────────────────────────────┘  │
│                    ↓                          │
│  ┌────────────────────────────────────────┐  │
│  │ 4. Execute Local Tools                 │  │
│  │                                          │  │
│  │  ┌──────────────────────────────────┐  │  │
│  │  │ plot_historical_price_chart()    │  │  │
│  │  │ → Returns matplotlib Figure       │  │  │
│  │  └──────────────────────────────────┘  │  │
│  │                                          │  │
│  │  ┌──────────────────────────────────┐  │  │
│  │  │ plot_analyst_recommendations()   │  │  │
│  │  │ → Returns matplotlib Figure       │  │  │
│  │  └──────────────────────────────────┘  │  │
│  │                                          │  │
│  │  ┌──────────────────────────────────┐  │  │
│  │  │ generate_financial_summary()     │  │  │
│  │  │ → Calls Gemini AI                │  │  │
│  │  │ → Returns markdown text           │  │  │
│  │  └──────────────────────────────────┘  │  │
│  └────────────────────────────────────────┘  │
└──────┬───────────────────────────────────────┘
       │
       │ Tools Executed
       ↓
┌──────────────────────────────────────────────┐
│  PHASE 3: RESULT RENDERING                   │
│  ┌────────────────────────────────────────┐  │
│  │ 1. Separate Results by Type            │  │
│  │    - figures = [chart1, chart2]        │  │
│  │    - summaries = [text_report]         │  │
│  └────────────────────────────────────────┘  │
│                    ↓                          │
│  ┌────────────────────────────────────────┐  │
│  │ 2. Combine Multiple Charts             │  │
│  │    ┌────────────────────────────┐      │  │
│  │    │  Chart 1: Price History    │      │  │
│  │    ├────────────────────────────┤      │  │
│  │    │  Chart 2: Recommendations  │      │  │
│  │    └────────────────────────────┘      │  │
│  │    → Single combined figure            │  │
│  └────────────────────────────────────────┘  │
│                    ↓                          │
│  ┌────────────────────────────────────────┐  │
│  │ 3. Format Summary Text                 │  │
│  │    - Apply markdown formatting         │  │
│  │    - Handle missing data gracefully    │  │
│  └────────────────────────────────────────┘  │
│                    ↓                          │
│  ┌────────────────────────────────────────┐  │
│  │ 4. Return Final Output                 │  │
│  │    - final_plot (image)                │  │
│  │    - status_message (text)             │  │
│  │    - final_summary (markdown)          │  │
│  └────────────────────────────────────────┘  │
└──────┬───────────────────────────────────────┘
       │
       │ Results Ready
       ↓
┌─────────────────────────────────────────┐
│  GRADIO UI UPDATES                      │
│  ┌───────────────────────────────────┐  │
│  │ Status: ✅ Analysis complete       │  │
│  └───────────────────────────────────┘  │
│  ┌───────────────────────────────────┐  │
│  │         [CHART IMAGE]             │  │
│  │  ────────────────────────────────  │  │
│  │   📈 7-Day Price History          │  │
│  │  ────────────────────────────────  │  │
│  │   📊 Analyst Recommendations      │  │
│  └───────────────────────────────────┘  │
│  ┌───────────────────────────────────┐  │
│  │  ### Summary                      │  │
│  │  Apple (AAPL) closed at $150...   │  │
│  │                                    │  │
│  │  ### Overall Outlook              │  │
│  │  Based on strong analyst...       │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
       │
       ↓
┌─────────────┐
│     END     │
│ User views  │
│   report    │
└─────────────┘
```

---

### Detailed Technical Flow - Data Path

```
┌──────────────┐
│ User Input:  │
│   "AAPL"     │
└──────┬───────┘
       │
       ↓
┌─────────────────────────────────────────┐
│  mcp_client.py:analyze_and_plot()       │
│  Entry point function                   │
└──────┬──────────────────────────────────┘
       │
       ├─────────────────────────────────┐
       │                                 │
       ↓                                 ↓
┌──────────────────┐           ┌─────────────────┐
│  Initialize MCP  │           │  Validate Input │
│  Client Session  │           │  ticker != ""   │
└──────┬───────────┘           └─────────────────┘
       │
       ↓
┌─────────────────────────────────────────────┐
│  session.list_tools()                       │
│                                             │
│  Client → Server: "What tools available?"   │
│  Server → Client: ["get_stock_history",... │
└──────┬──────────────────────────────────────┘
       │
       ↓
┌──────────────────────────────────────────────────┐
│  asyncio.gather(*tasks)                          │
│                                                  │
│  ┌────────────────────────────────────────────┐ │
│  │ Task 1: call_tool("get_stock_history")    │ │
│  │         ↓                                  │ │
│  │   mcp_server.py:get_stock_history()       │ │
│  │         ↓                                  │ │
│  │   stock = yf.Ticker("AAPL")               │ │
│  │   history = stock.history(period="7d")    │ │
│  │         ↓                                  │ │
│  │   return {"dates": [...], "prices": [...]}│ │
│  └────────────────────────────────────────────┘ │
│                                                  │
│  ┌────────────────────────────────────────────┐ │
│  │ Task 2: call_tool("get_latest_quote")     │ │
│  │         ↓                                  │ │
│  │   mcp_server.py:get_latest_quote()        │ │
│  │         ↓                                  │ │
│  │   finnhub_client.quote("AAPL")            │ │
│  │         ↓                                  │ │
│  │   return {"c": 150.25, "h": 151, ...}     │ │
│  └────────────────────────────────────────────┘ │
│                                                  │
│  [...Tasks 3, 4, 5 execute in parallel...]      │
└──────┬───────────────────────────────────────────┘
       │
       ↓
┌──────────────────────────────────────┐
│  Aggregate Results                   │
│  full_context = {                    │
│    "get_stock_history": {...},       │
│    "get_latest_quote": {...},        │
│    ...                                │
│  }                                    │
└──────┬───────────────────────────────┘
       │
       ↓
┌─────────────────────────────────────────────┐
│  agent.generate_content_async(prompt)       │
│                                             │
│  Sends to: Gemini API                       │
│  ┌───────────────────────────────────────┐ │
│  │ Gemini AI Processing                  │ │
│  │                                        │ │
│  │ 1. Parse context JSON                 │ │
│  │ 2. Identify available data            │ │
│  │ 3. Decide which tools to call         │ │
│  │ 4. Generate tool call instructions    │ │
│  └───────────────────────────────────────┘ │
│                                             │
│  Returns:                                   │
│  [                                          │
│    {function: "plot_historical...", ...},   │
│    {function: "plot_analyst...", ...},      │
│    {function: "generate_summary", ...}      │
│  ]                                          │
└──────┬──────────────────────────────────────┘
       │
       ↓
┌──────────────────────────────────────────────┐
│  Execute Tools Locally                       │
│                                              │
│  ┌────────────────────────────────────────┐ │
│  │ plot_historical_price_chart(data)      │ │
│  │  ↓                                      │ │
│  │ dates = ["2025-10-17", ...]            │ │
│  │ prices = [145.5, 148.2, 150.25]        │ │
│  │  ↓                                      │ │
│  │ fig, ax = plt.subplots()               │ │
│  │ ax.plot(dates, prices)                 │ │
│  │  ↓                                      │ │
│  │ return fig                             │ │
│  └────────────────────────────────────────┘ │
│                                              │
│  ┌────────────────────────────────────────┐ │
│  │ generate_financial_summary(context)    │ │
│  │  ↓                                      │ │
│  │ prompt = "You are analyst... {data}"   │ │
│  │  ↓                                      │ │
│  │ Gemini API call                        │ │
│  │  ↓                                      │ │
│  │ return "### Summary\nApple (AAPL)..."  │ │
│  └────────────────────────────────────────┘ │
└──────┬───────────────────────────────────────┘
       │
       ↓
┌──────────────────────────────────────┐
│  Combine Charts                      │
│  ┌────────────────────────────────┐  │
│  │ ╔══════════════════════════╗   │  │
│  │ ║ 7-Day Price History      ║   │  │
│  │ ║ 150├───────────●          ║   │  │
│  │ ║ 145├──────●               ║   │  │
│  │ ║ 140├──●                   ║   │  │
│  │ ╠══════════════════════════╣   │  │
│  │ ║ Analyst Recommendations  ║   │  │
│  │ ║  15├  ████               ║   │  │
│  │ ║  10├████████             ║   │  │
│  │ ║   0├───────────────────  ║   │  │
│  │ ╚══════════════════════════╝   │  │
│  └────────────────────────────────┘  │
└──────┬───────────────────────────────┘
       │
       ↓
┌──────────────────────────────────────┐
│  Return to Gradio                    │
│  (final_plot, status, summary)       │
└──────┬───────────────────────────────┘
       │
       ↓
┌──────────────────────────────────────┐
│  Gradio UI Render                    │
│  - Updates plot_output widget        │
│  - Updates status_output widget      │
│  - Updates summary_output widget     │
└──────┬───────────────────────────────┘
       │
       ↓
┌──────────────┐
│ User sees    │
│ final report │
└──────────────┘
```

---

## Setup Guide

### Prerequisites
- **Python**: Version 3.12 or higher
- **API Keys**:
  - Finnhub account (free tier available)
  - Google AI Studio account (Gemini API)

### Installation Steps

#### Step 1: Fork & Clone Repository
```bash
# Fork repository on GitHub first
git clone https://github.com/YOUR_USERNAME/ik-genai-w3-mcp-master.git
cd ik-genai-w3-mcp-master
```

#### Step 2: Get API Keys

**Finnhub API Key**:
1. Go to https://finnhub.io/
2. Click "Get Free API Key"
3. Sign up with email
4. Copy your API key

**Gemini API Key**:
1. Go to https://aistudio.google.com/app/apikey
2. Click "Create API Key"
3. Copy your API key

#### Step 3: Create Environment File
```bash
# Create .env file in project root
touch .env

# Add these lines (replace with your keys):
FINNHUB_API_KEY=your_finnhub_key_here
GEMINI_API_KEY=your_gemini_key_here
```

#### Step 4: Setup Python Environment
```bash
# Install uv (Python package manager)
pip install uv

# Create virtual environment
uv venv

# Activate virtual environment
# On Windows:
.venv\Scripts\activate
# On macOS/Linux:
source .venv/bin/activate
```

#### Step 5: Install Dependencies
```bash
uv sync
```

#### Step 6: Run Application
```bash
uv run mcp_client.py
```

#### Step 7: Access Web Interface
```
1. Terminal will show: "Running on local URL: http://127.0.0.1:7860"
2. Click the URL or open browser to that address
3. Start analyzing stocks!
```

---

### Troubleshooting

**Problem**: `ModuleNotFoundError: No module named 'mcp'`
**Solution**: Run `uv sync` to install dependencies

**Problem**: `ValueError: Please set the FINNHUB_API_KEY environment variable`
**Solution**: Check `.env` file exists and contains correct keys

**Problem**: Charts not displaying
**Solution**: Ensure matplotlib is installed: `uv pip install matplotlib`

**Problem**: Slow response times
**Solution**: Check internet connection; APIs may be rate-limited on free tier

---

## API Reference

### MCP Server Tools API

#### `get_stock_history(ticker: str) -> dict`
**Description**: Fetches 7-day historical closing prices

**Parameters**:
- `ticker` (string, required): Stock ticker symbol (e.g., "AAPL")

**Returns**:
```json
{
  "dates": ["2025-10-17", "2025-10-18", "2025-10-19", ...],
  "prices": [145.50, 147.25, 148.75, ...]
}
```

**Error Response**:
```json
{
  "error": "no history found for this ticker INVALID"
}
```

**Example Usage**:
```python
result = await session.call_tool(
    name="get_stock_history",
    arguments={"ticker": "AAPL"}
)
```

---

#### `get_latest_quote(ticker: str) -> dict`
**Description**: Fetches current real-time quote

**Parameters**:
- `ticker` (string, required): Stock ticker symbol

**Returns**:
```json
{
  "c": 150.25,    // Current price
  "h": 151.00,    // High of the day
  "l": 149.50,    // Low of the day
  "o": 150.00,    // Open price
  "pc": 149.75,   // Previous close
  "t": 1697558400 // Timestamp
}
```

**External API**: Finnhub `/quote` endpoint

---

#### `get_company_news(ticker: str) -> list`
**Description**: Fetches recent news articles (last 30 days, top 3)

**Parameters**:
- `ticker` (string, required): Stock ticker symbol

**Returns**:
```json
[
  {
    "category": "company news",
    "datetime": 1697558400,
    "headline": "Apple Announces New AI Features",
    "id": 123456,
    "image": "https://...",
    "related": "AAPL",
    "source": "Reuters",
    "summary": "Apple Inc. announced...",
    "url": "https://..."
  },
  ...
]
```

**External API**: Finnhub `/company-news` endpoint

---

#### `get_recommendation_trends(ticker: str) -> list`
**Description**: Fetches analyst recommendation trends

**Parameters**:
- `ticker` (string, required): Stock ticker symbol

**Returns**:
```json
[
  {
    "buy": 15,
    "hold": 8,
    "period": "2025-10-01",
    "sell": 2,
    "strongBuy": 10,
    "strongSell": 0,
    "symbol": "AAPL"
  },
  ...
]
```

**External API**: Finnhub `/stock/recommendation` endpoint

---

#### `get_earnings_reports(ticker: str) -> list`
**Description**: Fetches recent earnings calendar data

**Parameters**:
- `ticker` (string, required): Stock ticker symbol

**Returns**:
```json
[
  {
    "date": "2025-01-15",
    "epsActual": 1.25,
    "epsEstimate": 1.20,
    "hour": "amc",
    "quarter": 4,
    "revenueActual": 95000000000,
    "revenueEstimate": 94000000000,
    "symbol": "AAPL",
    "year": 2024
  },
  ...
]
```

**External API**: Finnhub `/calendar/earnings` endpoint

---

### Local Rendering Tools API

#### `plot_historical_price_chart(data_json: str) -> plt.Figure`
**Description**: Creates line chart from price history data

**Parameters**:
- `data_json` (string, required): JSON string with dates and prices

**Expected Input Format**:
```json
{
  "dates": ["2025-10-17", "2025-10-18", ...],
  "prices": [145.50, 147.25, ...]
}
```

**Returns**: matplotlib Figure object with line chart

**Error Handling**: Returns `None` if data invalid or missing

---

#### `plot_analyst_recommendations_chart(data_json: str) -> plt.Figure`
**Description**: Creates bar chart from analyst recommendations

**Parameters**:
- `data_json` (string, required): JSON string with recommendation counts

**Expected Input Format**:
```json
{
  "strongSell": 0,
  "sell": 2,
  "hold": 8,
  "buy": 15,
  "strongBuy": 10,
  "period": "2025-10-01"
}
```

**Returns**: matplotlib Figure object with bar chart

**Color Scheme**:
- Strong Sell: Dark Red (#8B0000)
- Sell: Red (#FF0000)
- Hold: Orange (#FFA500)
- Buy: Sky Blue (#87CEEB)
- Strong Buy: Dark Green (#006400)

---

#### `generate_financial_summary(full_context_json: str) -> str`
**Description**: Uses Gemini AI to generate report summary

**Parameters**:
- `full_context_json` (string, required): Complete data context as JSON

**Expected Input Format**:
```json
{
  "get_stock_history": {"dates": [...], "prices": [...]},
  "get_latest_quote": {"c": 150.25, ...},
  "get_company_news": [...],
  "get_recommendation_trends": [...],
  "get_earnings_reports": [...]
}
```

**Returns**: Markdown-formatted string with two sections:
```markdown
### Summary
Brief overview of current situation...

### Overall Outlook
Forward-looking statement based on data...
```

**AI Model**: `gemini-2.5-flash-lite-preview-06-17`

---

## Appendix: Key Concepts Explained

### What is an API?
**API** = Application Programming Interface

**Simple Explanation**: A menu of functions you can call to get data

**Real-World Analogy**:
- Restaurant menu = API
- Menu items = API endpoints
- Ordering food = Making API call
- Food delivered to table = API response

**Example**:
```python
# Instead of scraping website:
# 1. Open browser
# 2. Navigate to finnhub.io
# 3. Search for AAPL
# 4. Copy data manually

# Use API (one line):
finnhub_client.quote("AAPL")  # Returns data instantly
```

---

### What is Async Programming?
**Async** = Doing multiple things at the same time

**Synchronous (Normal)**:
```
Task 1: ████████ (5 seconds)
        Task 2: ████████ (5 seconds)
                Task 3: ████████ (5 seconds)
Total: 15 seconds
```

**Asynchronous**:
```
Task 1: ████████ (5 seconds)
Task 2: ████████ (5 seconds)
Task 3: ████████ (5 seconds)
Total: 5 seconds (all happen together)
```

**When to Use**:
- I/O-bound tasks (API calls, file reads)
- Multiple independent operations
- Improving user experience (faster response)

---

### What is JSON?
**JSON** = JavaScript Object Notation (data format)

**Simple Explanation**: A way to structure data as text

**Example**:
```json
{
  "name": "Apple Inc.",
  "ticker": "AAPL",
  "price": 150.25,
  "employees": 164000,
  "founded": "1976-04-01"
}
```

**Why Use JSON?**:
- Easy for humans to read
- Easy for computers to parse
- Supported by all programming languages
- Standard for web APIs

---

### What is AI Agent?
**AI Agent** = Program that uses AI to make decisions and take actions

**Components**:
1. **Goal**: What to achieve (e.g., "analyze this stock")
2. **Tools**: Functions it can call (e.g., create charts)
3. **Decision Engine**: AI that decides which tools to use
4. **Execution**: Actually runs the chosen tools

**Example Flow**:
```
User: "Analyze AAPL"
  ↓
Agent receives data about AAPL
  ↓
Agent thinks: "I have price history → use chart tool"
              "I have news → include in summary"
  ↓
Agent calls: plot_chart(), generate_summary()
  ↓
Returns complete report
```

---

### What is MCP (Model Context Protocol)?
**MCP** = Standard for AI tools to discover and use data sources

**Problem It Solves**:
- Without MCP: Hardcode every data source
- With MCP: Automatically discover available tools

**Analogy**:
- **Without MCP**: Going to restaurant, menu is in your head, must remember all dishes
- **With MCP**: Restaurant gives you menu, you see what's available

**Benefits**:
1. **Extensibility**: Add new tools without changing client
2. **Discoverability**: Client finds tools automatically
3. **Standardization**: All tools use same communication protocol

---

## Conclusion

This **GenAI Investment Advisor** demonstrates modern AI application architecture:

1. **Modular Design**: Separate data layer (MCP server) from logic layer (MCP client)
2. **AI-Powered**: Uses Google Gemini for intelligent analysis and report generation
3. **User-Friendly**: Simple web interface hides complex backend
4. **Scalable**: Easy to add new data sources via MCP protocol
5. **Fast**: Parallel execution ensures quick response times
6. **Resilient**: Comprehensive error handling for production-ready reliability

**Educational Value**:
- Learn AI agent patterns
- Understand async programming
- Experience MCP protocol
- Practice API integration
- Build real-world applications

**Commercial Potential**:
- Integrate into trading platforms
- Offer as SaaS product
- License to financial advisors
- Educational tool for finance students

---

*Generated with detailed technical documentation for ik-genai-w3-mcp-master project*
