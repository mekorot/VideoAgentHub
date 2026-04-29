# Video Retrieval Agent (Copilot Studio)
## Introduction

This repository presents a **production‑grade reference implementation** for building a **Copilot Studio agent** that retrieves precise Microsoft Stream (365) video segments based on **Excel‑based transcripts stored in SharePoint**.

The solution is intentionally designed around a strict separation of responsibilities:

- **Probabilistic reasoning** (semantic understanding, multilingual interpretation, intent detection) is handled by a language model.
- **Deterministic execution** (video resolution, timestamp calculation, Stream URL generation, and response formatting) is handled by Power Automate flows.

All components are packaged within a **single Power Platform Solution** and follow consistent naming conventions, deterministic data contracts, and **transcript‑only answer policies**. This architecture ensures the system is **auditable, reproducible, and enterprise‑safe**, while still leveraging modern AI capabilities where they add real value.

The repository includes:
- Reference architecture and setup guidelines  
- Sample transcript files (Excel and CSV)  
- Deterministic Power Automate flow definitions  
- Copilot Studio system instructions  

This implementation is well‑suited for organizations that require **accurate video discovery**, strict governance, and predictable behavior when deploying Copilot Studio agents at scale.

## Steps to Produce

Follow these steps to implement and operationalize the Video Retrieval Agent end‑to‑end.  
All components **must be created inside a single Power Platform Solution**.

---

### 1. Prepare SharePoint Structure

1. Create a **dedicated SharePoint site** with an English URL.
2. Create a document library named **`Videos`** for storing video files.
3. Create a document library named **`Transcripts`** for storing transcript files (Excel).
4. Enforce **English CamelCase naming** for all files.
   - Video and transcript file names must be **identical**.

---

#### CamelCase File Naming Examples

All video and transcript files **must use the same CamelCase name**, written in English, with no spaces or special characters. This is critical for deterministic matching between SharePoint, Excel transcripts, and Power Automate flows.

##### ✅ Correct CamelCase Examples

| Purpose | File Name |
|------|----------|
| Video file | `WaterConsumptionDashboard.mp4` |
| Transcript file | `WaterConsumptionDashboard.xlsx` |
| Video file | `LabSafetyTraining.mp4` |
| Transcript file | `LabSafetyTraining.xlsx` |
| Video file | `AIAutomationIntroduction.mp4` |
| Transcript file | `AIAutomationIntroduction.xlsx` |

---

##### ❌ Incorrect Examples (Do Not Use)

- `water consumption dashboard.mp4` ❌ (spaces, lowercase)
- `Water_Consumption_Dashboard.mp4` ❌ (underscores)
- `Water-Consumption-Dashboard.mp4` ❌ (hyphens)
- `לוח-צריכת-מים.mp4` ❌ (non‑English)
- `WaterConsumptionDashboard_v2.mp4` ❌ (versioning in filename)

---

##### ✅ CamelCase Rules Summary

- Each word starts with a **capital letter**
- No spaces, underscores, dashes, or symbols
- No version numbers or dates
- File extension preserved (`.mp4`, `.xlsx`)
- **Video and transcript file names must match exactly**

Following these conventions guarantees reliable SharePoint filtering, deterministic transcript resolution, and correct Stream 365 timestamped link generation.

---

#### Example 1 – Meeting Recording

**Original file name (downloaded):**

---

### 3. Generate and Structure Transcripts

1. Download the transcript file (usually `.vtt`).
2. Convert the transcript:
   - `VTT → CSV → Excel`
3. In Excel:
   - Convert the data into a **Table**
   - Required columns:
     - `StartTimeMs`
     - `EndTimeMs`
     - `Text`
4. Save the Excel file using the **exact same CamelCase name** as the video.
5. Upload the Excel file to the **Transcripts** library.

---

### 4. Enrich Transcript Metadata (Optional but Recommended)

1. Use a language model to generate a **Hebrew title** based on the transcript.
2. Store the Hebrew title:
   - As SharePoint metadata, or
   - In an associated SharePoint list

---

### 5. Create Power Platform Solution

1. In the Power Platform development environment, create a solution:

```markdown
# 🎥 Video Retrieval Agent (Copilot Studio)

A deterministic Copilot Studio agent for locating **Microsoft Stream (365) video segments** based on **Excel‑based transcripts**, using SharePoint, Power Automate, and GPT‑4.1.

This repository documents the **architecture, conventions, and setup steps** required to build an enterprise‑grade video discovery agent with auditable and reproducible behavior.

---

## 🚀 Overview

The agent enables users to:
- Search video content via **structured transcripts**
- Retrieve **precise Stream links with timestamps**
- Receive **clear Markdown responses**, including Hebrew titles when applicable

Key design principles:
- ✅ Deterministic execution for URLs and timestamps  
- ✅ Probabilistic reasoning only for semantic understanding  
- ✅ All assets contained within **one Power Platform Solution**

---

## 🧱 Architecture Components

- **Copilot Studio** – Conversational agent
- **SharePoint Online** – Video & transcript storage
- **Power Automate** – Deterministic logic (filtering, URL generation, formatting)
- **Excel Online** – Transcript storage format (table‑based)
- **GPT‑4.1** – Language model for understanding & enrichment

---

## 🗂️ SharePoint Structure

### 1. Dedicated Site
Create a SharePoint site with an **English URL** for clean paths.

Example:
```

