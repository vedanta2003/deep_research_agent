# Deep Research AI Agent System


## **High-Level Overview**
The code builds a system that:
1. Generates targeted web search queries for a report section.
2. Searches the web and retrieves relevant content.
3. Writes the section content using the search results as context.
4. Formats completed sections for use in the final report.
5. Writes introduction and conclusion sections using the compiled report content.

The system is designed as a stateful graph, where each stage (query generation, web search, section writing) is a node in the graph.

Let’s walk through each part!

---

## **1. Query Generation**

The `generate_queries` function creates web search queries based on a given report section. 

- It uses a prompt (`REPORT_SECTION_QUERY_GENERATOR_PROMPT`) to instruct the LLM to generate focused, technically accurate queries.
- It ensures the queries cover multiple aspects of the topic (e.g., features, comparisons, recent updates) and aims for authoritative sources.
- The system prints the section name and returns the generated queries.

**Key components:**
- `section_topic`: The description of the section.
- `number_of_queries`: Set to 5, which balances breadth and depth.
- `structured_llm.invoke`: Calls the LLM to generate structured query output.

### **Example Query Generation:**
For a section on “Federated Learning in 2024,” the queries might look like:
- “Federated learning algorithms in 2024: advancements and limitations”
- “Federated learning vs centralized learning: performance benchmarks”

---

## **2. Web Search**

The `search_web` function executes the queries and fetches web content.

- It calls `run_search_queries` with a limit of 6 results per query.
- The function deduplicates and formats the results into a searchable string (`search_context`) with a 4000-token limit.

**Key components:**
- `query_list`: List of search query strings.
- `search_docs`: Raw search results, including content snippets.
- `format_search_query_results`: Compiles results into a usable context for the LLM.

### **Example Output:**
After searching for “Federated Learning in 2024,” the context might include:
- Snippets from research papers.
- Blog posts with practical implementations.
- Official documentation.

---

## **3. Section Writing**

The `write_section` function drafts the section using the web search results.

- It uses the `SECTION_WRITER_PROMPT`, guiding the LLM to write concise, technically accurate sections with strict length limits (150–200 words).
- The format is in Markdown, with:
  - A clear title (`## Section Title`)
  - A bold opening insight.
  - One optional structural element (table or list).
  - A “Sources” section.

**Key components:**
- `section_title` and `section_topic`: Used to frame the section.
- `source_str`: Context gathered from web searches.
- `section.content`: The final written section.

### **Example Output:**
For a section on “Privacy in Federated Learning,” the output might look like:

```
## Privacy in Federated Learning

**Privacy-preserving techniques like differential privacy and secure aggregation are crucial in federated learning to prevent data leakage.**

- **Differential Privacy:** Adds noise to gradients during training, preventing sensitive data extraction.
- **Secure Aggregation:** Encrypts model updates to hide individual contributions.

### Sources
- "Differential Privacy in Federated Learning (2023)": https://example.com
- "Secure Aggregation Techniques (2024)": https://example2.com
```

---

## **4. Parallelizing Section Writing**

The `parallelize_section_writing` function allows multiple sections to be researched and written simultaneously.

- It iterates over sections that require research and dispatches them to the `section_builder_with_web_search` subagent.
- This parallelism speeds up report generation, especially for multi-section documents.

---

## **5. Formatting and Compiling Sections**

The `format_sections` and `format_completed_sections` functions compile the written sections into a single formatted string.

- Each section is labeled (`Section 1`, `Section 2`, etc.).
- Sections without content are marked as `[Not yet written]`.

---

## **6. Writing Introduction and Conclusion**

The `write_final_sections` function handles special sections like the introduction and conclusion.

- It uses the `FINAL_SECTION_WRITER_PROMPT` to guide the LLM:
  - **Introduction:** 50–100 words, no structural elements, narrative-driven.
  - **Conclusion:** 100–150 words, with an optional table or list to summarize key insights.

### **Example Output (Conclusion):**

```
## Conclusion

**Federated learning has evolved into a privacy-first paradigm, balancing performance with security.**

| Aspect               | Approach               | Benefit                         |
|----------------------|-----------------------|----------------------------------|
| Privacy              | Differential Privacy  | Protects individual data points |
| Communication Overhead | Compression Algorithms | Reduces bandwidth usage        |
| Model Performance   | Personalized Training | Adapts models to local data     |

Next steps include exploring hybrid federated systems and refining communication protocols to further optimize learning efficiency.
```

---

## **7. Orchestration with State Graph**

The system uses `StateGraph` (from LangGraph) to organize the pipeline:

- **Nodes:** `generate_queries`, `search_web`, `write_section`
- **Edges:** Define the sequence of operations, starting with query generation and ending with section writing.
- **Subagent:** The `section_builder_subagent` runs the graph.

### **Visual Representation:**

```
START → generate_queries → search_web → write_section → END
```

The graph structure makes the system modular, allowing for easy scaling and section-specific customization.

---

## **8. Key Tools and Libraries**

- **LangChain:** For LLM interaction and stateful management.
- **LangGraph:** For building and executing the state graph.
- **Markdown:** For section formatting.
- **Web Search API:** To fetch real-time content.

---

## **9. Potential Improvements**

- **Search Quality Control:** Rank sources by credibility or relevance.
- **Query Refinement:** Use iterative query expansion if results are sparse.
- **Section Validation:** Check content for factual accuracy or hallucinations.
- **Customizable Section Lengths:** Allow flexible word limits based on report needs.

---

## **10. Final Thoughts**

This code provides a powerful framework for automating technical report writing. It combines real-time research with structured content generation, balancing depth, accuracy, and speed. By handling sections in parallel and synthesizing results intelligently, it significantly reduces manual effort while maintaining high-quality output.

Would you like me to help optimize this flow, improve the prompts, or guide you through building a complete report using your own section topics? Let me know! 🚀
