# Personalized and Explainable Learning Path Recommendation System

This repository contains the implementation of my Master's research thesis:

**"Towards Personalized Course Recommendation in E-Learning: Modeling Learning Pathways"**

The proposed system generates **personalized, ordered, and explainable learning paths** rather than simply recommending isolated courses.

It combines:

- a heterogeneous **Knowledge Graph**,
- a **GraphSAGE** graph neural network,
- a **hybrid course-ranking strategy**,
- inferred **prerequisite relationships**,
- prerequisite-aware **learning path generation**,
- and **retrieval-grounded explanations**.

---

##Problem Statement

The rapid growth of e-learning platforms makes it increasingly difficult for learners to identify courses that match their:

- current knowledge,
- target skills,
- learning objectives,
- and appropriate difficulty level.

Traditional recommender systems generally provide lists of relevant courses but do not necessarily indicate:

- which course should be taken first,
- how courses depend on one another,
- whether the sequence follows an appropriate difficulty progression,
- or why a course is recommended.

This project addresses these limitations by generating **personalized learning paths** that consider both recommendation relevance and pedagogical progression.

---

## Objective

The main objective is to answer the following question:

> **What sequence of courses should a learner follow to progress from their current knowledge toward their target skills?**

Instead of producing only a ranked list of courses, the proposed system constructs an ordered path while taking into account:

- learner objectives,
- target skills,
- completed courses,
- course difficulty,
- course relevance,
- and prerequisite relationships.

---

#  Proposed Approach

The system is composed of several stages.

---

## 1. Data Preprocessing

The project uses the **Coursera Courses Dataset 2021**.

The original data is cleaned and enriched before being used by the recommendation system.

The preprocessing process includes:

- cleaning course information,
- parsing the skills field,
- cleaning and standardizing skills,
- extracting course domains and subdomains,
- normalizing difficulty levels,
- removing duplicates,
- and assigning identifiers to courses.

After preprocessing, the dataset contains:

- **6,231 unique skills**
- approximately **10 skills per course**
- a minimum of **2 skills per course**
- a maximum of **14 skills per course**
- **12 main course domains**

The domains are:

- Business
- Computer Science
- Data Science
- Information Technology
- Health
- Math and Logic
- Personal Development
- Physical Science and Engineering
- Language Learning
- Social Sciences
- Life Sciences
- Arts and Humanities

Courses whose domain could not be reliably identified are assigned to an `Others` category.

These courses are kept in the course catalog but are excluded when generating simulated learner profiles.

---

## 2. Simulated Learner Profiles

Because the dataset mainly contains course information and does not provide real learner interaction histories, synthetic learner profiles are generated for experimental evaluation.

Each learner profile contains information such as:

- current knowledge level,
- target domain,
- target skills,
- completed courses,
- and courses used during recommendation evaluation.

These profiles allow the recommendation system to simulate different learning objectives and knowledge backgrounds.

---

## 3. Knowledge Graph Construction

A heterogeneous Knowledge Graph is constructed using **NetworkX**.

The graph is represented using:

```python
networkx.MultiDiGraph()
```

A `MultiDiGraph` is used because the graph is:

- **directed**, and
- capable of representing multiple types of relationships between entities.

### Node Types

The graph contains five main types of nodes:

- **Course**
- **Skill**
- **Domain**
- **Level**
- **Learner**

Example node identifiers include:

```text
C12
SKILL::machine learning
DOM::Data Science
LVL::Beginner
L1
```

Prefixes such as `SKILL::`, `DOM::`, and `LVL::` are used to prevent different entity types from sharing the same identifier.

---

## 4. Knowledge Graph Relationships

The graph models relationships between learners, courses, skills, domains, and levels.

Examples include:

```text
Course → has_skill → Skill

Course → belongs_to_domain → Domain

Course → has_level → Level

Learner → target_skill → Skill

Learner → completed → Course
```

The graph also contains inferred prerequisite relationships between courses:

```text
Course A → prerequisite_of → Course B
```

These relationships allow the recommendation model to use more than simple course metadata.

---

## 5. Prerequisite Inference

The Coursera dataset does not directly provide complete prerequisite relationships between courses.

Therefore, prerequisite relationships are **inferred** using heuristic rules.

The rules consider information such as:

- shared or related skills,
- course domain,
- course difficulty level,
- and logical progression between levels.

For example, when two related courses cover similar skills but one has a lower difficulty level, the lower-level course may be considered a prerequisite candidate for the more advanced course.

These relationships are stored in the Knowledge Graph using:

```text
prerequisite_of
```

edges.

