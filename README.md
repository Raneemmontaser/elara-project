# Elara — AI-Powered Educational Platform

**Elara** is an AI-powered educational platform for Egyptian high school students, combining adaptive
performance tracking, curriculum-grounded tutoring, and automated learning analytics into a single
mobile experience.

This repository brings together the three core ML/AI components of Elara, each maintained in its own
subfolder:

| Folder | Component | What it does |
|---|---|---|
| [`student-performance-kpis/`](./student-performance-kpis) | Student Performance Classification | A Random Forest model that classifies students into 5 performance levels (Beginner → Outstanding) from 11 learning KPIs, enabling adaptive hints and personalized interventions. |
| [`rag-tutoring/`](./rag-tutoring) | RAG-Based Socratic Tutoring | A Retrieval-Augmented Generation system that grounds tutor answers in official Egyptian high school textbooks (Arabic, Biology, Chemistry, Dynamics, English, Physics, Statics) using multilingual embeddings + Supabase vector search. |
| [`report-generation/`](./report-generation) | Automated Session Report Generation | A QLoRA-fine-tuned Qwen2.5-14B-Instruct model that turns raw tutoring session transcripts into structured, schema-validated cognitive evaluation reports, deployed serverlessly on Modal with vLLM. |

---

## How the pieces fit together

```
                Student interacts with Elara Mobile App
                              |
              +---------------+----------------+
              |                                |
              v                                v
   Student Performance KPIs           RAG Socratic Tutoring
   (Random Forest, 5 levels)          (multilingual-e5-small +
              |                        Supabase pgvector)
              |                                |
              |     Level informs tutoring     |
              +---------------> style <--------+
                                 |
                                 v
                    Tutoring session transcript
                                 |
                                 v
                     Report Generation (Elara-14B)
                     Structured JSON cognitive report
                                 |
                                 v
                  Student, teacher & parent insights
```

- **Student Performance KPIs** decides *where a student stands* (5 levels), which shapes how much
  scaffolding the tutor gives (hints vs. simplified explanations).
- **RAG Tutoring** answers student questions strictly from the curriculum, using Socratic-style
  guidance rather than direct answers.
- **Report Generation** analyzes the resulting tutoring sessions and produces structured reports on
  conceptual gaps, misconceptions, strengths, and recommendations for students, teachers, and parents.

---

## Component details

### 1. Student Performance Classification (`student-performance-kpis/`)
- **Algorithm:** Random Forest Classifier (scikit-learn)
- **Input:** 11 KPIs (accuracy, first-try success rate, hint usage, time-before-first-hint,
  post-hint improvement, topic weakness count, score trend, etc.)
- **Output:** Student level, 1 (Beginner) to 5 (Outstanding)
- **Training data:** 300 real records augmented to ~90k–100k synthetic records (noise injection,
  ~12% intentional contradictions, ~2% synthetic outliers) for robustness
- **Test accuracy:** ~95%

See [`student-performance-kpis/README.md`](./student-performance-kpis/README.md) for full KPI
definitions and dataset methodology.

### 2. RAG-Based Socratic Tutoring (`rag-tutoring/`)
- **Embedding model:** multilingual-e5-small (Arabic/English semantic search)
- **Vector store:** Supabase (pgvector)
- **Pipeline:** PDF textbook extraction → line-level chunking → embedding → similarity search →
  context-augmented LLM generation
- **Subjects covered:** Arabic, Biology, Chemistry, Dynamics, English, Physics, Statics

See [`rag-tutoring/README.md`](./rag-tutoring/README.md) for the full retrieval and query pipeline.

### 3. Automated Report Generation (`report-generation/`)
- **Base model:** Qwen2.5-14B-Instruct, fine-tuned via QLoRA (rank 64, alpha 128) on an L40S GPU
- **Training data:** 3,946 annotated tutoring sessions across 7 Egyptian curriculum subjects
- **Output:** Schema-validated JSON reports (conceptual gaps, misconceptions, strengths,
  recommendations)
- **Deployment:** Serverless on Modal, served with vLLM using guided JSON decoding on an
  NVIDIA A100 (40GB)

See [`report-generation/README.md`](./report-generation/README.md) for the full training pipeline
and deployment guide.

---

## Repository structure

```
elara-project/
├── README.md                    (this file)
├── student-performance-kpis/    # Random Forest performance classifier
├── rag-tutoring/                # RAG Socratic tutoring system
└── report-generation/           # Qwen2.5-14B fine-tuned report generator
```

## License
Each component retains its own license — see the `LICENSE` file inside the respective subfolder.
