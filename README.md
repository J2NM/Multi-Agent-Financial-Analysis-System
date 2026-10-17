# Multy-Agent-Financial-Analysis-System
This project will consist in a system to build a real-world financial analysis system powered by agentic AI. Unlike traditional pipelines, agentic systems can reason, plan, and act, coordinating multiple specialized LLM agents to handle complex financial tasks end-to-end.
The primary deliverable is [Multi_agent_system_final.ipynb](Multi_agent_system_final.ipynb). It contains the implementation, a live research run, a routing evaluation, and sections for documenting results.

## How it works

```text
Ticker + research question
    - Planner/Coordinator Agent: select tasks and data windows
    - Yahoo Finance and NewsAPI: retrieve current inputs
    - News Analysis Agent: classify articles and route relevant facts
    - Market and Financial/Earnings Agents: calculate and interpret metrics
    - Specialist self-review -> combined research report
    - Evaluator–Optimizer Agent: score, request targeted revisions, reevaluate
```

The planner chooses which agents run and their order. News is retrieved and routed first when selected so market and earnings articles can inform the corresponding specialists. Python calculates numeric metrics; the LLM interprets them. The evaluator scores accuracy, completeness, evidence, and clarity. The coordinator stops when the report passes, evaluation fails, or two revisions have been attempted.

| Agent | Responsibility |
| --- | --- |
| Planner/Coordinator | Builds the plan, calls data tools and agents, and manages revisions. |
| Market Analysis | Examines returns, volatility, moving averages, drawdown, and volume. |
| Financial/Earnings | Examines revenue, profit, cash flow, debt, and liquidity. |
| News Analysis | Classifies articles as earnings, market, news, or irrelevant; extracts and routes facts. |
| Evaluator–Optimizer | Reviews the report and identifies corrections for the responsible agent. |

`BaseAgent` shares validation, logging, self-review, and revision code. `AgentContext` gives the agents access to the same model wrapper, evidence records, and execution log.

## Data and model

- **Yahoo Finance via `yfinance`:** company profile, historical prices, and annual financial statements.
- **NewsAPI:** recent article titles, descriptions, dates, sources, and URLs. These are previews, not necessarily full articles.
- **OpenAI API:** planning, news classification and extraction, interpretation, synthesis, and evaluation. The notebook's model setting is `gpt-4o-mini` and can be changed in its setup cell.

## Run the notebook

1. Open [Multi_agent_system_final.ipynb](Multi_agent_system_final.ipynb) in Google Colab or Jupyter.
2. Install its dependencies if necessary: `pandas`, `yfinance`, `requests`, and `openai`. The setup cell contains a `%pip install` command you can uncomment.
3. Supply `OPENAI_API_KEY` and `NEWSAPI` as Colab Secrets or environment variables. Do not place keys in notebook source or committed outputs.
4. Set `TEST_TICKER` and `RESEARCH_QUESTION`, then restart the runtime and run the cells in order. The research cell makes external data requests and paid OpenAI API calls.
5. Inspect the plan, specialist findings, sources, article-routing table, evaluation history, and any revisions before exporting the notebook to HTML or PDF.

The notebook initially requests up to three recent news articles for the research run. Adjust `page_size` in `retrieve_news()` if your experiment needs broader coverage, and record that choice in your report.

## Memory across runs

`AgentMemory` saves brief process lessons to `agent_memory.json`, including evaluator feedback, article-format failures, and insufficient price-history reminders. A later run reads relevant lessons and supplies them to the planner and agents. These notes are **process guidance**, not current financial evidence or model retraining. A successful run with no flagged issues can save an empty `lessons` list; that does not demonstrate adaptation.

In Colab, the default path is `/content/agent_memory.json`, which lasts only for the current runtime. Set `MEMORY_PATH` to a mounted Google Drive location if you need the file across runtime restarts. Keep memory files and API keys out of public commits.

## Routing evaluation

Section 6 uses the production `NewsAnalysisAgent.route()` method and records accuracy, macro-F1, a confusion matrix, processing failures, and per-article predictions. The [original examples](labeled_articles.json) are retained; the [reviewed development set](labeled_articles_reviewed.json) documents a CFO-succession label change to `news` and a Q3-to-Q4 title correction to match its supplied body. The source articles and URLs in these examples have not been verified.

An earlier run on the **original labels** reported **90% accuracy and 0.903 macro-F1 on 20 development examples**, with no processing failures and no prior memory lessons. That result measures the routing prompt on an inspected set. The reviewed-label set must be run separately, and its score must not be presented as the earlier result. A new, independently labeled set is needed for a final held-out evaluation. The optional memory comparison runs the same reviewed set with and without available general news lessons; it makes additional API calls.

The LLM evaluator's report score is feedback within the workflow, not an independent measure of financial accuracy. Citation checks confirm that referenced evidence IDs exist, but human review is still needed to check whether each claim is supported.

- [test_agentic_system.py](test_agentic_system.py): development tests for calculations, routing, validation, memory, and revision behavior. From the repository root, run `python3 -m unittest discover -s output/final_project -p 'test_*.py'` after installing `pandas`.

The final submission should include real run outputs, a fresh routing evaluation, limitations, source references, and each team member's actual contributions.