The objective is not to claim that these are official Coursera prerequisites, but to estimate useful prerequisite relationships for constructing coherent learning paths.

---

# GraphSAGE Recommendation Model

The graph is used as input to a **Graph Neural Network based on GraphSAGE**.

GraphSAGE learns vector representations, or embeddings, of the entities in the graph by aggregating information from neighboring nodes.

This means that a course representation can indirectly incorporate information about:

- its skills,
- its domain,
- its level,
- related courses,
- and surrounding graph structure.

Similarly, learner representations can incorporate information about:

- target skills,
- completed courses,
- and related graph entities.

---

## Why GraphSAGE?

GraphSAGE was selected because it provides an **inductive graph-learning approach**.

Unlike methods that only memorize embeddings for the nodes observed during training, GraphSAGE learns how to aggregate neighborhood information.

This makes it suitable for recommendation settings where new learners or courses may later be introduced.

---

# Hybrid Recommendation Strategy

The final recommendation ranking does **not rely exclusively on the GraphSAGE score**.

The system uses a **hybrid ranking strategy**.

It combines:

1. the recommendation signal learned by the GraphSAGE model;
2. explicit relevance between the course skills and the learner's target skills.

Conceptually:

```text
Final Recommendation Score
          =
GraphSAGE-based relevance
          +
Target-skill relevance
```

The purpose of this combination is to benefit from both:

- graph-based relational information,
- and explicit alignment with the learner's learning objectives.

Courses that are structurally relevant according to the graph but do not contribute sufficiently to the learner's target skills can therefore be deprioritized.

---

# Personalized Learning Path Generation

The ranked recommendations are then transformed into an **ordered learning path**.

The learning path generation process considers:

- the final hybrid recommendation ranking,
- target skills,
- already completed courses,
- learner level,
- course difficulty,
- prerequisite relationships,
- and skill contribution.

Instead of returning:

```text
Course A
Course B
Course C
```

as independent recommendations, the system attempts to produce a meaningful sequence such as:

```text
Beginner Course
      ↓
Intermediate Course
      ↓
Advanced Course
```

---

## Prerequisite-Aware Ordering

Before placing a recommended course into the final path, the system checks whether relevant prerequisite courses should appear earlier.

This helps ensure that advanced courses are not recommended before the knowledge required to follow them.

The learning path therefore attempts to respect both:

- **recommendation relevance**
- and **learning progression**

---

## Difficulty Progression

The generated paths also consider course difficulty.

The goal is to avoid incoherent sequences such as:

```text
Advanced
   ↓
Beginner
   ↓
Intermediate
```

and instead favor progression such as:

```text
Beginner
   ↓
Intermediate
   ↓
Advanced
```

when appropriate.

---

# Explainability

The system provides an explanation for recommended courses.

The final explanation mechanism is better described as a:

**retrieval-grounded, template-guided explanation approach**

rather than a fully generative RAG pipeline.

For each recommendation, factual information is retrieved from the recommendation system.

This information can include:

- matched target skills,
- course difficulty level,
- course domain,
- prerequisite information,
- related courses,
- and the position of the course in the generated learning path.

These retrieved facts are inserted into a controlled explanation structure.

An explanation may therefore indicate that a course was selected because:

- it covers one or more of the learner's target skills,
- its difficulty matches the learner's current progression,
- it follows a prerequisite course,
- or it prepares the learner for a later course in the path.

### Example

```text
This course is recommended because it covers Python and Machine Learning,
which are among the learner's target skills. Its intermediate difficulty
is appropriate for the current learning progression, and it follows a
relevant prerequisite course included earlier in the learning path.
```

Using retrieved facts rather than unconstrained text generation makes the explanations more transparent and reduces unsupported or hallucinated information.

---

# Complete System Pipeline

The overall architecture can be summarized as:

```text
Coursera Courses Dataset
          │
          ▼
Data Cleaning & Preprocessing
          │
          ▼
Course Enrichment
          │
          ├── Skills
          ├── Domains
          └── Levels
          │
          ▼
Simulated Learner Profiles
          │
          ▼
Knowledge Graph Construction
          │
          ├── Courses
          ├── Skills
          ├── Domains
          ├── Levels
          └── Learners
          │
          ▼
Prerequisite Inference
          │
          ▼
GraphSAGE
          │
          ▼
Learner & Course Embeddings
          │
          ▼
Hybrid Recommendation Ranking
          │
          ├── GraphSAGE relevance
          └── Target-skill relevance
          │
          ▼
Prerequisite-Aware
Learning Path Construction
          │
          ▼
Personalized Learning Path
          │
          ▼
Retrieval-Grounded Explanation
```

