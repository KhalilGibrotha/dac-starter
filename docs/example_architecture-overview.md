---
title: "Example Architecture Overview"
doc_type: overview
domain: platform
department: architecture
status: Draft
version: 0.1
date: 2026-07-13
author: "Author Name"
owner: "Team or Role"
---

# Overview

This sample document demonstrates the starter repo structure and a renderable
Mermaid diagram.

## System Context

```mermaid
flowchart LR
  Client["Client"] --> Gateway["Ingress Gateway"]
  Gateway --> Service["Application Service"]
  Service --> Database["Database"]
```

## Notes

Use this file as a first smoke test for linting and DOCX rendering.
