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

---
## Platform Context and Rationale

All videos referenced in this solution are **recorded and transcribed using Microsoft Teams Premium**. As a result, both the **video assets** and the **native transcripts** are already generated within the Microsoft 365 ecosystem and stored in SharePoint / Stream (on SharePoint).

Because of this operating model:

- Transcripts are **available by default**, consistently structured, and time‑aligned.
- Videos are stored in **Microsoft Stream (365)**, backed by SharePoint document libraries.
- Authentication, permissions, and compliance are already handled by Microsoft Entra ID.

### Why Copilot Studio Is the Preferred Platform

Given that the entire lifecycle (recording → transcription → storage) is managed by Microsoft 365, **Copilot Studio** is the preferred platform for building the retrieval agent because it:

- Natively integrates with **Teams, SharePoint, and Stream (365)**
- Allows tight coupling with **Power Automate** for deterministic execution
- Supports **enterprise governance**, auditing, and solution packaging
- Enables a clear separation between:
  - **Probabilistic understanding** (language model)
  - **Deterministic logic** (flows, filtering, URL generation)

This makes Copilot Studio the most natural and secure choice for building a production‑grade video discovery agent in environments where **Teams Premium recordings and transcripts are the system of record**.

> ⚠️ The agent does not generate or infer transcripts.  
> It operates **only on transcripts produced by Teams Premium** and stored in SharePoint, ensuring consistency and traceability across the system.
### Current Transcript Conversion Workflow (VTT → Excel)

At this stage, **there is no automated workflow** in place that converts VTT transcript files directly into Excel format.

Instead, the conversion process is performed **manually** using the open‑source tool **Subtitle Edit**.

#### Tool Used
- **Subtitle Edit** – desktop software for subtitle conversion  
  Download: https://www.nikse.dk/subtitleedit

---

#### Current Manual Process

1. Download the **VTT transcript file** produced by Microsoft Teams Premium.
2. Open the VTT file in **Subtitle Edit**.
3. Use Subtitle Edit to **export the transcript to CSV format**.
4. Open **Microsoft Excel**.
5. Import the CSV file into Excel.
6. Convert the imported data into an **Excel Table**.
7. Ensure the required columns exist:
   - `StartTimeMs`
   - `EndTimeMs`
   - `Text`
8. Save the Excel file using the **exact CamelCase name** of the corresponding video file.
9. Upload the Excel transcript to the **SharePoint Transcripts library**.

---

#### Important Notes

- This manual process is currently required to ensure:
  - Correct timestamp alignment
  - Clean and predictable data structure
  - Compatibility with deterministic Power Automate flows
- Automation of this pipeline (VTT → Excel) **may be added in the future**, but is **out of scope for the current implementation**.

✅ Until automation is introduced, **Subtitle Edit is the recommended and supported tool** for transcript conversion in this solution.

### Important Warnings and Current Limitations

#### ⚠️ Transcript Generation from Microsoft Teams

At present, **transcript file generation from Microsoft Teams recordings is also a manual process**.

- Transcript files (VTT) **are not automatically retrievable** via standard APIs.
- There is **no supported out‑of‑the‑box workflow** that programmatically fetches transcripts for all meetings in a tenant.

Automatic retrieval is possible **only** if all of the following conditions are met:

- You have **tenant‑level administrator privileges**
- You have explicitly granted:
  - **Microsoft Graph – Read all meetings and transcripts**
  - Organization‑wide **Application permissions** on Graph
- You are authorized to access transcripts for **all users and all meetings** in the tenant

> ⚠️ These permissions are **highly sensitive**, require security approval, and are **not commonly granted** in production enterprise environments.

---

#### Practical Implication for This Solution

Because of these restrictions:

- ✅ **Transcript download from Teams is currently manual**
- ✅ Users must explicitly download the `.vtt` file from the Teams / Stream interface
- ✅ The downloaded VTT file then enters the **manual conversion pipeline**:
  **VTT → CSV → Excel → SharePoint**

This design choice is intentional and ensures:
- Compliance with tenant security policies
- Clear data ownership and traceability
- No reliance on elevated Graph permissions

---

#### Future Consideration (Out of Scope)

If, in the future, the organization approves:
- Tenant‑wide Microsoft Graph access
- Automated access to meeting artifacts

Then a fully automated pipeline **may be designed**.  
Until then, **manual transcript retrieval and conversion is the supported and recommended approach** for this solution.

✅ The Copilot Studio agent **never attempts to fetch transcripts directly from Teams**  
✅ It operates only on **explicitly uploaded and validated transcript files**

### Downloadable Example Files (VTT → CSV → Excel)

To illustrate the **complete transcript preparation pipeline**, this repository includes the **same transcript in three stages**, downloadable directly from the repo.  
These examples should be used as the reference for both **format and structure**.

#### 1. Original Transcript (VTT – Source Format)

This file represents the **raw transcript** produced by **Microsoft Teams Premium**.

- **VTT file (source transcript)**  
  [prompt-engineering.vtt](https://github.com/mekorot/VideoAgentHub/blob/main/prompt-engineering.vtt)

Used for:
- Initial extraction
- Timestamp grounding
- Conversion to structured formats

---

#### 2. Intermediate Format (CSV)

This file demonstrates the **intermediate conversion step** between VTT and Excel.

- **CSV file (intermediate format)**  
  [prompt-engineering.csv](https://github.com/mekorot/VideoAgentHub/blob/main/prompt-engineering.csv)

Used for:
- Cleaning
- Normalization
- Preparing data for Excel ingestion

---

#### 3. Final Transcript (Excel – Production Format)

This is the **authoritative transcript format** consumed by the Copilot Studio agent.

- **Excel transcript (final format)**  
  [prompt-engineering.xlsx](https://github.com/mekorot/VideoAgentHub/blob/main/prompt-engineering.xlsx)

Excel requirements:
- Data saved as a **Table**
- Mandatory columns:
  - `StartTimeMs`
  - `EndTimeMs`
  - `Text`
- File name must exactly match the video file name (CamelCase)

---

✅ These three files together demonstrate the **end‑to‑end transcript lifecycle** expected by the solution:  

**Teams Premium → VTT → CSV → Excel (Table) → Copilot Studio Agent**

No other formats are supported in production.


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

Enriching transcript metadata improves discoverability, ranking quality, and future agent capabilities.

1. Use a language model to generate a **Hebrew title** based on the transcript content.
2. Store the Hebrew title:
   - As **SharePoint metadata** on the transcript file, or  
   - In an **associated SharePoint list** linked to the video/transcript.
3. *(Optional – Future Enhancement)* Add a **Brief / Summary field**:
   - Generate a short, neutral summary (1–2 sentences) describing the video content.
   - Store this value as a dedicated **`Brief` metadata field** in SharePoint.
   - **Index this field** once added (coordinate with the SharePoint team).

#### Why add and index a Brief field?

- Enables faster semantic narrowing before transcript‑level search
- Improves relevance scoring and ranking
- Allows future hybrid retrieval strategies (metadata + transcript)
- Reduces unnecessary transcript scanning for high‑level queries

> ℹ️ **Action item:**  
> Coordinate with the SharePoint team to:
> - Add a `Brief` site column
> - Attach it to the relevant libraries
> - Mark it as **indexed** to support scale and performance

This extension is not mandatory for the initial implementation but is **strongly recommended** for improving the agent’s effectiveness as the content library grows.

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
