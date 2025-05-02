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
  
  The dataset used in this study was obtained from [DrugBank Complete Database](https://go.drugbank.com/releases/latest), specifically focusing on all drugs released on January 2, 2025, under version 5.1.13.
</p>
