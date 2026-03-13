# JPIS Database Architecture

This document defines the database architecture used by the JPIS system.

JPIS uses a hybrid storage architecture consisting of:

Relational database (PostgreSQL)
Vector database (Qdrant)

This architecture allows the system to combine structured legal data with semantic AI search.

---

# Overview

JPIS stores legal information using two data layers.

PostgreSQL stores structured legal data.

Qdrant stores vector embeddings used for semantic search.

System architecture:

Frontend
↓
Backend API
↓
PostgreSQL (legal structure)
↓
Qdrant (vector search)

---

# PostgreSQL Database

PostgreSQL stores the structured representation of the legal system.

Main entities include:

laws
law_articles
court_decisions
legal_literature
legal_concepts

These tables define the legal knowledge graph.

---

# Table: laws

Stores high-level information about laws.

Fields:

id
title
short_title
jurisdiction
official_gazette
status
created_at
updated_at

Example record:

Criminal Code of Bosnia and Herzegovina

---

# Table: law_articles

Stores individual law articles.

Fields:

id
law_id
article_number
title
content
created_at
updated_at

Each law article is stored as a separate record.

Example:

Article 239 – Fraud

This structure allows precise referencing and indexing.

---

# Table: court_decisions

Stores court decisions.

Fields:

id
court_name
case_number
decision_date
summary
full_text
created_at

Court decisions often reference specific law articles.

---

# Table: court_decision_articles

Defines relationships between court decisions and law articles.

Fields:

id
decision_id
article_id

Example relationship:

Court decision Kž-123/20 → Article 239

This table forms part of the legal knowledge graph.

---

# Table: legal_literature

Stores legal academic materials.

Fields:

id
title
author
publication_year
content
created_at

Legal literature helps interpret laws and court decisions.

---

# Table: legal_concepts

Stores legal ontology concepts.

Fields:

id
name
definition

Example concepts:

Fraud
Intent
Property Damage
Appeal

---

# Table: concept_article_links

Links legal concepts with law articles.

Fields:

concept_id
article_id

Example:

Concept Fraud → Article 239

---

# Knowledge Graph

The knowledge graph emerges from relationships between tables.

Examples:

LawArticle → CourtDecision
CourtDecision → LegalPrinciple
LawArticle → LegalLiterature
Concept → LawArticle

This structure allows AI to navigate legal reasoning.

---

# Qdrant Vector Database

Qdrant stores embeddings used for semantic search.

Collections include:

law_articles
court_decisions
legal_literature

Each document stored in PostgreSQL has a corresponding embedding in Qdrant.

---

# Example Vector Record

Example vector payload:

```
{
  "id": "law_article_kz_bih_239",
  "vector": [...],
  "payload": {
    "type": "law_article",
    "law": "Criminal Code of Bosnia and Herzegovina",
    "article": "239"
  }
}
```

Vectors represent the semantic meaning of legal texts.

---

# Retrieval Workflow

When a user asks a question:

User question
↓
Embedding generation
↓
Vector search (Qdrant)
↓
Retrieve relevant document IDs
↓
Fetch full documents from PostgreSQL
↓
Construct AI prompt
↓
Generate response

This architecture enables accurate AI-assisted legal research.

---

# Data Integrity

PostgreSQL is the source of truth for legal data.

Qdrant contains derived embeddings.

If embeddings need to be regenerated, they can be recreated from PostgreSQL data.

---

# Indexing Strategy

Each law article is indexed separately.

Each court decision summary is indexed separately.

Legal literature may be indexed by paragraph.

This improves semantic search precision.

---

# Scalability

The architecture supports large legal datasets.

Possible scaling strategies include:

horizontal scaling of Qdrant nodes
read replicas for PostgreSQL
distributed ingestion pipelines

This allows the platform to handle millions of legal documents.

---

# Long-Term Goal

The JPIS database architecture is designed to support a comprehensive legal knowledge system.

Future capabilities may include:

AI legal reasoning
legal precedent discovery
judicial analytics
legal concept mapping

This architecture forms the foundation of a national legal AI infrastructure.
