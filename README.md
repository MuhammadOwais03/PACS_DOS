I'll create a comprehensive GitHub README format documentation for your LEADTOOLS PACS Framework setup. This will cover everything from databases to network layer with a complete test flow.

```markdown
# LEADTOOLS PACS Framework - Complete Architecture Guide

> **Version:** 1.0  
> **Author:** [Your Name]  
> **Date:** 2024  
> **System:** DESKTOP-UAB1GD8\SQLEXPRESS (SQL Server Express)  
> **Framework:** LEADTOOLS PACS Framework 64-bit

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture Diagram](#architecture-diagram)
- [Database Layer](#database-layer)
- [Server/Devices Layer](#serverdevices-layer)
- [Network Layer (DICOM Protocol)](#network-layer-dicom-protocol)
- [DICOM Command Reference](#dicom-command-reference)
- [Complete Test Flow](#complete-test-flow)
- [PACS Integration](#pacs-integration)
- [Configuration Reference](#configuration-reference)
- [Troubleshooting](#troubleshooting)

---

## Overview

This document describes the complete architecture of the LEADTOOLS PACS Framework deployment, including:
- **Database Layer**: SQL Server storage for metadata and configuration
- **Server Layer**: DICOM services (SCP) for medical device communication
- **Network Layer**: DICOM protocol implementation (DIMSE)
- **Integration**: Connection with modalities, workstations, and external PACS

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         LEADTOOLS PACS FRAMEWORK                            │
│                         Complete System Architecture                        │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                              DATABASE LAYER                                 │
│                           (SQL Server Express)                              │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐             │
│  │  Storage DB     │  │  Workstation DB │  │  Worklist DB    │             │
│  │  (PACSStorage)  │  │  (Viewer Config)│  │  (MWL Schedule) │             │
│  │                 │  │                 │  │                 │             │
│  │ • Patient Data  │  │ • User Settings │  │ • Appointments  │             │
│  │ • Study Metadata│  │ • Annotations   │  │ • Procedure     │             │
│  │ • Series Info   │  │ • Layouts       │  │   Steps         │             │
│  │ • Image Headers │  │ • Preferences   │  │ • Modality      │             │
│  │ • Audit Logs    │  │ • Query History │  │   Worklists     │             │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘             │
└───────────┼────────────────────┼────────────────────┼──────────────────────┘
            │                    │                    │
            └────────────────────┼────────────────────┘
                                 │
┌────────────────────────────────▼────────────────────────────────────────────┐
│                              SERVER LAYER                                   │
│                         (DICOM SCP Services)                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      STORAGE SERVER (SCP)                           │   │
│  │  AE Title: L23_PACS_SCP64    │    Port: 533                        │   │
│  │                                                                     │   │
│  │  Services:  C-STORE (Receive images from modalities)                │   │
│  │             C-ECHO  (Connectivity verification)                     │   │
│  │                                                                     │   │
│  │  Receives:  CT, MRI, X-Ray, Ultrasound, etc.                        │   │
│  │  Stores To: File System + Storage DB                                │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                   WORKSTATION SERVER (SCP)                          │   │
│  │  AE Title: L23_WS_SERVER64   │    Port: 334                        │   │
│  │                                                                     │   │
│  │  Services:  C-FIND  (Query for studies/patients)                    │   │
│  │             C-MOVE  (Retrieve images to workstation)                │   │
│  │             C-GET   (Alternative retrieve - direct download)        │   │
│  │                                                                     │   │
│  │  Clients:   Radiologist Workstations, Review Stations               │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                     WORKLIST SERVER (SCP)                           │   │
│  │  AE Title: L23_MWL_SCP64     │    Port: 633                        │   │
│  │                                                                     │   │
│  │  Services:  C-FIND  (Modality Worklist - MWL)                       │   │
│  │                                                                     │   │
│  │  Purpose:   Provide scheduled exams to modalities                   │   │
│  │  Clients:   CT Scanners, MRI Machines, Ultrasound Equipment         │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      DEMO SERVER (SCP)                              │   │
│  │  AE Title: L23_SERVER64      │    Port: 333                        │   │
│  │                                                                     │   │
│  │  Services:  C-ECHO, C-STORE, C-FIND (Testing/Development)           │   │
│  │                                                                     │   │
│  │  Purpose:   Testing and demonstration of DICOM services             │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    BROKER HOST ADD-IN                               │   │
│  │  Port: 8033                                                         │   │
│  │                                                                     │   │
│  │  Purpose:   Internal communication between LEADTOOLS services       │   │
│  │             (Service orchestration and message routing)             │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                           NETWORK LAYER                                     │
│                        (DICOM DIMSE Protocol)                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐    C-STORE     ┌─────────────┐    SQL Queries   ┌────────┐ │
│  │  MODALITY   │───────────────►│   STORAGE   │◄────────────────►│ STORAGE│ │
│  │  (CT/MRI)   │   (Port 533)   │   SERVER    │                  │   DB   │ │
│  │   [SCU]     │                │    [SCP]    │                  │        │ │
│  └─────────────┘                └──────┬──────┘                  └────────┘ │
│                                        │                                    │
│                                        ▼                                    │
│                              ┌─────────────────┐                            │
│                              │  FILE STORAGE   │                            │
│                              │  (DICOM Images) │                            │
│                              │  D:\PACS_Images\│                            │
│                              └─────────────────┘                            │
│                                                                             │
│  ┌─────────────┐    C-FIND      ┌─────────────┐    C-MOVE/C-GET   ┌────────┐ │
│  │ WORKSTATION │◄──────────────►│ WORKSTATION │◄────────────────►│ STORAGE│ │
│  │  (Viewer)   │   (Port 334)   │   SERVER    │   (Retrieve)      │ SERVER │ │
│  │   [SCU]     │                │    [SCP]    │                   │        │
│  └─────────────┘                └─────────────┘                   └────────┘ │
│                                                                             │
│  ┌─────────────┐    C-FIND      ┌─────────────┐    SQL Queries   ┌────────┐ │
│  │  MODALITY   │◄──────────────►│  WORKLIST   │◄────────────────►│WORKLIST│ │
│  │  (CT/MRI)   │   (Port 633)   │   SERVER    │                  │   DB   │ │
│  │   [SCU]     │   (MWL Query)  │    [SCP]    │                  │        │ │
│  └─────────────┘                └─────────────┘                  └────────┘ │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                         CLIENT CONFIGURATION                                │
│                      (LEADTOOLS as DICOM SCU)                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      MAIN CLIENT (SCU)                              │   │
│  │  AE Title: L23_CLIENT64      │    Port: 1033                       │   │
│  │                                                                     │   │
│  │  Role:      Initiates C-STORE, C-FIND, C-MOVE to remote servers    │   │
│  │  Use Case:  Sending studies to external PACS                        │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                   WORKSTATION CLIENT (SCU)                          │   │
│  │  AE Title: L23_WS_CLIENT64   │    Port: 1034                       │   │
│  │                                                                     │   │
│  │  Role:      Viewer application acting as client                     │   │
│  │  Use Case:  Querying and retrieving from remote PACS                │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Database Layer

### Purpose
Databases store **metadata** and **configuration** while actual DICOM images are stored in the file system. This separation optimizes performance and enables fast searching.

### Database Types

| Database | Purpose | Key Tables | Connection String Component |
|----------|---------|------------|----------------------------|
| **Storage Database** | Central repository for all imaging studies | Patients, Studies, Series, Instances, StorageLocations | `Database=PACSStorageDB` |
| **Workstation Database** | Viewer-specific settings and user preferences | Users, Layouts, Annotations, ToolSettings, QueryHistory | `Database=WorkstationDB` |
| **Worklist Database** | Scheduled procedures for modalities (MWL) | WorklistItems, ScheduledProcedures, ModalityWorklist | `Database=WorklistDB` |
| **Demo Database** | Sample data for testing | DemoPatients, DemoStudies | `Database=DemoDB` |

### Database Schema Relationships

```
┌─────────────┐       ┌─────────────┐       ┌─────────────┐       ┌─────────────┐
│   Patient   │◄─────►│    Study    │◄─────►│    Series   │◄─────►│   Instance  │
│             │  1:M  │             │  1:M  │             │  1:M  │  (Image)    │
│ • PatientID │       │ • StudyUID  │       │ • SeriesUID │       │ • SOP UID   │
│ • Name      │       │ • Accession │       │ • Modality  │       │ • FilePath  │
│ • DOB       │       │ • Date      │       │ • Number    │       │ • Size      │
│ • Sex       │       │ • Time      │       │ • Description│      │ • Checksum  │
└─────────────┘       └─────────────┘       └─────────────┘       └─────────────┘
                                                                           │
                                                                           ▼
                                                                  ┌─────────────┐
                                                                  │ File System │
                                                                  │ (DICOM Files)│
                                                                  └─────────────┘
