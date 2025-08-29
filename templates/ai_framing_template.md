# ML Problem Framing

Application Name:  
Task Name: 

---
## Business Goal Identification
*Business goal identification is the most important phase of the ML lifecycle. An organization considering ML should have a clear idea of the problem to be solved, and the business value to be gained.*

- [ ] Understand business requirements
- [ ] Align affected stakeholders with this initiative
- [ ] Form a business question
- [ ] Identify critical, must-have features
- [ ] Consider new business processes that might come out of this implementation
- [ ] Consider how business value can be measured using business metrics

### What specific business value will this ML solution deliver?

### What are the key business performance metrics this will impact?

### Who are the stakeholders and how are they aligned?

### What problem is solved by this application or application feature?

### What new business processes might emerge from this implementation?

### What are the critical, must-have features for success?

---
## ML Problem Framing
*In this phase, the business problem is framed as a machine learning problem: what is observed and what should be predicted.*

- [ ] Define criteria for a successful outcome of the project
- [ ] Establish an observable and quantifiable performance metric for the project, such as accuracy
- [ ] Define the relationship between the technical metric (for example, accuracy) and the business outcome (for example, sales)
- [ ] Help ensure business stakeholders understand and agree with the defined performance metrics
- [ ] Formulate the ML question in terms of inputs, desired outputs, and the performance metric to be optimized

### ML Approach Validation
- [ ] Evaluate all available options for achieving the business goal
- [ ] Determine accuracy requirements vs cost and scalability of each approach
- [ ] Assess if simple business rules could achieve the same outcome
- [ ] Validate sufficient relevant, high-quality training data is available
- [ ] Evaluate whether ML is the right approach

#### Could this problem be solved without ML? What alternatives exist?

#### What makes ML the optimal approach for this specific use case?

#### What is the cost-benefit analysis of ML vs alternative approaches?

### ML Problem Definition

#### What is the ML Problem Statement?

#### What is observed (inputs) and what should be predicted (outputs)?

#### What types of ML problems need to be solved to deliver this solution (generation, summarization, retrieval, classification, regression, clustering, etc.)?

#### What are the Available Data sources?

#### What is the expected data volume and velocity?

#### What are the Output Requirements?

#### What is the target accuracy/performance threshold?

#### What are the latency requirements for predictions?

#### What Guardrails are required?

#### How will the model handle edge cases and out-of-distribution data?

### Performance Metrics and Success Criteria

#### What is a correct output?

#### What is an incorrect output?

#### What is the impact of a false positive?

#### What is the impact of a false negative?

#### What are the acceptable error rates for your use case?

#### How will model performance degrade gracefully under different conditions?

#### What are the baseline performance metrics to compare against?

#### How will you collect ongoing metrics during usage?

#### What constitutes model drift and how will you detect it?

### Metric Selection Guide
*Select applicable metrics for your evaluation. Delete metrics that don't apply to your use case.*

#### **Quality & Accuracy Metrics**
- [ ] **Correctness** - Measures if the model's response is factually correct and accurate
- [ ] **Completeness** - Evaluates if the response addresses all parts of the input request
- [ ] **Accuracy** - Proportion of correct predictions made by the model (traditional ML metric)
- [ ] **Precision** - True positives / (True positives + False positives)
- [ ] **Recall** - True positives / (True positives + False negatives)
- [ ] **F1-Score** - Harmonic mean of precision and recall
- [ ] **BLEU** - Bilingual evaluation score for text similarity (default threshold 0.5)
- [ ] **ROUGE-N** - Recall-oriented evaluation for text summarization (default threshold 0.75)
- [ ] **METEOR** - Metric for evaluation of translation with explicit ordering (default threshold 0.5)
- [ ] **GLEU** - Google-BLEU score for text similarity (default threshold 0.5)

#### **Coherence & Logic Metrics**
- [ ] **Logical Coherence** - Checks for logical gaps, inconsistencies, and contradictions in responses
- [ ] **Following Instructions** - Evaluates whether the model respects exact directions in the prompt
- [ ] **Relevance** - Measures how focused the response is on the given question
- [ ] **Context Faithfulness** - Ensures response uses provided context without hallucination
- [ ] **Context Relevance** - Ensures context is relevant to the original query
- [ ] **Context Recall** - Ensures ground truth information appears in the provided context

#### **Style & Readability Metrics**
- [ ] **Professional Style and Tone** - Evaluates appropriate business/professional communication style
- [ ] **Readability** - Assesses terminological and linguistic complexity for target audience
- [ ] **Helpfulness** - Holistic evaluation of how useful the response is for the user's needs