---

# Evaluation

The recommendation component is evaluated using common ranking metrics.

These include:

### Precision@K

Measures the proportion of recommended courses in the top K that are relevant.

### Recall@K

Measures how many relevant courses are successfully retrieved in the top K recommendations.

### NDCG@K

Evaluates ranking quality while giving more importance to relevant courses appearing near the top of the recommendation list.

### Hit@K

Determines whether at least one relevant course appears within the top K recommendations.

---

# Learning Path Evaluation

Recommendation accuracy alone is not sufficient to determine whether a generated learning path is pedagogically coherent.

Additional metrics are therefore used to evaluate the final paths.

## Prerequisite Respect

Measures whether prerequisite courses appear before the courses that depend on them.

**Result:**

```text
Proposed approach: 100%
Naive ordering:     25%
```

---

## Difficulty Monotonicity

Measures whether course difficulty progresses coherently through the learning path.

**Result:**

```text
Proposed approach: 0.933
Naive ordering:     0.822
```

The value is not necessarily exactly `1.0` because some course difficulty labels can share equivalent numerical levels.

---

## Skill Coverage

Measures how well the generated learning path covers the learner's target skills.

**Result:**

```text
99.2%
```

---

## Average Path Length

The average generated learning path contains:

```text
3.04 courses
```

---

## Summary of Learning Path Results

| Metric | Proposed System | Naive Baseline |
|---|---:|---:|
| Prerequisite Respect | **100%** | 25% |
| Difficulty Monotonicity | **0.933** | 0.822 |
| Skill Coverage | **99.2%** | — |
| Average Path Length | **3.04 courses** | — |

These results indicate that incorporating prerequisite information and explicit path construction improves the structural coherence of the generated learning sequences.

---

# 🧪 Implementation

The current implementation is contained primarily in a **Google Colab/Jupyter notebook**.

The notebook contains the complete experimental pipeline, including:

1. dataset loading,
2. preprocessing,
3. skill extraction and normalization,
4. domain extraction,
5. learner profile generation,
6. Knowledge Graph construction,
7. prerequisite inference,
8. graph preparation,
9. GraphSAGE training,
10. recommendation scoring,
11. hybrid ranking,
12. learning path generation,
13. explanation generation,
14. and evaluation.

---

# Technologies Used

The project is implemented in **Python** and uses libraries including:

- Python
- Pandas
- NumPy
- NetworkX
- PyTorch
- PyTorch Geometric
- Scikit-learn
- Matplotlib
- Google Colab

Additional libraries may be used for data processing and evaluation.

---

# Running the Project

The easiest way to run the project is using **Google Colab**.

### 1. Open the notebook

Open the `.ipynb` file contained in this repository.

### 2. Open it with Google Colab

You can either upload the notebook manually to Colab or open the GitHub version directly from Colab.

### 3. Install required dependencies

Some libraries may need to be installed when starting a new Colab runtime.

### 4. Load the dataset

Make sure the required dataset files are available in the expected notebook location.

### 5. Run the notebook

Execute the cells sequentially from the beginning to reproduce the complete pipeline.

---

# Current Repository Structure

The project is currently notebook-based.

```text
personalized-learning-path-recommendation/
│
├── README.md
│
└── thesis_pipeline.ipynb
```

Additional files such as the dataset or generated experimental outputs may be added separately when appropriate.

---

# Main Contributions

The main contributions of this work are:

1. **Construction of a heterogeneous Knowledge Graph** containing courses, skills, domains, levels, and learners.

2. **Inference of prerequisite relationships** between related courses using heuristic rules.

3. **Application of GraphSAGE** to learn graph-based representations of learners and courses.

4. **Hybrid recommendation ranking** combining GraphSAGE-based relevance with explicit target-skill matching.

5. **Generation of personalized learning paths** instead of isolated course recommendations.

6. **Prerequisite-aware ordering** to improve learning path coherence.

7. **Difficulty-aware progression** across recommended courses.

8. **Retrieval-grounded explanations** based on factual recommendation and learning-path information.

---

# Research Context

This project was developed as part of a Master's research thesis in Intelligent Information Systems.

The research investigates how:

- Knowledge Graphs,
- Graph Neural Networks,
- hybrid recommendation,
- prerequisite modeling,
- learning path generation,
- and explainability

can be combined to improve personalized course recommendation in e-learning environments.

---

#  Thesis Title

**Towards Personalized Course Recommendation in E-Learning: Modeling Learning Pathways**

---

## Author

Master's Research Project  
Intelligent Information Systems
