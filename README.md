
# Dualliste C2: Tactical Security Orchestration Core

<img style="padding-top: 0;" width="130" align="right" src="shared/Dualliste.png"  alt="Dualliste C2 Logo" >

Dualliste C2 is a high-performance, unified command-and-control (C2) workspace built for cybersecurity profiling, vulnerability auditing, and network topology discovery. Rather than juggling dozens of terminal windows, Dualliste coordinates security scanners in the background, parses raw outputs, maps assets on an interactive graph, and generates actionable, heuristic-based recomendations.

An integrated AI Consultation panel acts as a real-time operations copilot, providing tactical advice and even executing approved command sequences (HITL YOLO mode).

---

## Architecture & Tech Stack

The platform is designed with a lightweight, decentralized layout to ensure speed and zero heavy dependencies:

*   **Backend**: Written in **Go (Golang)** using the **Gin** HTTP framework. It handles process execution, terminal pseudo-ttys (PTY), and real-time state synchronization via WebSockets.
*   **Database**: File-based session database. All discoveries, timeline events, scan history, and active sessions are serialized as structured JSON archives under `shared/sessions/` for portability and easy backups.
*   **Frontend**: Built with **Next.js** (App Router), **Zustand** for global tactical state management, and **Tailwind CSS** for a flat, cyberpunk-inspired operational HUD.
*   **Active Graph**: Rendered using **React Flow (@xyflow/react)** for dynamic asset and service topology mapping.

---

## Core Features

### 1. Multi-Tool Orchestraton Engine
The backend engine loads tool definitions dynamically from YAML configurations in `shared/tools/`. It executes security binaries (like `nmap`, `gobuster`, `feroxbuster`, `httpx`, etc.) locally and streams outputs over WebSockets.
Each tool profile is mapped to raw stdout regex Parsers. The parsed tokens are automatically registered as **Discoveries** (e.g., ports, services, subdomains, vulnerabilities).

### 2. Heuristic Rule System
When new discoveries are logged, the backend evaluates them against the rules defined in `shared/heuristics/core_rules.yaml`. For instance:
*   A newly identified domain triggers a recommendation to map web services via `httpx`.
*   An open port `80` or `443` suggests technolgy fingerprinting via `whatweb` and directory fuzzing via `feroxbuster`.
These recommendations appear directly in the CTF Playbook / Operation Centre views.

### 3. AI Consultation & HITL YOLO Mode
The left panel houses a live assistant loaded with custom system prompts and API presets (defined in `shared/config/ai_presets.json`). 
*   **Interactive Consultation**: Discuss current scan findings and seek verification commands.
*   **HITL (Human-in-the-loop) Approvals**: When the AI proposes an exploit or verification command, the frontend prompts the user with a confirmation modal. The command is only executed on the host OS once approved.

### 4. Interactive Topology Graph
Discovered nodes are rendered in real-time. Target IPs, hostnames, open ports, and technologies are visualy linked to build a clean representation of the network attack surface.

---

## Setup & Instalation

### Prerequisites
Make sure your host platform is running Linux and has the desired security tools installed on the system PATH (e.g. `nmap`, `gobuster`, `whatweb`, etc.).

### 1. Launch the Backend
Navigate to the backend directory, install Go dependencies, and run:

```bash
cd backend
go mod tidy
go run cmd/main.go
```

The backend server starts on port `:33` by default and listens for WebSocket clients on `/ws`.

### 2. Launch the Frontend
Navigate to the frontend directory, install npm packages, and run the Next.js development server:

```bash
cd frontend
pnpm install
pnpm dev
```

Open your browser and navigate to `http://localhost:3000`. The interface will automatically prompt you to initialize a new session or load an existing operation.

---

## Configuration & Customization

### Adding a Custom Tool
Create a new YAML file under `shared/tools/` using the following schema:

```yaml
name: "whatweb"
category: "recon"
description: "Web technology identifier"
default_binary_name: "whatweb"
is_installed: true
profiles:
  - id: 1
    name: "Standard Fingerprint"
    args: ["--color=never", "-v", "{{TARGET}}"]
    rationale: "Fingerprint web servers to detect technology stack and potential CMS versions."
```

### Modifying Heuristics
Add or edit matching rules in `shared/heuristics/core_rules.yaml` to specify what recomendations are triggered by different types of discoveries.

---

## System Diagnostics & Telemetry

When connected, the frontend receives periodic system load updates over WebSockets. The system landing page displays live CPU, RAM, and Disk utilization of the host machine, along with active scan jobs currently running.
