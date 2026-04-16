# The Peoples App: Digital Transformation for The People's Guide

## 1. Our Mission
Project Identity
- **Web:** [peoplesapp.org](https://peoplesapp.org) 
- **Mission:** Powering a mobile ecosystem with 20 years of People's Guide research.

## 2. Strategic Activation (The Phase-0 Pivot)
This project represents the transition from static print assets to a resilient, data-driven infrastructure. We recognize HALA’s significant annual impact and the vital role of "The People's Guide." Our objective is to protect that legacy by ensuring food security information is:
- **Instant:** Searchable in milliseconds via a modern mobile interface.
- **Always Available:** Accessible in "data deserts" via offline-first architecture.
- **Sustainable:** Maintained via a low-overhead, AI-augmented data pipeline.

## 3. Engineering Stewardship
Social-impact technology must meet enterprise-grade standards. We utilize a **Staged Governance** model to ensure 100% data integrity, protecting HALA’s reputation and providing the Board with a transparent, high-ROI digital asset.

---

## Setup & Installation (The "Single Door" Policy)

To maintain synchronization between the public application and private strategy, we utilize a **Recursive Submodule** architecture. **Do not maintain separate standalone clones of the internal repositories.**

### 1. Initial Clone
You **must** clone the repository recursively to "hydrate" the internal documentation and engine:
```bash
# 1. Clone the repository recursively
git clone --recursive git@github.com:The-Peoples-App/ppls-app.git
cd ppls-app

# 2. Permissions: Make the HALA toolset executable
chmod +x bin/*

# 3. Environment Sync: Initialize the internal core
./bin/hsetup
```

### 2. Environment Sync
If you have already cloned the repo or need to sync the latest internal strategy:
```bash
./bin/hsetup
```
*(This script runs the recursive submodule update and initializes local protocols.)*

### 3. Deployment & Push Workflow
To prevent "Broken Pointer" errors and ensure private data remains air-gapped, always use the **HALA Push Auditor** from the root:
```bash
./bin/hpush
```

## Project Governance
- **Operations:** See `docs/COMMITS.md` for the "Inside-Out" commit protocol.
- **Accessibility:** Our commitment to WCAG 2.1 AA is detailed in `docs/A11Y_CONFORMANCE.md`.
- **Architecture:** Explore our decision records in `docs/architecture/`.


---
*Developed by the HALA Technical Lead & Design Partners.*

