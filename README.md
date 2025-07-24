# Hybrid LLM Agent System for Clinical Decision Support
This project develops and evaluates a hybrid Large Language Model (LLM) agent system designed to provide accurate and reliable information on Drug-Drug Interactions (DDIs) for clinical decision support and medical education.

https://github.com/user-attachments/assets/a958fa8c-3472-4ae5-9326-af55ba65c8b0

## Situation
Drug-Drug Interactions (DDIs) are a major cause of adverse medical events and represent a significant challenge in healthcare. While comprehensive databases like DrugBank exist, clinicians often find their technical query interfaces difficult to use in fast-paced environments. At the same time, standard Large Language Models (LLMs) are not suitable for this critical task because they can "hallucinate" or generate factually inconsistent information, which is unacceptable for medication safety.

## Objectives
The primary goal of this project is to create a reliable clinical decision support tool that bridges the gap between the conversational power of LLMs and the factual accuracy of structured medical databases.

The key objectives are:
- Develop a hybrid LLM agent that can accurately answer natural language questions about DDIs.
- Ensure factual correctness by forcing the system to retrieve information directly from a verified database for critical queries.
- Build an intelligent routing system that can distinguish between a specific DDI query and a general pharmacology question, using the best tool for each.
- Evaluate the system's performance using advanced, meaning-based metrics to prove its reliability for clinical use.

## Approaches
Designed a hybrid agent system that intelligently orchestrates two distinct tools, avoiding the potential ambiguities of a standard Retrieval-Augmented Generation (RAG) approach.

**1. Architecture:** The system uses GPT-3.5 Turbo with LangChain to power an agent with access to two specialized tools. The agent analyzes the user's intent to decide which tool to use.

**2. Deterministic Database Tool:**
- For specific DDI queries (e.g., "Interaction between Lepirudin and Apixaban?"), the agent uses a custom, schema-aware pipeline to generate a precise SQL query.
- This query is run against a PostgreSQL database containing the complete DrugBank dataset.
- This process is highly controlled (low temperature setting) to ensure the output is factually grounded and free of hallucinations.

**3. General Search Tool:**
- For broader questions (e.g., "What is the history of Lepirudin?"), the agent routes the query to a web search tool (SerpApi).
- This allows for more flexible, conversational answers on topics not contained within the structured DDI database.

## Impacts
The system was rigorously evaluated and demonstrated exceptional performance, proving its potential for real-world clinical and educational applications.

- **High Factual Accuracy:** The system achieved 98.17% accuracy in identifying correct drug entities and 88.50% accuracy in classifying the interaction type, validating the reliability of the database-grounded approach.
- **Excellent Semantic Performance:** Traditional text-matching metrics showed 0% accuracy, but our advanced evaluation revealed the system's true capabilities. It achieved 89.4% average semantic similarity and a 96% equivalence score from an LLM-based judge (GPTScore), confirming that it consistently provides the correct information, even if worded differently.
- **Proven Reliability:** The intelligent routing mechanism successfully mitigates the risk of LLM hallucination for critical queries, establishing a robust and trustworthy framework for using LLMs in a clinical context.
- **Clinical and Educational Value:** This project provides a strong foundation for a tool that can help clinicians make safer prescribing decisions and serve as an advanced, interactive learning platform for medical students.
