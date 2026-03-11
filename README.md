
# LEADTOOLS PACS Framework – Architecture & Setup Guide

**Version:** 1.0
**System:** DESKTOP-UAB1GD8\SQLEXPRESS
**Framework:** LEADTOOLS PACS Framework 64-bit

---

# Table of Contents

* [Overview](#overview)
* [Architecture](#architecture)
* [Database Layer](#database-layer)
* [Server / Devices Layer](#server--devices-layer)
* [Network Layer (DICOM)](#network-layer-dicom)
* [DICOM Commands](#dicom-commands)
* [Complete PACS Workflow](#complete-pacs-workflow)
* [Integration with Hospital Systems](#integration-with-hospital-systems)
* [Configuration](#configuration)
* [Testing](#testing)
* [Troubleshooting](#troubleshooting)

---

# Overview

This repository documents the **LEADTOOLS PACS Framework architecture** including:

* PACS storage database
* DICOM servers
* Network communication
* Query/Retrieve workflows
* Modality worklists
* Integration with hospital systems

The system is composed of **three main layers**:

1️⃣ Database Layer
2️⃣ Server / Device Layer
3️⃣ Network Layer (DICOM protocol)

---

# Architecture

```
                LEADTOOLS PACS SYSTEM

         ┌─────────────────────────────┐
         │        DATABASE LAYER       │
         │                             │
         │  Storage DB                 │
         │  Workstation DB             │
         │  Worklist DB                │
         └──────────────┬──────────────┘
                        │
         ┌──────────────▼──────────────┐
         │        SERVER LAYER         │
         │                             │
         │  Storage SCP  (Port 533)    │
         │  Workstation SCP (Port 334) │
         │  Worklist SCP  (Port 633)   │
         │  Demo SCP       (Port 333)  │
         └──────────────┬──────────────┘
                        │
         ┌──────────────▼──────────────┐
         │        NETWORK LAYER        │
         │                             │
         │  CT / MRI / X-Ray           │
         │  Workstations               │
         │  External PACS              │
         └─────────────────────────────┘
```

---

# Database Layer

The database stores **metadata only**.

Actual **DICOM image files are stored on disk**.

## Database Types

| Database       | Purpose                                |
| -------------- | -------------------------------------- |
| Storage DB     | Stores patient, study, series metadata |
| Workstation DB | Viewer preferences                     |
| Worklist DB    | Modality worklist schedules            |
| Demo DB        | Testing data                           |

---

## Database Schema

```
Patient
   │
   ▼
Study
   │
   ▼
Series
   │
   ▼
Instance (Image)
   │
   ▼
File System (DICOM File)
```

---

### Key Tables

| Table            | Purpose                  |
| ---------------- | ------------------------ |
| Patients         | Patient demographic data |
| Studies          | Imaging study metadata   |
| Series           | Image series information |
| Instances        | Individual images        |
| StorageLocations | File path mapping        |

---

# Server / Devices Layer

The PACS system runs several **DICOM SCP servers**.

## DICOM Servers

| Server             | AE Title        | Port | Function            |
| ------------------ | --------------- | ---- | ------------------- |
| Storage Server     | L23_PACS_SCP64  | 533  | Receives images     |
| Workstation Server | L23_WS_SERVER64 | 334  | Query/Retrieve      |
| Worklist Server    | L23_MWL_SCP64   | 633  | Modality Worklist   |
| Demo Server        | L23_SERVER64    | 333  | Development testing |

---

## Clients

| Client             | AE Title        | Port | Role                         |
| ------------------ | --------------- | ---- | ---------------------------- |
| Main Client        | L23_CLIENT64    | 1033 | Send images to external PACS |
| Workstation Client | L23_WS_CLIENT64 | 1034 | Query PACS                   |

---

# Network Layer (DICOM)

Communication happens through **DICOM DIMSE protocol** over **TCP/IP**.

Before communication begins, an **association** must be established.

```
SCU (Client) ---- A-ASSOCIATE ----> SCP (Server)
SCU <--- A-ASSOCIATE-AC ---------- SCP
```

After association is established, DICOM commands can run.

---

# DICOM Commands

## C-ECHO (Verification)

Used to test connectivity.

```
SCU → C-ECHO → SCP
SCP → C-ECHO Response → SCU
```

Status `0000` = Success

---

## C-STORE (Image Upload)

Used to send images to PACS.

```
Modality → Storage SCP
```

Flow:

```
SCU → C-STORE Request
SCP → C-STORE Response
```

---

## C-FIND (Query)

Used to search for studies.

Query levels:

| Level    | Description          |
| -------- | -------------------- |
| Patient  | Search patient       |
| Study    | Search imaging study |
| Series   | Search image series  |
| Instance | Search image         |

Example query:

```
PatientName = *DOE*
StudyDate = TODAY
```

---

## C-MOVE (Retrieve)

Retrieves images through **third party transfer**.

```
Workstation → PACS
PACS → sends images → workstation
```

Requires **two connections**.

---

## C-GET (Retrieve Alternative)

Images returned **on the same connection**.

```
Workstation → PACS
PACS → returns images
```

---

### C-MOVE vs C-GET

| Feature           | C-MOVE | C-GET |
| ----------------- | ------ | ----- |
| Connections       | 2      | 1     |
| Firewall friendly | No     | Yes   |
| Used in hospitals | Yes    | Rare  |

---

# Complete PACS Workflow

## Step 1 — Modality Worklist

Modality queries scheduled patients.

```
Modality → Worklist Server
C-FIND
```

Result: list of scheduled exams.

---

## Step 2 — Image Acquisition

Scanner sends images to PACS.

```
CT / MRI → Storage SCP
C-STORE
```

Images stored in:

```
D:\PACS_Images\
```

Metadata stored in database.

---

## Step 3 — Query

Radiologist searches studies.

```
Workstation → Workstation SCP
C-FIND
```

---

## Step 4 — Retrieve

Images downloaded to workstation.

```
Workstation → PACS
C-MOVE / C-GET
```

---

## Step 5 — Verification

Admin checks system connectivity.

```
Admin → Demo Server
C-ECHO
```

---

# Integration with Hospital Systems

```
HIS  ↔  RIS  ↔  PACS
                │
                │
         ┌──────▼──────┐
         │ Worklist    │
         │ Server      │
         └──────┬──────┘
                │
                ▼
          CT / MRI
```

---

# Configuration

## SQL Server Connection

### Windows Authentication

```
Server=DESKTOP-UAB1GD8\SQLEXPRESS;
Database=PACSStorageDB;
Integrated Security=true;
```

---

### SQL Authentication

```
Server=DESKTOP-UAB1GD8\SQLEXPRESS;
Database=PACSStorageDB;
User Id=sa;
Password=YourPassword;
```

---

# Ports

| Service         | Port |
| --------------- | ---- |
| Storage SCP     | 533  |
| Workstation SCP | 334  |
| Worklist SCP    | 633  |
| Demo SCP        | 333  |
| Broker Host     | 8033 |
| SQL Server      | 1433 |

---

# Testing

## Check SQL Server

```
Get-Service MSSQL$SQLEXPRESS
```

---

## Check Ports

```
netstat -an | findstr "533 334 333 633"
```

---

## Test C-ECHO

```
echoscu DESKTOP-UAB1GD8 533 -aec L23_PACS_SCP64
```

Expected:

```
Echo Response Success
```

---

## Test C-STORE

```
storescu DESKTOP-UAB1GD8 533 sample.dcm
```

---

## Test Query

```
findscu DESKTOP-UAB1GD8 334 -aec L23_WS_SERVER64
```

---

# Troubleshooting

| Problem              | Cause         | Fix              |
| -------------------- | ------------- | ---------------- |
| Association rejected | AE mismatch   | Verify AE title  |
| C-MOVE fails         | Firewall      | Open ports       |
| C-STORE fails        | Disk full     | Check storage    |
| Query empty          | DB issue      | Check SQL server |
| Images not visible   | Codec missing | Install codec    |

---

# Resources

* LEADTOOLS PACS Documentation
* DICOM Standard
* DCMTK Tools
* Orthanc DICOM Guide

---

# License

Internal documentation for **LEADTOOLS PACS deployment**.

---

If you want, I can also improve this README further by adding:

* **📊 PACS Architecture diagram (professional SVG)**
* **🧠 DICOM tag explanation**
* **🖼 DICOM thumbnail generation flow** (related to your recent work)
* **⚙ Leadtools PACS configuration guide**
* **🚀 How to deploy PACS server locally**

which would make this **look like a professional GitHub repository README**.
