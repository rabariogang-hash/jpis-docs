# JPIS Infrastructure Architecture

This document defines the infrastructure and deployment architecture of the JPIS system.

JPIS is composed of several independent services deployed across cloud infrastructure.
Each service is responsible for a specific layer of the system.

The architecture follows a modern microservice pattern suitable for AI-driven applications.

---

# System Overview

JPIS consists of the following repositories:

jpis-frontend
jpis-backend
jpis-ingestion
jpis-datasets
jpis-docs

These repositories correspond to separate components deployed across different environments.

---

# Infrastructure Layers

The JPIS platform is composed of the following layers:

Frontend Layer
API Layer
Data Layer
AI Infrastructure Layer

Each layer is deployed using specific technologies.

---

# Frontend Layer

Repository:

jpis-frontend

Technology:

Next.js
TypeScript
Tailwind CSS

Deployment platform:

Vercel

The frontend provides the main user interface for the system.

Features include:

AI chat interface
law browsing
court decision browsing
legal search
analytics dashboards

Example domain:

web.jpis.ba

---

# Backend API Layer

Repository:

jpis-backend

Technology:

Medusa v2 (custom modules)

Responsibilities:

legal API endpoints
AI orchestration
vector search queries
knowledge graph queries

Example endpoints:

/api/laws/search
/api/cases/search
/api/ai/chat

Deployment platform:

Render

Example domain:

api.jpis.ba

---

# Data Layer

JPIS stores structured legal data in PostgreSQL.

Recommended provider:

Neon (serverless PostgreSQL)

Stored data includes:

laws
law articles
court decisions
legal literature
legal ontology concepts

PostgreSQL acts as the primary data source for the system.

---

# Vector Database Layer

Vector embeddings are stored in a dedicated vector database.

Recommended technology:

Qdrant

Qdrant stores embeddings for:

law articles
court decisions
legal literature

These embeddings allow semantic search across legal documents.

Deployment:

Docker container

Example deployment options:

Render
VPS server

---

# AI Retrieval Pipeline

The AI system uses a Retrieval Augmented Generation (RAG) pipeline.

Processing steps:

User question
↓
embedding generation
↓
vector search in Qdrant
↓
retrieve document IDs
↓
fetch documents from PostgreSQL
↓
construct AI prompt
↓
generate AI response

This architecture ensures the AI system relies on verified legal sources.

---

# Ingestion Pipeline

Repository:

jpis-ingestion

Purpose:

Process raw legal data and insert it into the system.

Responsibilities include:

parsing legal documents
segmenting law articles
extracting metadata
generating embeddings
loading data into PostgreSQL and Qdrant

The ingestion service does not require permanent deployment.

It can run:

locally
via GitHub Actions
via scheduled jobs

---

# Dataset Repository

Repository:

jpis-datasets

Purpose:

Store raw legal datasets used by the ingestion pipeline.

Data includes:

laws
court decisions
legal literature

Example structure:

laws/

criminal_code_bih/

law.json
articles/

001.json
239.json

Each article is stored as a separate JSON file.

---

# Documentation Repository

Repository:

jpis-docs

Purpose:

Store all architecture and system documentation.

This includes:

system architecture
AI pipeline design
legal ontology
database structure
infrastructure plans

Documentation serves as the reference blueprint for the entire project.

---

# Deployment Architecture

JPIS services are deployed as follows:

Frontend → Vercel

Backend API → Render

PostgreSQL → Neon

Vector Database → Qdrant (Docker container)

This architecture allows each component to scale independently.

---

# Domain Structure

The platform may use a modular domain architecture.

Example domains:

web.jpis.ba
api.jpis.ba
ai.jpis.ba
data.jpis.ba

Future modules may include:

zakoni.jpis.ba
presude.jpis.ba

---

# Continuous Integration

Each repository may include CI pipelines.

Possible CI tasks include:

build verification
type checking
linting
automated testing

GitHub Actions can be used for CI workflows.

---

# Scalability

The infrastructure is designed to support large legal datasets.

Possible scaling strategies:

multiple vector search nodes
database read replicas
distributed ingestion pipelines

This architecture ensures long-term system growth.

---

# Security

Security considerations include:

secured API endpoints
authentication and authorization
monitoring system activity
data integrity checks

Sensitive legal data must be handled with care.

---

# Long-Term Vision

The JPIS infrastructure is designed to support a national legal information platform.

Future capabilities may include:

AI legal research systems
judicial analytics
open legal data access
digital legal infrastructure for Bosnia and Herzegovina.
