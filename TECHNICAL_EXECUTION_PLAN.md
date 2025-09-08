# autovibe Technical Execution Plan - ORM & Data Architecture

## 🎯 EXECUTIVE SUMMARY

This document focuses on the **Object-Relational Mapping (ORM) architecture and data modeling decisions** for the autovibe Intelligent Machine orchestration system using **Protobuf-based type design**.

---

## 🗄️ ORM ARCHITECTURE & DATA MODELING

### Technology Stack - Data Layer
```yaml
Database: (ORM-abstracted, database-agnostic)
Database Service: Go microservice using Ent ORM (github.com/ent/ent)
Communication: gRPC with Protobuf between Python services and DB service
Type System: Protobuf for all structured data design
Schema Management: Buf (buf.build) for Protobuf schema management and code generation
Python Package Management: Poetry for dependency management and packaging
Migration Tool: Ent-integrated migration system
Connection Pool: Go service manages database connections
```

### ORM Design Philosophy

**Core Principles:**
- **Protobuf Type System**: Structured type definitions with schema evolution
- **Temporal Data Tracking**: Created/updated timestamps on all mutable entities
- **Type-Safe Serialization**: Cross-language compatibility for machine communication
- **Relationship Management**: Foreign keys with cascading behavior carefully designed
- **Performance Strategy**: Partitioning and indexing for high-volume data
---

## 🏗️ TECHNOLOGY ARCHITECTURE DECISIONS

### Communication Protocols
- **HTTP REST APIs**: For Intelligent Machine agents (Claude Code, Gemini CLI, Aider, QwenCoder and others, only Claude Code for MVP)
- **gRPC Internal Services**: For internal communication, telemetry collection, and database operations
- **Database Service**: Dedicated Go microservice using Ent ORM, accessed via gRPC by Python services
- **Data Format**: Protobuf for all structured data (storage, network, APIs, database operations)
- **Schema Management**: Buf (buf.build) for Protobuf schema versioning and code generation
- **Message Patterns**: Actor Model with asynchronous message-passing

---

## 🌐 COMMUNICATION STRATEGY

### External APIs - Machine Agents
- **Protocol**: HTTP REST with JSON
- **Usage**: Claude Code, Aider, QwenCoder agent communication

### Internal Services - Hub Components  
- **Protocol**: gRPC with binary Protobuf
- **Authentication**: mTLS certificates
- **Services**: Hub replication, resource monitoring, VM orchestration, database operations
- **Database Service**: Dedicated Go microservice with Ent ORM handling all database operations
- **Usage**: High-frequency internal communication, streaming data, database access

### Protocol Selection Logic
- **gRPC**: Internal services, streaming, high-frequency updates, binary efficiency, database operations
- **Go Database Service**: Centralized database operations using Ent ORM, called by Python services via gRPC

---

### Operating System Strategy
```yaml
MVP Base OS: Debian minimal 
Rationale: Lightweight, stable, well-supported for VM deployment
Package Management: APT with carefully curated package selection
```

### Intelligent Machine Types

**Conceptual Foundation**: Intelligent Machines implement the [Actor Model](https://en.wikipedia.org/wiki/Actor_model) - each machine is an independent actor that:
- Processes messages asynchronously 
- Maintains private state (checkpoints, progress)
- Communicates only through message passing (HTTP/gRPC)
- Can spawn other actors (sub-machines, Hub coordination)
- Provides fault isolation and concurrent execution

```yaml
# MVP: Full VM-based Intelligent Machines  
Standard Machines:
  - Claude Code, Aider, QwenCoder agents
  - Full VM isolation with Proxmox
  - Complete checkpoint/restore capability
  - Resource constraints: 4-8GB RAM per VM
  - Use case: Complex, long-running coding tasks
  - Actor Model: Each VM is an isolated actor with message-based communication

Hub Machines:
  - Central orchestration and coordination
  - Full VM with extended resources
  - Manages project states and checkpoints
  - Resource constraints: 8-16GB RAM
  - Use case: Project coordination and multi-agent workflows
  - Actor Model: Supervisor actor managing worker machine actors
```

### Technology-Agnostic Architecture Principles
- **Hypervisor-Agnostic**: Design works with Proxmox, VMware, KVM/libvirt
- **Evolution-Ready**: Architecture supports Full VM → Unikernel transition path
- **Cloud-Portable**: Can deploy on AWS, GCP, Azure with minimal changes  
- **Storage-Flexible**: Works with ZFS, Btrfs, or cloud block storage
- **Database Independence**: ORM abstraction enables database portability

---


## 📋 TECHNOLOGY SUMMARY

This execution plan focuses on high-level architecture decisions for the autovibe system:

### Core Technology Choices
- **Communication**: HTTP REST (For agent IMs) + gRPC (internal) 
- **Data Format**: Protobuf for all structured data
- **Schema Management**: Buf (buf.build) for Protobuf tooling and code generation
- **Python Package Management**: Poetry for dependency management and packaging
- **Architecture**: Actor Model with message-passing
- **Database**: Go microservice with Ent ORM, accessed via gRPC from Python services
- **Language Split**: Go for database operations, Python for business logic and AI agents
- **Performance**: Indexing, partitioning, materialized views
- **Infrastructure**: VM-based (Proxmox) with unikernel evolution path