```

---

## Server/Devices Layer

### DICOM Server Roles (SCP - Service Class Provider)

| Server | AE Title | Port | Primary Function | Receives From | Stores To |
|--------|----------|------|------------------|---------------|-----------|
| **Storage SCP** | `L23_PACS_SCP64` | 533 | Image ingestion | Modalities (CT, MRI, US, XA) | Storage DB + Files |
| **Workstation SCP** | `L23_WS_SERVER64` | 334 | Query/Retrieve | Radiologist workstations | Workstation DB |
| **Worklist SCP** | `L23_MWL_SCP64` | 633 | Modality worklist | Modalities (before scan) | Worklist DB |
| **Demo SCP** | `L23_SERVER64` | 333 | Testing/Development | Test applications | Demo DB |

### Client Roles (SCU - Service Class User)

| Client | AE Title | Port | Primary Function | Connects To |
|--------|----------|------|------------------|-------------|
| **Main Client** | `L23_CLIENT64` | 1033 | Send/Query external PACS | Remote Storage SCP |
| **Workstation Client** | `L23_WS_CLIENT64` | 1034 | Viewer queries | Remote Workstation SCP |

### External Device Integration

| Device Type | Role | Typical AE Title | Connects To Our | Protocol |
|-------------|------|------------------|-----------------|----------|
| **CT Scanner** | SCU | `CT_ROOM_01`, `SIEMENS_CT1` | Storage SCP (533), Worklist SCP (633) | C-STORE, C-FIND (MWL) |
| **MRI Scanner** | SCU | `MRI_MAIN`, `PHILIPS_MR1` | Storage SCP (533), Worklist SCP (633) | C-STORE, C-FIND (MWL) |
| **X-Ray (CR/DR)** | SCU | `XR_PORTABLE`, `CR_ROOM2` | Storage SCP (533) | C-STORE |
| **Ultrasound** | SCU | `US_CARDIAC`, `US_GENERAL` | Storage SCP (533), Worklist SCP (633) | C-STORE, C-FIND (MWL) |
| **PACS Workstation** | SCU/SCP | `WS_RAD_01`, `VIEWER_MAIN` | Workstation SCP (334) | C-FIND, C-MOVE, C-GET |
| **RIS/HIS** | SCU | `RIS_MAIN`, `HIS_HOSPITAL` | Worklist SCP (633) | C-FIND (MWL updates) |
| **External PACS** | SCP | `PACS_HOSPITAL2`, `VNA_ENTERPRISE` | Main Client (1033) | C-STORE, C-FIND, C-MOVE |

---

## Network Layer (DICOM Protocol)

### DIMSE Services Overview

DICOM uses **DIMSE** (DICOM Message Service Element) for network communication. All operations occur over **TCP/IP** on specific ports.

### Association Establishment

Before any DICOM operation, devices must establish an **Association**:

```
┌─────────┐                          ┌─────────┐
│   SCU   │ ───── A-ASSOCIATE-RQ ───►│   SCP   │
│ (Client)│◄───── A-ASSOCIATE-AC ───│ (Server)│
│         │◄───── A-ASSOCIATE-RJ ───│         │ (if rejected)
└─────────┘                          └─────────┘