/sites/VideoKnowledgeHub

```

---

### 2. Videos Library
- Library name: `Videos`
- Purpose: Store video files (MP4 / Stream)
- Naming convention: **English CamelCase**

Example:
```

WaterConsumptionDashboard.mp4

```

---

### 3. Transcripts Library
- Library name: `Transcripts`
- Purpose: Store Excel transcripts
- File name **must exactly match** the video file name

Example:
```

WaterConsumptionDashboard.xlsx

```

---

## 📝 Transcript Preparation Pipeline

1. Download video file  
2. Rename to **English CamelCase**  
3. Upload to `Videos` library  
4. Download transcript file (VTT)  
5. Convert: `VTT → CSV → Excel`  
6. In Excel:
   - Convert data to a **Table**
   - Required columns:
     - `StartTimeMs`
     - `EndTimeMs`
     - `Text`
7. Upload Excel file to `Transcripts` library  
8. (Optional) Use a language model to generate a **Hebrew title** and store it as metadata

> 🔁 This pipeline can be fully automated with **Power Automate** or external scripts.

---

## 🧩 Power Platform Solution

Create a dedicated solution in the **Power Platform Development Environment**:

```

VideoRetrievalSolution

```

✅ **All components below must reside in this solution**:
- Copilot Studio agent
- Power Automate flows
- Connections
- Custom logic

---

## 🤖 Copilot Studio Agent

### Agent Scope
- Operates **only on transcript data**
- Does **not infer or guess timestamps**
- Delegates deterministic work to flows

### System Instructions
- Enforce transcript‑only answers
- Require timestamp & URL generation via flows
- Prohibit manual URL construction

📎 See: `prompt-engineering.txt`

---

## 🧠 Recommended Model

```

GPT‑4.1

````

Why:
- High precision with structured data
- Strong multilingual performance (English + Hebrew)
- Better determinism boundaries

---

## 🔁 Deterministic Power Automate Flows

All flows are included in the same solution.

### Flow 1 – Resolve Video Metadata
- Filters SharePoint items by **CamelCase file name**
- Returns:
  - File name
  - Item ID
  - Hebrew display title (if available)

---

### Flow 2 – Generate Stream URL with Timestamp
- Input: Video file + timestamp (milliseconds)
- Logic:
  - Convert milliseconds → seconds
  - Build navigation object
  - Base64‑encode navigation payload
  - Append `nav` parameter to Stream URL

📎 See: `generate-video-url.json`

---

### Flow 3 – Markdown Response Formatter
Formats a consistent, readable response:

```markdown
### Short Answer (Hebrew)

> Transcribed quote

[Stream Video Link]
````

***

## 🔄 End‑to‑End Flow

1.  User submits a question
2.  Agent searches Excel transcripts
3.  Exact transcript row is selected
4.  Timestamp (ms) extracted
5.  Video metadata resolved
6.  Stream URL generated
7.  Markdown response returned

***

## ✅ Validation & Testing

### Probabilistic Validation

*   Open‑ended questions
*   Validate semantic accuracy

### Deterministic Validation

*   Known timestamp questions
*   Validate:
    *   No hallucinations
    *   Exact timestamp jump
    *   Correct video resolution

***

## 🔐 Architectural Rules (Mandatory)

✅ One solution only  
✅ English + CamelCase naming  
✅ Transcript‑based answers  
✅ Deterministic URL generation

❌ No inferred timestamps  
❌ No manual Stream link creation

***

## 📎 Repository Appendix

*   `prompt-engineering.txt` – Agent system instructions
*   `generate-video-url.json` – Deterministic Stream URL flow
*   Sample transcript files (CSV / Excel)
*   Reference deterministic assistant guide

***

## 📌 Optional Extensions

*   Mermaid / Visio architecture diagram
*   CI‑validated transcript ingestion
*   Automated VTT → Excel pipeline
*   Executive summary version

***

Built for **enterprise‑grade Copilot Studio deployments** 🏗️  
Deterministic where it matters. Probabilistic where it adds value.

```
```

## Sample Files

This repository includes sample files demonstrating the expected formats.

### Transcript Samples
- Excel transcript: ./samples/WaterConsumptionDashboardGuideAccessFilteringExport.xlsx
- CSV transcript: ./samples/WaterConsumptionDashboardGuideAccessFilteringExport.csv

### Agent & Flow Samples
- Copilot Studio system prompt: ./samples/prompt-engineering.txt
- Stream URL generation flow: ./samples/generate-video-url.json
- Deterministic architecture guide: ./samples/Deterministic_AI_Assistant_Guide.md

## Suggested Repository Structure

```text
.
├─ README.md
├─ samples/
│  ├─ WaterConsumptionDashboardGuideAccessFilteringExport.xlsx
│  ├─ WaterConsumptionDashboardGuideAccessFilteringExport.csv
│  ├─ prompt-engineering.txt
│  ├─ generate-video-url.json
│  └─ Deterministic_AI_Assistant_Guide.md
└─ solution/
   └─ VideoRetrievalSolution/
```
