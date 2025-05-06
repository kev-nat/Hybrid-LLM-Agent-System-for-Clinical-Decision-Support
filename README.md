# Hybrid LLM Agents for Clinical Support in Drug Drug Interactions

## Situation
<p align="justify"> 
Drug-Drug Interactions (DDIs) represent a significant global healthcare challenge, with adverse drug reactions (ADRs) accounting for approximately 30% of all adverse drug events and causing nearly 5% of hospital admissions worldwide <a href="https://www.pharmacytimes.com/view/tips-for-managing-adverse-drug-reactions">[1]</a>. The complexity of drug interactions has grown exponentially with the increasing prevalence of polypharmacy, where patients simultaneously use multiple medications, leading to an estimated annual cost of $28.5 billion in DDI-related complications in the United States alone <a href="https://milkeninstitute.org/sites/default/files/reports-pdf/ChronicDiseases-HighRes-FINAL.pdf">[2]</a>. The detection and prediction of DDIs pose substantial challenges in clinical practice. Traditional approaches rely heavily on clinical trials, post-market surveillance, and expert knowledge bases, which are time-consuming, resource-intensive, and often identify DDIs only after adverse events have occurred <a href="https://link.springer.com/article/10.1007/s11096-024-01709-x">[3]</a>.
</p>

<p align="justify"> 
Current computational approaches for accessing DDI information rely primarily on direct database queries to structured repositories like DrugBank, which already contain comprehensive interaction data <a href="https://jnanobiotechnology.biomedcentral.com/articles/10.1186/s12951-022-01605-4">[4]</a>. While these databases provide valuable information, healthcare professionals often struggle with technical query interfaces that require exact terminology and structured inputs, limiting their practical utility in fast-paced clinical environments. Additionally, these systems typically lack the ability to interpret natural language queries or provide contextual explanations that are crucial for clinical decision-making and education.
</p>

<p align="justify">
To address these limitations, Large Language Models (LLMs) have been applied to process natural language queries about drug interactions, enabling more intuitive access to pharmaceutical information <a href="https://bmcbioinformatics.biomedcentral.com/articles/10.1186/s12859-020-03724-x">[5]</a>. However, standalone LLMs face two critical challenges in clinical settings: (1) they frequently hallucinate when generating database queries due to the lack of schema constraints, and (2) they struggle with factual consistency when handling highly specialized pharmaceutical data from electronic health records <a href="https://link.springer.com/article/10.1007/s44163-024-00114-7">[6]</a>. These shortcomings are particularly problematic for medication safety applications, where deterministic correctness is non-negotiable.
</p>

<p align="justify">
This project bridges these gaps through a hybrid LLM agent system that combines the semantic capabilities of language models with rule-based tool selection. The architecture employs intent classification to dynamically route queries, directing interaction-specific requests through structured database retrieval with schema-constrained SQL generation, while general pharmacological questions activate curated search tools restricted to FDA-approved sources. Unlike conventional systems that apply uniform processing to all queries, this adaptive framework ensures clinical reliability for critical DDI checks while maintaining flexibility for educational use cases. This hybrid approach can enhance the learning experience of medical students and assist healthcare providers in their clinical workflow by providing organized, actionable interaction information in an accessible format attributable to the system's dual capability to interpret natural language queries while preserving deterministic accuracy for core interaction data.
</p>