Negotiated: Application Context, Presentation Contexts, 
            Transfer Syntaxes, Maximum PDU Size
```

---

## DICOM Command Reference

### C-ECHO (Verification)

| Attribute | Value |
|-----------|-------|
| **Purpose** | Verify connectivity between DICOM devices |
| **Command** | `C-ECHO-RQ` / `C-ECHO-RSP` |
| **Direction** | SCU → SCP |
| **Use Case** | "Ping" test before sending images |
| **Response** | Success (0000) or Failure (XXXX) |

**Example Flow:**
```
SCU: C-ECHO-RQ ──────► SCP
SCU: ◄─────────────── C-ECHO-RSP (Status: 0000H - Success)
```

---

### C-STORE (Storage)

| Attribute | Value |
|-----------|-------|
| **Purpose** | Transfer DICOM instances (images, reports) |
| **Command** | `C-STORE-RQ` / `C-STORE-RSP` |
| **Direction** | SCU → SCP (Push model) |
| **SOP Classes** | CT Image Storage, MR Image Storage, Secondary Capture, etc. |
| **Key Tags** | SOP Class UID, SOP Instance UID, Move Originator |

**Example Flow:**
```
SCU: C-STORE-RQ (CT Image Dataset) ──────► SCP
SCU: ◄────────────────────────────────── C-STORE-RSP (Status: 0000H)
```

**Status Codes:**
| Code | Meaning |
|------|---------|
| `0000` | Success |
| `0110` | Processing failure |
| `0122` | SOP Class not supported |
| `0213` | Resource limitation |

---

### C-FIND (Query)

| Attribute | Value |
|-----------|-------|
| **Purpose** | Search for studies, series, or instances |
| **Command** | `C-FIND-RQ` / `C-FIND-RSP` |
| **Direction** | SCU → SCP |
| **Query Levels** | Patient, Study, Series, Instance (Image) |
| **Matching** | Exact, Wildcard, Universal |

**Query/Retrieve Information Model:**

| Level | Key Attributes | Example Query |
|-------|---------------|---------------|
| **Patient** | PatientID, PatientName, PatientBirthDate | `PatientName=*DOE*` |
| **Study** | StudyInstanceUID, StudyDate, AccessionNumber | `StudyDate=20240101-20240131` |
| **Series** | SeriesInstanceUID, Modality, SeriesNumber | `Modality=CT` |
| **Instance** | SOPInstanceUID, InstanceNumber | `InstanceNumber=1` |

**Example Flow:**
```
SCU: C-FIND-RQ (Query: PatientName=*DOE*, StudyDate=TODAY) ──────► SCP
SCU: ◄────────────────────────────────────────────────────────── C-FIND-RSP (Match #1)
SCU: ◄────────────────────────────────────────────────────────── C-FIND-RSP (Match #2)
SCU: ◄────────────────────────────────────────────────────────── C-FIND-RSP (Match #3)
SCU: ◄────────────────────────────────────────────────────────── C-FIND-RSP (Status: 0000H, Done)
```

---

### C-MOVE (Retrieve - Push Model)

| Attribute | Value |
|-----------|-------|
| **Purpose** | Retrieve images via third-party transfer |
| **Command** | `C-MOVE-RQ` / `C-MOVE-RSP` |
| **Direction** | SCU → SCP (SCP pushes to third device) |
| **Complexity** | High (3 devices involved) |
| **Firewall** | Requires SCP → Target connection |

**Architecture:**
```
┌─────────┐     C-MOVE-RQ      ┌─────────┐     C-STORE      ┌─────────┐
│  SCU    │ ─────────────────► │   SCP   │ ────────────────►│ Target  │
│(Request)│                    │ (Source)│   (Images)       │(Retrieve)│
│         │◄────────────────── │         │◄──────────────── │  (SCP)  │
│         │    C-MOVE-RSP      │         │                  │         │
└─────────┘   (Status: Pending)└─────────┘                  └─────────┘
```

**Important:** C-MOVE requires the **SCP to initiate a new C-STORE connection** to the target AE. This often causes firewall issues.

---

### C-GET (Retrieve - Pull Model)

| Attribute | Value |
|-----------|-------|
| **Purpose** | Direct image download |
| **Command** | `C-GET-RQ` / `C-GET-RSP` |
| **Direction** | SCU → SCP (Same connection) |
| **Complexity** | Low (2 devices only) |
| **Firewall** | Simple (single connection) |

**Architecture:**
```
┌─────────┐     C-GET-RQ       ┌─────────┐
│  SCU    │ ─────────────────► │   SCP   │
│(Request)│                    │ (Source)│
│         │◄────────────────── │         │
│         │    C-GET-RSP       │         │
│         │    + Images        │         │
└─────────┘                    └─────────┘
```

**Comparison: C-MOVE vs C-GET**

| Feature | C-MOVE | C-GET |
|---------|--------|-------|
| Connection | Separate C-STORE association | Same association |
| Devices Involved | 3 (Requester, Source, Target) | 2 (Requester, Source) |
| Firewall Friendly | No (inbound to target needed) | Yes |
| Implementation Complexity | High | Low |
| Hospital Adoption | Very Common | Rare |
| Use Case | Standard PACS retrieve | Direct download, mobile |

---

## Complete Test Flow

### End-to-End Integration Test

This test validates the entire PACS workflow from modality simulation to image retrieval.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         END-TO-END TEST FLOW                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  PHASE 1: MODALITY WORKLIST (MWL)                                          │
│  ┌─────────┐         C-FIND (MWL)          ┌─────────┐    SQL     ┌────────┐│
│  │  Test   │ ─────────────────────────────►│Worklist │◄──────────►│Worklist││
│  │ Modality│  "Get scheduled CT for today" │  SCP    │            │   DB   ││
│  │  (SCU)  │◄──────────────────────────────│ (633)   │            │        ││
│  └─────────┘      Response: 3 patients      └─────────┘            └────────┘│
│                                                                             │
│  PHASE 2: IMAGE ACQUISITION & STORAGE                                       │
│  ┌─────────┐         C-STORE               ┌─────────┐    SQL     ┌────────┐│
│  │  Test   │ ─────────────────────────────►│ Storage │◄──────────►│Storage ││
│  │ Modality│  "Send CT study for Patient A"│  SCP    │            │   DB   ││
│  │  (SCU)  │◄──────────────────────────────│ (533)   │            │        ││
│  └─────────┘      Response: Success (0000)  └────┬────┘            └────────┘│
│                                                  │                          │
│                                                  ▼                          │
│                                           ┌─────────────┐                   │
│                                           │ File System │                   │
│                                           │ D:\PACS\... │                   │
│                                           └─────────────┘                   │
│                                                                             │
│  PHASE 3: QUERY & RETRIEVAL                                                 │
│  ┌─────────┐         C-FIND                ┌─────────┐    SQL     ┌────────┐│
│  │Radiologist│◄───────────────────────────►│Workstation│◄────────►│Storage ││
│  │Workstation│ "Find Patient A's CT today" │   SCP     │          │   DB   ││
│  │   (SCU)   │                             │  (334)    │          │        ││
│  └─────┬─────┘                             └───────────┘          └────────┘│
│        │                                                                    │
│        │         C-MOVE / C-GET                                            │
│        │ "Retrieve images to my workstation"                                │
│        └────────────────────────────────────────────────────────────────►    │
│                                                                             │
│  PHASE 4: VERIFICATION                                                      │
│  ┌─────────┐         C-ECHO                ┌─────────┐                      │
│  │  Admin  │ ─────────────────────────────►│  Demo   │                      │
│  │ Console │◄──────────────────────────────│  SCP    │                      │
│  │         │      "Verify all services up" │ (333)   │                      │
│  └─────────┘                               └─────────┘                      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Step-by-Step Test Execution

#### Step 1: Verify Services are Running

```powershell
# Check SQL Server
Get-Service MSSQL$SQLEXPRESS

# Check LEADTOOLS Services (if installed as Windows Services)
Get-Service | Where-Object {$_.Name -like "*LEADTOOLS*" -or $_.Name -like "*PACS*"}

# Check ports are listening
netstat -an | findstr "533 334 333 633 8033"
```

#### Step 2: Test C-ECHO (Connectivity)

Using `storescu` from DCMTK or LEADTOOLS utilities:

```bash
# Test Storage Server
echoscu -v DESKTOP-UAB1GD8 533 -aec L23_PACS_SCP64

# Expected output:
# I: Association Accepted (Max PDU: 16384)
# I: Received Echo Response (Success)
# I: Association Released
```

#### Step 3: Test C-STORE (Send Test Image)

```bash
# Send a sample DICOM file to Storage Server
storescu -v DESKTOP-UAB1GD8 533 -aec L23_PACS_SCP64 -aet TEST_MODALITY sample.dcm

# Expected output:
# I: Association Accepted
# I: Sending C-STORE Request
# I: Received C-STORE Response (Success)
# I: Association Released
```

#### Step 4: Test C-FIND (Query)

```bash
# Query for all studies today
findscu -v DESKTOP-UAB1GD8 334 -aec L23_WS_SERVER64 -aet TEST_WORKSTATION -S -k StudyDate=20240312

# Expected: List of matching studies
```

#### Step 5: Test C-MOVE (Retrieve)

```bash
# Retrieve study to local AE
movescu -v DESKTOP-UAB1GD8 334 -aec L23_WS_SERVER64 -aet TEST_WORKSTATION -S -k StudyInstanceUID=1.2.3.4.5 --move TEST_WORKSTATION

# Expected: Images transferred via separate C-STORE
```

#### Step 6: Verify Database Entries

```sql
-- Check Storage Database
USE PACSStorageDB;
SELECT TOP 10 PatientID, PatientName, StudyDate, ModalitiesInStudy
FROM Studies
ORDER BY StudyDate DESC;

-- Check if images are linked
SELECT s.PatientID, i.SOPInstanceUID, i.FilePath
FROM Instances i
JOIN Series sr ON i.SeriesUID = sr.SeriesUID
JOIN Studies s ON sr.StudyUID = s.StudyUID
WHERE s.StudyDate >= CAST(GETDATE() AS DATE);
```

---

## PACS Integration

### Integration with Hospital Systems

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         HOSPITAL IT LANDSCAPE                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐      HL7/FHIR      ┌─────────────┐      DICOM      ┌─────┐ │
│  │     HIS     │◄──────────────────►│     RIS     │◄───────────────►│ PACS│ │
│  │  (Hospital  │                    │ (Radiology  │                 │(Ours)│ │
│  │   Info Sys) │                    │  Info Sys)  │                 │      │ │
│  └─────────────┘                    └──────┬──────┘                 └──┬───┘ │
│                                            │                           │     │
│                                            │    Orders/Schedules       │     │
│                                            │    (HL7 ORM, O01)         │     │
│                                            ▼                           ▼     │
│                                     ┌─────────────┐              ┌─────────┐  │
│                                     │  Worklist   │              │ Storage │  │
│                                     │   Server    │              │ Server  │  │
│                                     │ (Modality   │              │ (Images)│  │
│                                     │  Worklist)  │              │         │  │
│                                     └──────┬──────┘              └────┬────┘  │
│                                            │                          │      │
│                                            │    DICOM MWL             │      │
│                                            │    (C-FIND)              │      │
│                                            ▼                          ▼      │
│                                     ┌─────────────┐              ┌─────────┐  │
│                                     │  Modality   │              │Radiologist│ │
│                                     │ (CT/MRI/US) │              │Workstation│ │
│                                     │             │              │ (Viewer) │ │
│                                     └─────────────┘              └─────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### External PACS Communication

When integrating with external PACS systems:

| Scenario | Our Role | Protocol | Configuration |
|----------|----------|----------|---------------|
| **Send to External PACS** | SCU (Client) | C-STORE | Configure Main Client (1033) with remote AE details |
| **Query External PACS** | SCU (Client) | C-FIND | Use Main Client to query remote Workstation SCP |
| **Receive from External PACS** | SCP (Server) | C-MOVE | External PACS initiates C-MOVE to our Storage SCP (533) |
| **Peer-to-Peer** | Both | All | Bidirectional association configuration |

---

## Configuration Reference

### SQL Server Connection Strings

```ini
; Windows Authentication (Current User)
Server=DESKTOP-UAB1GD8\SQLEXPRESS;Database=PACSStorageDB;Integrated Security=true;

; SQL Server Authentication (sa account)
Server=DESKTOP-UAB1GD8\SQLEXPRESS;Database=PACSStorageDB;User Id=sa;Password=YourStrongPassword;

; With connection pooling
Server=DESKTOP-UAB1GD8\SQLEXPRESS;Database=PACSStorageDB;Integrated Security=true;Pooling=true;Min Pool Size=5;Max Pool Size=100;
```

### DICOM Association Parameters

| Parameter | Typical Value | Description |
|-----------|---------------|-------------|
| **Calling AE Title** | `L23_CLIENT64` | Our application's AE title |
| **Called AE Title** | `L23_PACS_SCP64` | Remote server's AE title |
| **PDU Max Length** | 16384-65536 bytes | Maximum packet size |
| **Timeout** | 30-60 seconds | Association/operation timeout |
| **Transfer Syntax** | Implicit VR Little Endian, Explicit VR Little Endian, JPEG Lossless | Image encoding formats |

### Port Configuration Summary

| Service | Port | Protocol | Direction | Firewall Rule |
|---------|------|----------|-----------|---------------|
| Storage SCP | 533 | TCP | Inbound | Allow from modality subnet |
| Workstation SCP | 334 | TCP | Inbound | Allow from workstation subnet |
| Demo SCP | 333 | TCP | Inbound | Allow localhost only (dev) |
| Worklist SCP | 633 | TCP | Inbound | Allow from modality subnet |
| Broker Host | 8033 | TCP | Internal | Localhost only |
| Main Client | 1033 | TCP | Outbound | Allow to external PACS |
| Workstation Client | 1034 | TCP | Outbound | Allow to external PACS |
| SQL Server | 1433 | TCP | Internal | Localhost only (Express) |

---

## Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| **Association Rejected** | AE Title mismatch | Verify Called AE matches server configuration |
| **C-MOVE Fails** | Firewall blocking return connection | Open ports for C-STORE or use C-GET |
| **C-STORE Fails (0110)** | Insufficient disk space | Check storage drive capacity |
| **Query Returns Empty** | Database connection issue | Verify SQL Server service is running |
| **Worklist Empty** | No scheduled procedures | Check RIS/HL7 interface or manual entry |
| **Images Not Displaying** | Transfer syntax not supported | Install appropriate LEADTOOLS codecs |

### Diagnostic Commands

```powershell
# Test DICOM connectivity
Test-NetConnection -ComputerName DESKTOP-UAB1GD8 -Port 533

# View DICOM association logs (if logging enabled)
Get-Content "C:\ProgramData\LEADTOOLS\Logs\dicom.log" -Tail 50

# Check SQL Server connectivity
sqlcmd -S DESKTOP-UAB1GD8\SQLEXPRESS -E -Q "SELECT @@VERSION"

# Verify DICOM file integrity
dcmdump.exe sample.dcm | findstr "TransferSyntax SOPClass"
```

---

## Resources

- [LEADTOOLS PACS Framework Documentation](https://www.leadtools.com/help/sdk/tutorials/dotnet-winforms-handle-move-requests-in-a-pacs-server.html)
- [DICOM Standard - Part 7: Message Exchange](https://www.dicomstandard.org/standards/view/message-exchange)
- [Orthanc DICOM Guide](https://orthanc.uclouvain.be/book/dicom-guide.html)
- [DCMTK Tools Documentation](https://dicom.offis.de/dcmtk.php.en)

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2024-03-12 | Initial documentation for DESKTOP-UAB1GD8 deployment |

---

## License

This documentation is for internal use with LEADTOOLS PACS Framework licensed deployment.
```

---

This README format documentation covers everything you requested:

✅ **Database Layer** - Detailed purposes and relationships  
✅ **Server/Devices Layer** - Complete AE Title and port configuration  
✅ **Network Layer** - DIMSE protocol with all 5 major commands (C-ECHO, C-STORE, C-FIND, C-MOVE, C-GET)  
✅ **Complete Test Flow** - End-to-end workflow from modality to workstation  
✅ **PACS Integration** - How your system fits into the broader hospital IT landscape  

You can save this as `README.md` in your project repository. Would you like me to add any specific sections, such as **security hardening**, **backup procedures**, or **HL7 integration details**?