#### **Safety & Bias Metrics**
- [ ] **Harmfulness** - Detects harmful content including violence, hate, inappropriate material
- [ ] **Stereotyping** - Identifies content based on stereotypes (positive or negative)
- [ ] **Refusal** - Detects when model appropriately refuses inappropriate requests
- [ ] **Guardrails** - Ensures output doesn't contain harmful content using safety filters

#### **Technical Performance Metrics**
- [ ] **Latency** - Response time below specified threshold (milliseconds)
- [ ] **Cost** - Computational cost below threshold (for models with cost info)
- [ ] **Perplexity** - Language model uncertainty measure (lower is better)
- [ ] **Token Usage** - Tracks prompt, completion, and total token consumption

#### **Content Structure Metrics**
- [ ] **Is-JSON** - Validates output is properly formatted JSON
- [ ] **Contains-JSON** - Checks if output contains valid JSON structure
- [ ] **Is-HTML/XML** - Validates proper HTML or XML formatting
- [ ] **Is-SQL** - Validates SQL query syntax and structure
- [ ] **Function Call Validation** - Ensures function calls match expected schema

#### **Similarity & Matching Metrics**
- [ ] **Semantic Similarity** - Embeddings-based cosine similarity above threshold
- [ ] **Exact Match (Equals)** - Output matches expected value exactly
- [ ] **Contains** - Output contains specified substring(s)
- [ ] **Regex Match** - Output matches specified regular expression pattern
- [ ] **Levenshtein Distance** - Character-level edit distance below threshold

#### **Domain-Specific Metrics**
- [ ] **Answer Relevance** - Ensures LLM output relates to original query (RAG systems)
- [ ] **Conversation Relevance** - Maintains relevance throughout multi-turn conversations
- [ ] **Factuality** - Adherence to given facts using OpenAI evaluation methods
- [ ] **Classification** - Runs output through custom classifier for domain-specific categories

#### **Custom Evaluation Metrics**
- [ ] **JavaScript Custom** - Custom validation using JavaScript functions
- [ ] **Python Custom** - Custom validation using Python scripts
- [ ] **LLM Rubric** - Uses language model to grade against custom criteria
- [ ] **G-Eval** - Chain-of-thought evaluation based on custom criteria
- [ ] **Model-Graded Closed QA** - LLM-based evaluation for question-answering tasks

#### What specific metrics will you use to optimize results?

#### What are your success thresholds for each selected metric?

### Technical-Business Outcome Mapping
- [ ] Map the technical outcome to a business outcome
- [ ] Define clear relationships between technical metrics and business KPIs

#### How does each technical metric (accuracy, precision, recall) translate to business impact?

#### What technical performance threshold is required to achieve business objectives?

#### How will you communicate technical performance to business stakeholders?

---
## Data Strategy
*Create a strategy to achieve the data sourcing and data annotation objective.*

- [ ] Create a strategy to achieve the data sourcing and data annotation objective
- [ ] Review the project's ML feasibility and data requirements
- [ ] Start with a simple model that is easy to interpret, and makes debugging more manageable
- [ ] Iterate on the model by gathering more data, optimizing the parameters, or increasing the complexity as needed to achieve the business outcome

### Data Requirements and Availability

#### What data is available for testing?

#### How will you approach annotated data / Ground Truth to evaluate metrics?

#### What are the data quality requirements?

#### Are there any data gaps that need to be addressed?

### Iterative Development Approach
- [ ] Use evaluation tools to measure experiments with different models, prompts, and configurations
- [ ] Build solution components that assemble to complete the task
- [ ] Plan for iterative improvement through more data, different models, prompt and hyperparameter optimization, and architectural changes
- [ ] Design small, focused POCs to validate uncertain aspects

#### What is the simplest viable solution to start with?

#### How will you iterate and improve the solution over time?

#### What POCs are needed to validate uncertain aspects of the approach?

---
## Cost and Performance Optimization
*Evaluate the cost of data acquisition, training, inference, and wrong predictions.*

- [ ] Evaluate the cost of data acquisition, training, inference, and wrong predictions
- [ ] Evaluate whether bringing in external data sources might improve model performance
- [ ] Consider scalability requirements and associated costs

---
## Production Considerations
*Review how to handle ML-generated errors and establish pathways to production.*

- [ ] Review how to handle ML-generated errors
- [ ] Establish pathways to production
- [ ] Define monitoring and alerting strategies
- [ ] Plan for model drift and retraining

### Production Deployment

#### How will AI-generated errors be handled in production?

#### Will output keep the human in the loop?

### Monitoring and Maintenance

#### How will model performance be monitored over time?

#### What triggers model retraining or updates?

#### How is it monitored for ongoing performance?

#### What alerting mechanisms will be in place?