## Dataset
<p align="justify">
  
  The dataset used in this study was obtained from [DrugBank Complete Database](https://go.drugbank.com/releases/latest), specifically focusing on all drugs released on January 2, 2025, under version 5.1.13. A Python script was developed to scarp and extract all available properties from the XML format and convert them into a CSV file. This script successfully retrieved 20 columns from the original XML dataset, generating a CSV file of approximately 6.95 GB. However, a significant portion of the extracted data contained NaN (null) values, and not all attributes were relevant to this study as it shown in a figure below. 
</p>

<div align="center">
  <h4>Overview of Uncleaned Drug-Drug Interaction Dataset Used in the Proposed Research</h4>
</div>

![image](https://github.com/user-attachments/assets/d2e2b762-751a-4afe-ac23-01cc431725f9)

<p align="justify">
To address data quality, a cleaning process was performed, producing a refined dataset with five essential columns: <code>drug_id</code>, <code>drug_name</code>, <code>interacting_drug_id</code>, <code>interacting_drug_name</code>, and <code>description</code>. The resulting dataset contains 2,855,848 records with no missing values, a final size of 406.93 MB, and 4,566 unique drugs, as shown below. This cleaned drug interactions file was exported to PostgreSQL, with <code>drug_id</code> and <code>interacting_drug_id</code> forming a composite primary key laying the groundwork for the deterministic component of the system.
</p>

<p align="justify">
To visualize the intricate interaction relationships within the dataset, the <code>networkx</code> library was employed to transform the CSV data into a graph structure: each unique drug serves as a node, with <code>drug_id</code> as the source and <code>interacting_drug_id</code> as the target. The <code>description</code> column defines the edges between nodes, capturing the nature of each interaction. This network visualization offers a comprehensive view of the complex interdependencies among pharmaceuticals that the system is designed to manage.
</p>

<h4 align="center">Overview of Cleaned Drug-Drug Interaction Dataset Used in the Proposed Research</h4>
<p align="center">
  <img src="https://github.com/user-attachments/assets/03acaa39-7f97-4c0f-a26c-3490f95107ba" width="595" style="margin: 0 10px;" />
  <img src="https://github.com/user-attachments/assets/f0e66784-6892-4aeb-a55a-e27e5728d52c" width="400" style="margin: 0 10px;" />
</p>

## Hybrid LLM Agent System Approaches
<p align="justify">
While LLM agents and frameworks like LangChain offer theoretical advantages for clinical decision support of DDIs, their practical implementation requires careful integration of deterministic safeguards and patient-specific contextualization. This work addresses these needs through a hybrid LLM Agent system that combines the strengths of both deterministic and probabilistic approaches. The system employs two primary tools: one related to the database for drug interactions queries and another for searching the internet for general information about drugs beyond DDI data, creating a comprehensive knowledge framework that maintains accuracy while providing flexibility.  
</p>

### Deterministic Query Processing
<p align="justify">
The implementation a custom chained QA LLM pipeline with PostgreSQL offers a robust approach to interpreting clinical queries with high precision. By using schema-aware prompting and grounding the model in the database, the system effectively constrains its outputs and reduces hallucinations, as shown below.
</p>

<h4 align="center">Custom Chained QA LLM Pipeline with PostgreSQL Database Integration using LangChain</h4>
<p align="center">
  <img src="https://github.com/user-attachments/assets/f014fe8c-c7d4-472f-af86-092848d416c5" width="800" style="margin: 0 10px;" />
</p>

#### Step 1
<p align="justify">
The pipeline begins with a user submitting a natural language question, which enters the full chain processing system. Within this system, the SQL chain first processes the query by routing it through an initial LLM that has been provided with the database schema as context. This LLM transforms the natural language question into a structured SQL query, serving as a controlled interpretation layer. The schema-aware prompting is critical here, as it ensures the LLM generates syntactically correct queries that align with the actual database structure built from the DrugBank dataset.
</p>

#### Step 2
<p align="justify">
Once the SQL query is formulated, it moves to the execution phase where it runs against the PostgreSQL database, retrieving precise, factual information. This retrieved data then passes through a LLM which translates the technical database results back into natural language, producing a comprehensive answer for the user. By grounding responses in actual database content, this architecture significantly reduces the risk of hallucinations that typically plague pure LLM interactions.
</p>

#### Step 3
<p align="justify">
The implementation specifically targets pharmaceutical applications, where the database contains drug interaction information, but the pattern remains adaptable across various domains. The database initialization function establishes the connection to PostgreSQL, while the SQL chain function incorporates conversation history and expert context to generate appropriate queries.
</p>

#### Step 4
<p align="justify">
Finally, the response generation function orchestrates the entire flow, ensuring seamless transformation from user question to database query to natural language answer. This deterministic approach creates a reliable interface between users and complex clinical data, allowing non-technical users to access pharmaceutical information through natural conversation while maintaining factual accuracy. This custom pipeline will be defined as a tool for the system, which later will be used by the LLM Agent. However, while this tool excels at answering database-specific queries, it requires complementary capabilities for handling broader pharmaceutical information.
</p>

### External Search Integration
<p align="justify">
To complement the database query system described in the previous section, a SerpApi is incorporated to address queries that extend beyond the structured database content while remaining relevant to drug-related inquiries. When users pose questions about information not stored in the database, such as drug manufacturing processes, historical development, or chemical properties, the SerpApi Search tool provides supplementary knowledge.
</p>

<p align="justify">
The tool is formally defined and registered within the system with a descriptive name and function pointer to the wrapper's run method, following the LangChain framework. The search tool is then incorporated into a <code>zero-shot-react-description</code> agent framework, which uses a ChatOpenAI model with controlled temperature settings to ensure reliable responses exemplifying the LLM agent architecture.
</p>
  
<p align="justify">
This agent evaluates the incoming query and determines the appropriate action sequence, leveraging the React framework's ability to reason before acting. When processing a query like "When was Lepirudin created?", the agent recognizes this as a search operation, invokes SerpApi, and returns comprehensive information about the drug's history, manufacturer, and related details that wouldn't be present in an interaction database.
</p>  

<p align="justify">
This dual-tool approach creates a more comprehensive knowledge system, allowing users to receive detailed answers whether their questions pertain to drug interactions stored in the database or broader pharmacological information available through SerpApi, all while maintaining a consistent conversational interface. However, effective implementation requires orchestration between these specialized tools.  
</p>  

### Workflow Orchestration
<p align="justify">
The workflow orchestration layer serves as the intelligent routing mechanism that seamlessly coordinates between the deterministic database queries and external search capabilities described in the previous sections, creating a cohesive hybrid system as it shown in a figure below.
</p>

<h4 align="center">Complete framework of Hybrid LLM Agent System for Clinical Decision Support in DDI</h4>
<p align="center">
  <img src="https://github.com/user-attachments/assets/1639e27f-3052-461a-b4cb-c1190c629851" width="400" style="margin: 0 10px;" />
</p>

<p align="justify">
This integration operates through a structured decision-making pipeline where each user query is first evaluated by an agent to determine its appropriate processing path. The agent architecture combines LLMs with specialized tools, enabling the system to extend beyond the limitations of isolated models. When implementing tools within this framework, three critical components must be defined: a distinctive name for identification, a function pointer for execution, and a comprehensive description that guides the LLM in understanding the tool's capabilities and appropriate use cases directly 
</p>

<p align="justify">
The orchestration process begins when a user submits a query, which the LLM first analyzes to understand the underlying intent. Based on this analysis, the agent determines whether the query requires database access for drug interaction information or external search for broader pharmaceutical knowledge. This dynamic tool selection leverages the <code>zero-shot-react-description</code> agent, where the LLM reasons through the appropriate action sequence before execution. For instance, when encountering a query like <code>"What are the interactions between Lepirudin and Aspirin?"</code>, the agent routes it to the custom database tool, whereas a question such as <code>"History of Lepirudin?"</code> triggers the SerpApi search tool.
</p>

<h4 align="center">Runnable Sequence and Agent Executor Reasoning Results viewed in LangSmith</h4>
<p align="center">
  <img src="https://github.com/user-attachments/assets/92704bcb-320b-4b37-b40f-ca18f9127263" width="400" style="margin: 0 10px;" />
  <img src="https://github.com/user-attachments/assets/59b062a4-e6b5-4cc5-9686-606eb22764d6" width="480" style="margin: 0 10px;" />
</p>

<p align="justify">
This orchestration addresses fundamental limitations of standalone LLMs, which cannot directly interact with real-time data sources or perform complex database operations. By functioning as an intelligent coordinator, the agent bridges the gap between the LLM's reasoning capabilities and external data access requirements. The system's architecture ensures that each query receives the most appropriate treatment whether it requires the deterministic precision of database lookups or the broader information gathering of web searches while maintaining a unified conversational interface. This workflow orchestration creates a more robust and comprehensive clinical decision support system that can handle the full spectrum of drug-related inquiries while preserving factual accuracy and relevance, fulfilling the objective of LLM agents for DDI analysis. 
</p>

## Key Factors that Influenced System Outcomes

### 1. LLM Specifications and Temperature
<p align="justify">
This study use GPT-3.5-turbo as the core language model, which has a token limitation of 4,096 tokens. When this limit is exceeded, the system will generate an error. The token limitation is an important consideration in the design of prompts and the overall interaction flow, as it constrains the amount of context and response data that can be processed in a single query.
</p>

<p align="justify">
Not only token limitations that is impacted but also the temperature settings are a crucial parameter in controlling the output behavior of LLMs in this hybrid system. For the Database QA tool, a low temperature range (0.0-0.3) is implemented to ensure high accuracy and consistency. This low temperature setting makes the model's outputs more deterministic and factual, which is essential when generating SQL queries that must adhere strictly to database schema requirements. The reduced randomness ensures that queries are predictable, precise, and directly relevant to the information requested.
</p>

<p align="justify">
In contrast, the SerpApi Search tool use a higher temperature setting (0.4-0.7) to introduce beneficial diversity in responses. This moderate increase in temperature allows the model to handle the inherent ambiguity in general search queries by exploring a wider range of possible interpretations. For open-ended pharmaceutical questions, this approach provides more natural, conversational responses that feel more human-like and comprehensive. The temperature difference between the two tools reflects their fundamentally different objectives: the Database QA tool prioritizes factual accuracy and technical precision, while the Search tool emphasizes breadth, versatility, and natural communication.
</p>

### 2. Prompt Engineering
<p align="justify">
The efficacy of the hybrid LLM agent depends heavily on designed prompts that direct the model toward specific tasks while maintaining domain expertise. The system implements three distinct prompt templates, each engineered for specific functions within the overall workflow.
</p>

### 2.1 QA DB Tools Prompt (SQL Query Generation)
<p align="justify">
The SQL query generation prompt positions the model as an expert pharmacologist analyzing a drug interaction database. This prompt template receives the database schema and conversation history as context, then transforms the user's natural language question into a precisely formatted SQL query. The template explicitly instructs the model to generate only the SQL query without explanatory text or code formatting elements like backticks. It includes illustrative examples showing how specific user questions (e.g., "What's the interaction of Lepirudin and Apixaban?") should be translated into corresponding SQL queries (e.g., "SELECT description FROM ddi WHERE (drug_name = 'Lepirudin' AND interacting_drug_name = 'Apixaban');"). These examples demonstrate expected output formats and common query patterns, helping the model understand the translation process from natural language to database queries. The prompt maintains strict output constraints to ensure the generated SQL is directly executable against the database without requiring additional processing or cleanup.
</p>

### 2.2 DB Response Prompt Query (SQL Query to Natural Language)
<p align="justify">
The database response prompt serves as an interpretive layer between raw database results and user-friendly explanations. This template takes multiple inputs the database schema, conversation history, the executed SQL query, the user's original question, and the raw SQL response data and synthesizes them into coherent natural language explanations. The prompt establishes the same expert pharmacologist persona as the query generation prompt, maintaining consistency in the user experience. Unlike the previous prompt that produces only SQL code, this prompt generates comprehensive explanations that contextualize the database findings in terms relevant to the user's original question. For example, when processing results about drug interactions, the model will explain potential risks, severity, or recommendations in pharmacological terms rather than simply reporting the raw data. This transformation step is essential for making technical database information accessible and meaningful to users who may lack database expertise or detailed pharmaceutical knowledge.
</p>

### 2.3 DB Query vs Internet Query (Query Categorization)
<p align="justify">
The query categorization prompt functions as the decision-making gateway in the system's workflow. This prompt template instructs the model to analyze incoming user queries and strictly categorize them as either "DATABASE" or "SerpApi" based on specific criteria. Database queries are identified as those specifically about drug-to-drug interactions between named medications or direct questions about the database structure itself. All other pharmaceutical queries including those about drug compositions, side effects (unless interaction-specific), pharmacology, chemical properties, manufacturing processes, historical information, mechanisms of action, or dosage information are categorized as "SerpApi" queries to be handled by the general search tool. The prompt emphasizes strict categorization with explicit instructions and clearly defined boundaries, preventing edge cases from being incorrectly routed. This critical routing function ensures that each user question is directed to the most appropriate processing pipeline, maximizing the system's ability to provide relevant and accurate information while optimizing computational resource utilization.
</p>

<p align="justify">
Following the detailed examination of each prompt, the key differences between the three prompt templates can be clearly summarized in a Table below. Each prompt serves a distinct purpose within the workflow of handling user queries related to a drug interaction database. The QA DB Tools prompt transforms natural language into SQL queries, the DB Response prompt converts technical results into user-friendly explanations, and the Query Categorization prompt ensures questions are routed to the appropriate system component. It provides a concise overview of these key distinctions, highlighting their objectives, outputs, contextual requirements, focus areas, and representative use cases.
</p>

<div align="center">
  <h4>Summary of differences among three prompts used for handling drug interaction queries, including SQL query generation, result interpretation, and query categorization</h4>
</div>

<div align="center">
<table>
  <thead>
    <tr style="border-top: 2px solid black; border-bottom: 1px solid black;">
      <th style="text-align: left; padding: 8px;">Aspects</th>
      <th style="text-align: left; padding: 8px;">QA DB Tools Prompt (SQL Query)</th>
      <th style="text-align: left; padding: 8px;">DB Response Prompt (Natural Language)</th>
      <th style="text-align: left; padding: 8px;">DB Query vs Web Search Query (Categorization)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: left; padding: 8px;">Objectives</td>
      <td style="text-align: left; padding: 8px;">Generate SQL queries based on user questions.</td>
      <td style="text-align: left; padding: 8px;">Convert SQL query results into a natural language response.</td>
      <td style="text-align: left; padding: 8px;">Classify the query as either for the database or for SerpApi</td>
    </tr>
    <tr>
      <td style="text-align: left; padding: 8px;">Output</td>
      <td style="text-align: left; padding: 8px;">SQL query only (no extra text).</td>
      <td style="text-align: left; padding: 8px;">Natural language explanation of the SQL query’s results.</td>
      <td style="text-align: left; padding: 8px;">Either "DATABASE" or "SerpApi".</td>
    </tr>
    <tr>
      <td style="text-align: left; padding: 8px;">Context</td>
      <td style="text-align: left; padding: 8px;">Schema, conversation history, and user question.</td>
      <td style="text-align: left; padding: 8px;">Schema, conversation history, SQL query, and SQL response.</td>
      <td style="text-align: left; padding: 8px;">Conversation history and user query.</td>
    </tr>
    <tr>
      <td style="text-align: left; padding: 8px;">Focus</td>
      <td style="text-align: left; padding: 8px;">Constructing accurate SQL queries.</td>
      <td style="text-align: left; padding: 8px;">Interpreting and explaining SQL results in plain language.</td>
      <td style="text-align: left; padding: 8px;">Determining whether the query is database-specific or general info.</td>
    </tr>
    <tr>
      <td style="text-align: left; padding: 8px;">Example Use Case</td>
      <td style="text-align: left; padding: 8px;">"What's the interaction of Lepirudin and Apixaban?"</td>
      <td style="text-align: left; padding: 8px;">"Lepirudin and Apixaban interact, increasing bleeding risk."</td>
      <td style="text-align: left; padding: 8px;">""What is the composition of Apixaban?" → routed to Internet."</td>
    </tr>
  </tbody>
</table>
</div>

