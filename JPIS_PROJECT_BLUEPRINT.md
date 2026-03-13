# JPIS – Project Blueprint

JPIS (Judicial Public Information System) is an AI-powered legal information platform designed for Bosnia and Herzegovina.

The goal of JPIS is to create a modern legal infrastructure that combines:

* legal databases
* artificial intelligence
* legal analytics
* open legal data

JPIS aims to become a centralized platform for accessing laws, court decisions, legal literature, and legal analytics.

The system is designed as a modular architecture composed of several independent repositories.

---

# Core Vision

JPIS is not only a legal database.

It is a **legal intelligence platform** capable of:

AI-assisted legal research
semantic legal search
linking laws with court decisions
legal analytics and research
public legal transparency

The platform may eventually support:

judicial digitalization
legal research tools
academic legal analysis
government legal infrastructure

---

# System Modules

JPIS consists of the following modules:

Law Module
Court Decisions Module
Legal Literature Module
AI Assistant Module
Legal Analytics Module
Open Data Module

Each module corresponds to a domain within the legal system.

Example domains:

/zakoni
/presude
/analitika
/biblioteka
/data
/ai

---

# Repository Architecture

The system is organized into five repositories.

jpis-frontend
jpis-backend
jpis-ingestion
jpis-datasets
jpis-docs

Each repository has a specific role.

---

# jpis-frontend

Purpose:

User interface for accessing the JPIS system.

Technology stack:

Next.js
TypeScript
Tailwind CSS

Deployment:

Vercel

Responsibilities:

AI chat interface
law browsing interface
court decision browsing
legal search UI
analytics dashboards

Structure example:

src/app

/zakoni
/presude
/analitika
/biblioteka
/data
/ai

The frontend communicates with the backend through REST APIs.

Example API endpoint:

/api/ai/chat

---

# jpis-backend

Purpose:

Main API and AI orchestration layer.

Technology:

Medusa v2 (custom modules)

Medusa is used as a modular backend framework capable of hosting domain modules.

Example modules:

modules/laws
modules/court-decisions
modules/ai
modules/analytics

Responsibilities:

legal data APIs
AI request processing
vector search queries
knowledge graph queries

Example API routes:

/api/laws/search
/api/cases/search
/api/ai/chat

Deployment options:

Render
Railway
VPS
AWS

---

# jpis-ingestion

Purpose:

Processing and importing legal data into the system.

This repository contains scripts that:

parse legal documents
extract structured law articles
parse court decisions
generate vector embeddings
load data into the database

Example structure:

src

/laws
/court-decisions
/literature
/parsers
/embeddings

Example ingestion scripts:

ingest-laws.ts
ingest-cases.ts

This repository does not require permanent deployment.

It can be run:

locally
via GitHub Actions
via scheduled jobs

---

# jpis-datasets

Purpose:

Raw legal data repository.

This repository contains structured datasets used by the ingestion pipeline.

Example structure:

laws/

criminal_code_bih/

law.json
articles/

001.json
002.json
239.json

court-decisions/

case_123_2020.json

literature/

legal_commentary.json

Each law article is stored as a separate JSON document.

This structure allows precise AI indexing.

---

# jpis-docs

Purpose:

System documentation repository.

Contains all architecture, design, and technical documentation.

Example contents:

system architecture
AI pipeline documentation
legal ontology
knowledge graph design
data ingestion guidelines
security and ethics

Documentation may later be published as a documentation website.

Possible tool:

Docusaurus

---

# System Data Flow

The JPIS system follows a layered architecture.

Frontend
↓
Backend API
↓
Vector search and knowledge graph
↓
Legal dataset

AI query flow:

User question
↓
embedding generation
↓
vector search
↓
knowledge graph expansion
↓
prompt construction
↓
AI response generation

This architecture is known as Retrieval Augmented Generation (RAG).

---

# Vector Search Strategy

Vector search is used to retrieve relevant legal documents.

Documents indexed:

law articles
court decisions
legal literature

Recommended vector database:

Qdrant

Alternative options:

pgvector (PostgreSQL extension)
Weaviate

Qdrant is recommended due to performance and simplicity.

---

# Knowledge Graph

The legal knowledge graph represents relationships between legal entities.

Examples of relationships:

LawArticle → interpreted_by → CourtDecision

CourtDecision → establishes → LegalPrinciple

LawArticle → discussed_in → LegalLiterature

This allows AI to understand legal reasoning rather than only text similarity.

---

# Legal Ontology

The ontology defines core legal concepts.

Examples:

Fraud
Intent
Property Damage
Appeal
Jurisdiction

Concepts link legal texts and judicial interpretation.

Example relationship:

Concept: Fraud
↓
Law Article 239 – Criminal Code BiH

---

# AI Assistant

The AI assistant answers legal questions based on retrieved legal sources.

Example question:

Which court decisions interpret Article 239 of the Criminal Code of BiH?

The system retrieves:

law article
relevant court decisions
legal principles

The AI then generates an explanation with citations.

The AI system functions as a **legal research assistant**, not as a legal decision maker.

---

# Security and Ethics

The system follows ethical AI principles.

AI must:

avoid hallucinated legal information
cite legal sources
remain transparent about data sources

AI responses are informational and do not replace legal professionals.

---

# Long-Term Vision

JPIS aims to become a comprehensive legal information infrastructure for Bosnia and Herzegovina.

Potential future capabilities include:

AI legal reasoning
legal analytics
judicial transparency tools
open legal research datasets

The system is designed to grow into a national legal information platform.
