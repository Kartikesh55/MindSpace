# MindSpace

> **Think out loud. Watch it evolve.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square)](LICENSE)
[![React](https://img.shields.io/badge/React-19-cyan.svg?style=flat-square)](https://react.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-6-blue.svg?style=flat-square)](https://www.typescriptlang.org/)
[![Vite](https://img.shields.io/badge/Vite-8-purple.svg?style=flat-square)](https://vitejs.dev/)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen?style=flat-square)](package.json)
[![Lint Status](https://img.shields.io/badge/lint-0%20errors-brightgreen?style=flat-square)](package.json)
[![Tests](https://img.shields.io/badge/tests-passing-brightgreen?style=flat-square)](package.json)

An AI-powered, voice-first system architect that transforms natural language requirements into evolving software architecture graphs in real time.

---

## 1. Overview & Problem Statement

Traditional system architecture diagrams are created through static drag-and-drop tools or one-shot text-to-diagram generators. Both suffer from fundamental flaws:

1. **No Incremental Evolution:** Real architectural thinking is conversational and exploratory. As requirements evolve, updating static diagrams requires manual rewiring or regenerating everything from scratch.
2. **Slow, Opaque Feedback:** Conventional AI code or diagram generators block the user while computing, giving no intermediate feedback.
3. **Rigid Execution without Interruption:** If a user clarifies or pivots their requirement mid-thought (e.g., *"Wait, make it multi-agent"* or *"Actually, use RabbitMQ instead of Kafka"*), standard tools either ignore the correction or produce conflicting, duplicate nodes.

**MindSpace** solves this with a **full-duplex, interruptible conversational pipeline**:
- Speak or type natural language requirements and watch the architecture assemble node-by-node in real time.
- Decouples immediate conversational feedback (**<60ms response**) from asynchronous layout calculation and operation execution.
- Interrupt at any point mid-sentence or mid-build. MindSpace immediately acknowledges the pivot, safely aborts in-flight work via cancellation tokens, marks stale entities as superseded, and replans dynamically without corrupting valid upstream state.

---

## 2. Key Features

- **Voice-First & Multimodal Ingress:** Speak requirements naturally via microphone or enter them via keyboard. Support for production WebRTC via LiveKit, zero-config in-browser speech via the Web Speech API, or deterministic mock modes for offline testing.
- **Conversational Decoupling (<60ms Ack):** Provides instant conversational acknowledgement (e.g., *"Got it — synthesizing your architecture."*) before executing computationally intensive layout and scheduling tasks.
- **Dynamic Intent Classification:** Automatically classifies user prompts into `NEW_REQUIREMENT`, `MODIFICATION`, `CORRECTION`, and `CANCELLATION`.
- **Topological Revision Engine:** Performs differential graph analysis to determine exactly which nodes to preserve, supersede, or rewire, avoiding destructive full-canvas re-renders.
- **Interruptible Operation Engine:** Employs `AbortController` cancellation tokens. When a user interrupts mid-flight, running tasks transition cleanly (`RUNNING → INTERRUPTED → SUPERSEDED`) without race conditions or orphan nodes.
- **7 Pre-Commit Safety Gates:** Every graph mutation must pass 7 invariant checks (existence, status, signal, intent currency, idempotency, supersession, and base version) before committing to state.
- **Visual Interruption HUD:** A dynamic on-canvas banner visibly reflects operational phases: `USER_INTERRUPTED`, `STALE_PLAN`, `REPLANNING`, `NEW_INTENT`, and `STATE_COMMITTED`.
- **Interactive Graph Canvas:** Powered by React Flow (`@xyflow/react`) and Dagre auto-layout, rendering categorized component cards with health metrics, technology badges, and live connection lines.
- **Immutable Snapshot History:** Every committed graph evolution creates an immutable snapshot (`v1`, `v2`, `v3...`), allowing one-click rollback to prior architectures.
- **Full-Duplex-Bench v3 (FDB-v3) Adapter:** Implements a standardized benchmark integration layer with dynamic tool execution, in-flight argument updating, and structured JSONL telemetry logging with automated secrets redaction.

---

## 3. Demo & Walkthrough

![MindSpace Architecture Interface](src/assets/hero.png)

### Interactive Scenarios

MindSpace includes multiple built-in interactive demonstration flows accessible directly from the interface:

#### Scenario A: Automated 3-Stage Evolution Demo
Click **`▶ Run 60s Demo`** in the top navigation to trigger the automated 3-stage demonstration:
1. **Stage 1 (Initial Requirement):** *"I'm building an AI customer support system."*  
   → Synthesizes `User → Support Agent → RAG → Knowledge Base`.
2. **Stage 2 (Mid-Flight Interruption):** User interrupts with *"Actually, make it a multi-agent system."*  
   → Halts the single support agent, marks it `SUPERSEDED`, and seamlessly deploys `Agent Router → Sales Agent, Billing Agent, Technical Agent`.
3. **Stage 3 (Targeted Evolution):** *"Billing should have its own database."*  
   → Provisions a dedicated `Billing Database` (ACID Ledger) and links it directly to `Billing Agent`.

#### Scenario B: Social Media Platform & Mid-Flight Broker Swap
1. **Initial Requirement:**  
   > *"Design a real-time social media platform with a mobile app, API gateway, authentication service, user service, post service, media storage, Redis cache, notification service, Kafka, and PostgreSQL."*  
   → Deploys a 10-node, 9-edge architecture across 6 distinct tiers.
2. **Self-Correction Interruption:**  
   > *"Actually, replace Kafka with RabbitMQ and add a recommendation service that uses the post service data."*  
   → Aborts in-flight Kafka operations, supersedes Kafka, introduces RabbitMQ, and connects Recommendation Service while keeping Mobile App, Auth, User, and Storage intact.
3. **Dependency Correction:**  
   > *"The recommendation service should use MongoDB instead of PostgreSQL."*  
   → Adds a dedicated MongoDB instance and rewires only the Recommendation Service dependency.

---

## 4. Technology Stack

MindSpace is built using modern TypeScript and web standards:

| Category | Technology | Purpose |
|---|---|---|
| **Frontend Framework** | React 19 (`react`, `react-dom`) | Modern component architecture and concurrent rendering |
| **Language & Tooling** | TypeScript 6, Vite 8, `tsx` | Strict type safety, sub-second HMR, and TypeScript test execution |
| **Graph Visualization** | React Flow (`@xyflow/react` v12) | Node canvas, customizable handles, interactive pan/zoom |
| **Graph Layout Engine** | Dagre (`@dagrejs/dagre`) | Deterministic left-to-right hierarchical topological layout |
| **State Management** | Zustand 5 (`zustand`) | Centralized, reactive state store with immutable snapshots |
| **Styling & UI** | Tailwind CSS 3, `clsx`, `tailwind-merge` | Responsive dark-mode interface and custom UI themes |
| **Iconography** | Lucide React (`lucide-react`) | Consistent iconography across nodes, sidebar, and telemetry |
| **Voice Transport** | Web Speech API & LiveKit Agents | Local browser speech capture + optional WebRTC LiveKit server |
| **Linter** | Oxlint (`oxlint`) | Ultra-fast Rust-based static code analysis |

---

## 5. System Architecture & Component Pipeline

The following diagram illustrates how MindSpace handles real-time user voice/text input, guarantees sub-60ms acknowledgement, prevents stale mutations, and updates the canvas:

```
[ User Microphone / Keyboard ]
              │
              ▼
   ┌───────────────────────┐
   │ Voice / Text Ingress  │ (LiveKit WebRTC | Browser Web Speech | Keyboard)
   └──────────┬────────────┘
              │
              ▼
   ┌────────────────────────────────────────────────────────┐
   │ EventBridge Pipeline Coordinator                       │
   │  ├── Immediate Conversational Ack (<60ms)              │
   │  └── Hardware Latency Telemetry (performance.now())    │
   └──────────┬─────────────────────────────────────────────┘
              │
              ▼
   ┌────────────────────────────────────────────────────────┐
   │ Intent Manager & Change Detector                       │
   │  └── Classifies: NEW_REQ | MODIFICATION | CORRECTION   │
   └──────────┬─────────────────────────────────────────────┘
              │
              ▼
   ┌────────────────────────────────────────────────────────┐
   │ Revision Engine & Architecture Interpreter             │
   │  ├── Calculates minimal topological difference         │
   │  └── Supersedes stale components, preserves valid ones │
   └──────────┬─────────────────────────────────────────────┘
              │
              ▼
   ┌────────────────────────────────────────────────────────┐
   │ Interruptible Operation Engine                         │
   │  ├── Operation Scheduler (Deterministic Idempotency)   │
   │  ├── AbortController Signal Dispatch                   │
   │  └── 7 Pre-Commit Safety Gates:                        │
   │       1. Existence Guard        5. Idempotency Guard   │
   │       2. Status Guard (RUNNING) 6. Supersession Guard  │
   │       3. Signal Guard (aborted) 7. Version Currency    │
   │       4. Intent Currency Guard                         │
   └──────────┬─────────────────────────────────────────────┘
              │
              ▼
   ┌────────────────────────────────────────────────────────┐
   │ Reactive Store & UI Presentation                       │
   │  ├── Zustand Store (Versioned Snapshots v1, v2, v3...)  │
   │  ├── Dagre Graph Layout (Non-negative finite coords)   │
   │  ├── React Flow Canvas (Custom ArchitectureNode cards) │
   │  └── Interruption HUD (Real-time lifecycle badges)     │
   └────────────────────────────────────────────────────────┘
```

---

## 6. Project Directory Structure

```
mindspace/
├── agent/                         # Optional Python LiveKit voice agent
│   ├── agent.py                   # LiveKit Agent worker with Silero VAD
│   ├── README.md                  # Setup guide for Python agent worker
│   └── requirements.txt           # Python dependencies for LiveKit agent
├── docs/                          # Technical deep-dive documentation
│   ├── architecture.md            # System architecture and layer breakdown
│   ├── benchmark-integration.md   # Full-Duplex-Bench v3 specification
│   ├── demo-script.md             # Interactive demo scripts and scenarios
│   ├── interruption-model.md      # State machine and 7 pre-commit gates
│   └── operation-lifecycle.md     # Atomic operations and idempotency keys
├── public/                        # Static assets (favicons, SVG icon sets)
├── scripts/                       # Benchmark runners (Python, Bash, PowerShell)
│   ├── run-fdb-v3.py
│   ├── run-fdb-v3.sh
│   └── run-fdb-v3.ps1
├── src/                           # Application source code
│   ├── assets/                    # Application images and branding
│   ├── components/                # React UI components
│   │   ├── ActivityPanel/         # Real-time event and operation timeline
│   │   ├── GraphCanvas/           # React Flow canvas and empty states
│   │   ├── GraphNode/             # Custom ArchitectureNode cards & themes
│   │   ├── Header/                # Navigation, version badge, snapshot history
│   │   ├── IntentPanel/           # Active intent display and telemetry
│   │   ├── InterruptionHUD/       # Color-coded interruption state banner
│   │   ├── Sidebar/               # Component hierarchy and dependency tree
│   │   ├── StatusBar/             # System health and latency telemetry
│   │   └── VoiceBar/              # Floating mic controller & input bar
│   ├── demo/                      # Automated demo controller & scenario catalog
│   ├── engine/                    # Core compilation, planning, & revision engine
│   │   ├── architectureInterpreter.ts  # NLP tier classification
│   │   ├── architectureReplanner.ts    # Graph rewiring and diffing
│   │   ├── deterministicInterpreter.ts # Grammar and keyword parser
│   │   ├── graphEngine.ts              # Dagre layout coordinate calculator
│   │   ├── intentManager.ts            # Intent classification
│   │   ├── operationEngine.ts          # AbortController & 7 pre-commit gates
│   │   ├── planner.ts                  # High-level architecture planner
│   │   └── revisionEngine.ts           # Evolution & invalidation engine
│   ├── fdb/                       # Full-Duplex-Bench v3 adapter subsystem
│   │   ├── FDBAdapter.ts               # Scenario adapter and tool dispatcher
│   │   ├── InterruptibleToolExecutor.ts# Benchmark tool execution engine
│   │   ├── ToolRegistry.ts             # Generic tool definition catalog
│   │   └── logger.ts                   # JSONL logging with secret redaction
│   ├── state/                     # Zustand state management
│   │   └── graphStore.ts               # Versioned store with rollback support
│   ├── test/                      # Automated test suites
│   │   ├── architectureIR.test.ts      # Intermediate representation tests
│   │   ├── fdbExecutor.test.ts         # FDB-v3 executor & invariant tests
│   │   ├── foodDeliveryInterruption.test.ts # Mid-flight interruption tests
│   │   ├── genericPlanner.test.ts      # Multi-tier system generation tests
│   │   ├── officialDemo.test.ts        # 3-stage evolution verification
│   │   └── voiceEngine.test.ts         # Sub-60ms conversational ack tests
│   ├── types/                     # TypeScript interfaces and type definitions
│   │   ├── graph.ts
│   │   ├── intent.ts
│   │   ├── ir.ts
│   │   └── operations.ts
│   └── voice/                     # Voice providers and speech coordination
│       ├── BrowserSpeechProvider.ts   # Web Speech API in-browser provider
│       ├── EventBridge.ts             # Telemetry & dispatch coordinator
│       ├── latencyTracker.ts          # Hardware-timed latency collector
│       ├── LiveKitVoiceProvider.ts    # WebRTC provider for LiveKit cloud
│       ├── MockVoiceProvider.ts       # Offline fallback for testing
│       └── types.ts
├── .env.example                   # Environment configuration template
├── .gitignore                     # Git ignore rules for node, Python, & IDEs
├── .oxlintrc.json                 # Oxlint configuration
├── index.html                     # HTML root entry point
├── LICENSE                        # MIT License
├── package.json                   # Project manifest, dependencies, & scripts
├── requirements.txt               # Root Python environment requirements
├── tailwind.config.js             # Tailwind CSS configuration
├── tsconfig.json                  # TypeScript compiler settings
└── vite.config.ts                 # Vite bundler configuration
```

---

## 7. Prerequisites & Installation

### Prerequisites
- **Node.js**: v18.0.0 or higher (v20+ recommended)
- **npm**: v9.0.0 or higher
- **Python**: 3.10+ (*optional*, only needed if running the standalone LiveKit Python agent worker)

### Installation Steps

1. Clone the repository:
   ```bash
   git clone https://github.com/<your-username>/mindspace.git
   cd mindspace
   ```

2. Install JavaScript dependencies using npm:
   ```bash
   npm install
   ```

---

## 8. Environment Variable Setup

MindSpace provides a pre-configured template in [`.env.example`](.env.example).

Copy the example file to create your local `.env`:
```bash
cp .env.example .env
```

### Configuration Variables

| Variable | Target | Description | Default / Fallback |
|---|---|---|---|
| `VITE_VOICE_MODE` | Frontend | Voice mode: `'mock'` \| `'browser'` \| `'livekit'` | `'mock'` (zero external dependencies) |
| `VITE_LIVEKIT_URL` | Frontend | LiveKit WebRTC Server URL (`wss://...`) | Optional; falls back to Web Speech or Mock |
| `LIVEKIT_URL` | Agent | Server URL for Python LiveKit worker | Optional |
| `LIVEKIT_API_KEY` | Agent | LiveKit API Key | Optional |
| `LIVEKIT_API_SECRET` | Agent | LiveKit API Secret | Optional |
| `OPENAI_API_KEY` | Agent | OpenAI API key for LiveKit LLM | Optional |
| `DEEPGRAM_API_KEY` | Agent | Deepgram API key for speech-to-text | Optional |

> [!NOTE]
> **Zero-Config Offline Mode:** MindSpace runs completely locally out-of-the-box without any cloud credentials. If no LiveKit keys are provided, it automatically uses the browser's native Web Speech API or the built-in offline simulation engine.

---

## 9. Local Development & Execution

### Start the Frontend Application
```bash
npm run dev
```
Open your browser at `http://localhost:5173`.

### Production Build
```bash
npm run build
```
Generates production assets in `dist/` with full type checking (`tsc -b`).

### Preview Production Build
```bash
npm run preview
```

### (Optional) Run the LiveKit Voice Agent Worker
If you wish to use an external LiveKit Cloud instance:
```bash
cd agent
python -m venv venv
# Windows:
.\venv\Scripts\activate
# Linux/macOS:
source venv/bin/activate

pip install -r requirements.txt
python agent.py dev
```

---

## 10. Verification & Automated Testing

MindSpace comes with extensive automated test suites verifying all architectural, voice, and scheduling invariants.

Run all tests sequentially:
```bash
npm run test:all
```

Run specific test suites:
```bash
npm run test:generic   # Multi-tier system generation & arbitrary NLP parsing
npm run test:ir        # Architecture Intermediate Representation (IR) validation
npm run test:food      # Real-time food delivery build & broker replacement
npm run test:demo      # Automated 3-stage evolution & interruption verification
npm run test:voice     # Sub-60ms conversational ack & telemetry verification
npm run test:fdb       # FDB-v3 interruptible executor, idempotency, & isolation
```

Run static analysis:
```bash
npm run lint
```

---

## 11. Troubleshooting

- **Microphone Permissions Denied:**
  If the browser displays a microphone permission warning, ensure you have granted microphone access in your browser's site settings. Alternatively, you can type any prompt directly into the input bar.
- **Port Conflict on 5173:**
  If port 5173 is in use, run `npm run dev -- --port 3000` to bind to an alternative port.
- **TypeScript Build Issues:**
  Ensure you are using Node.js v18+ and that `npm install` completed cleanly. You can run `npx tsc --noEmit` to verify type checking independently.

---

## 12. Security & Credential Hygiene

- **No Hardcoded Secrets:** MindSpace contains zero hardcoded API keys or credentials.
- **Automated Secrets Redaction:** The built-in benchmark logger (`src/fdb/logger.ts`) automatically scrubs API keys, bearer tokens, and password fields before writing JSONL outputs.
- **Local Credentials:** Always keep your actual API keys in `.env` (which is excluded via `.gitignore`) and never commit credentials to source control.

---

## 13. Future Roadmap

The following capabilities represent potential future enhancements:
- [ ] Export to Terraform and AWS CloudFormation templates.
- [ ] Direct export to Mermaid and PlantUML diagram formats.
- [ ] Collaborative multi-user WebRTC canvas editing.
- [ ] Custom LLM provider configuration via frontend settings modal.

---

## 14. License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

## 15. Acknowledgments

MindSpace was originally conceived and developed during the **Samsung PRISM GenAI Hackathon 2026** (Theme: Voice & Multimodal Full-Duplex Agents), where it was created to solve the challenge of conversational, interruptible system architecture design. It has since been refined and maintained as an independent open-source project.
