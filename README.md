# Speech Intelligence Dashboard

An intelligent, web-based audio intelligence and speech analytics application. The Speech Intelligence Dashboard enables users to record spoken audio directly through a browser or upload audio files to perform automated speech-to-text transcription, named entity extraction, IAB topic classification, sentence-level sentiment analysis, interactive gauge visualizations, and large language model (LLM) powered executive summarization.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Problem Statement](#problem-statement)
- [Proposed Solution](#proposed-solution)
- [Key Features](#key-features)
- [How It Works (Step-by-Step)](#how-it-works-step-by-step)
- [System Architecture](#system-architecture)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Technical Implementation](#technical-implementation)
- [Installation & Setup](#installation--setup)
- [Environment Configuration](#environment-configuration)
- [How to Run](#how-to-run)
- [Usage Instructions](#usage-instructions)
- [Example Input and Output](#example-input-and-output)
- [API Documentation](#api-documentation)
- [Database & Data Management](#database--data-management)
- [Testing & Verification](#testing--verification)
- [Actual Project Metrics](#actual-project-metrics)
- [Current Implementation Status](#current-implementation-status)
- [Limitations & Known Issues](#limitations--known-issues)
- [Future Improvements](#future-improvements)
- [Contributing](#contributing)
- [License](#license)
- [Acknowledgements](#acknowledgements)

---

## Project Overview

The **Speech Intelligence Dashboard** is a single-page application (SPA) built with React 18, Vite, and Chakra UI. It serves as an end-to-end voice processing and audio analysis workstation.

The application accepts spoken audio from two sources:
1. Live in-browser microphone recording.
2. Direct audio file uploads (`.mp3`, `.wav`, `.m4a`, etc.).

Once ingested, the audio is processed via cloud-based AI services:
- **AssemblyAI v2 REST API**: Performs asynchronous speech-to-text transcription, named entity detection, sentence-level sentiment classification, and contextual topic modeling based on Interactive Advertising Bureau (IAB) categories.
- **Google Gemini 1.5 Flash (`@google/generative-ai`)**: Generates a natural-language executive summary (constrained to 100–150 words) capturing the core message of the conversation.
- **Apache ECharts**: Renders an animated, multi-ring circular gauge visualizing the quantitative percentage split between positive, neutral, and negative sentiment.

---

## Problem Statement

Spoken audio is one of the most common mediums for meetings, customer support calls, interviews, podcasts, and lectures. However, extracting actionable intelligence from raw audio presents several challenges:
- **Time-Consuming Review**: Listening to long audio files to extract key takeaways or determine speaker intent requires substantial manual effort.
- **Lack of Searchability & Structure**: Unprocessed audio cannot be indexed, searched, or cross-referenced with ease.
- **Subjective Sentiment Evaluation**: Manually evaluating conversational tone across multi-sentence dialogue is inconsistent and lacks quantitative measurement.
- **Information Overload**: Extracting key entities (people, places, products) and high-level themes requires reading through extensive transcripts.

---

## Proposed Solution

The Speech Intelligence Dashboard resolves these issues by delivering an automated, multi-tiered audio analytics pipeline accessible through an intuitive graphical user interface:
- **Asynchronous Cloud Transcription**: Offloads compute-heavy speech recognition to AssemblyAI, returning word-for-word transcriptions with high accuracy.
- **Contextual Entity & Topic Tagging**: Automatically surfaces recognized entities with hover tooltips and categorizes discussion domains using standardized IAB taxonomy.
- **Color-Coded Sentiment Inspection**: Highlights individual sentences in the transcript according to sentiment polarity (green for positive, pink for negative, gray for neutral) and provides aggregated sentiment groups.
- **Visual Analytics Dashboard**: Calculates real-time sentiment distribution percentages and displays them in a multi-gauge chart.
- **Generative AI Summarization**: Uses Google's Gemini 1.5 Flash model to synthesize the entire dialogue into a structured 100–150 word summary.

---

## Key Features

- **Dual Audio Input Channels**:
  - In-browser microphone recording powered by `react-voice-recorder` within an interactive modal dialog.
  - Native file picker supporting all standard audio MIME types (`audio/*`).
- **Asynchronous Polling Engine**:
  - Manages file uploads, initiates transcription requests, and polls the AssemblyAI status endpoint every 5 seconds until processing is complete.
- **Interactive Transcript Viewer**:
  - Sentences are marked with sentiment-specific background colors (`lightgreen`, `pink`, `lightgray`).
  - Entities (names, locations, dates, organizations) are dynamically detected within sentences and rendered as bold text with Chakra UI tooltips displaying their classification.
- **Keyword & Entity Discovery**:
  - Deduplicates all recognized entities across the transcript.
  - Displays detected entities alongside their associated sentence sentiment and entity type tags.
- **IAB Topic Categorization**:
  - Analyzes thematic content against the IAB category hierarchy.
  - Filters and displays topic tags meeting a confidence score threshold greater than 0.6 (`> 0.6`).
- **Categorized Sentiment Breakdown**:
  - Groups sentences into distinct positive, neutral, and negative cards for rapid sentiment auditing.
- **Dynamic Multi-Ring Gauge Chart**:
  - Built with Apache ECharts.
  - Calculates and animates exact percentage shares for Positive, Neutral, and Negative sentiments to two decimal places.
- **AI-Powered Executive Summary**:
  - Generates a 100–150 word natural-language synthesis of the audio using Google's `gemini-1.5-flash` model.

---

## How It Works (Step-by-Step)

```
+-----------------------------------------------------------------------------------+
|                                  USER INTERFACE                                   |
|   1. Record Audio (Microphone)       OR       2. Select Audio File (.wav, .mp3)   |
+-----------------------------------------+-----------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|                           INGESTION & ASSEMBLYAI PIPELINE                         |
|   Step 1: POST binary audio payload to https://api.assemblyai.com/v2/upload        |
|   Step 2: Receive upload_url (hosted audio resource)                              |
|   Step 3: POST to https://api.assemblyai.com/v2/transcript                        |
|           Payload: { audio_url, sentiment_analysis: true,                          |
|                      entity_detection: true, iab_categories: true }               |
|   Step 4: Poll GET https://api.assemblyai.com/v2/transcript/{id} every 5 seconds |
|           until status === "completed"                                            |
+-----------------------------------------+-----------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|                               TRANSCRIPT DISPATCH                                 |
|   Full JSON response populated into React state (transcript)                      |
+-----------------------------------------+-----------------------------------------+
                                          |
         +--------------------------------+--------------------------------+
         |                                |                                |
         v                                v                                v
+------------------+            +------------------+            +------------------+
|  TAB 1: RESULTS  |            |  TAB 2: KEYWORDS |            | TAB 3: SENTIMENT |
| Highlighted text |            | Unique entities  |            | Categorized lists|
| Sentiment colors |            | IAB topics (>0.6)|            | Positive/Neutral/|
| Entity tooltips  |            | Entity metadata  |            | Negative cards   |
+------------------+            +------------------+            +------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|                         TAB 4: REPORTING & VISUALIZATION                          |
|   Sub-Process A (Gemini 1.5 Flash):                                               |
|     POST prompt to Google Generative AI -> Returns 100-150 word summary           |
|   Sub-Process B (Apache ECharts):                                                 |
|     Computes % positive, neutral, negative -> Renders circular gauge chart       |
+-----------------------------------------------------------------------------------+
```

### Detailed Execution Steps

1. **Audio Acquisition**:
   - **File Upload**: The user selects an audio file via `<input type="file" accept="audio/*">`. The file is stored in React state.
   - **Microphone Recording**: The user clicks "Record Audio", launching a Chakra UI `AlertDialog` containing the `Recorder` component. On completion, the recorded audio blob is stored.
2. **Uploading to AssemblyAI**:
   - The audio blob or file is sent via an HTTP `POST` request to `https://api.assemblyai.com/v2/upload` with an authorization token.
   - AssemblyAI stores the file and returns a temporary `upload_url`.
3. **Triggering the Analysis Pipeline**:
   - An HTTP `POST` request is dispatched to `https://api.assemblyai.com/v2/transcript` containing the `audio_url` and feature flags enabling sentiment analysis, entity detection, and IAB topic categorization.
   - AssemblyAI registers the job and returns a unique transcript ID (`id`).
4. **Polling for Completion**:
   - The application enters a polling loop using `setTimeout(resolve, 5000)`, making repeated `GET` calls to `https://api.assemblyai.com/v2/transcript/${transcriptId}`.
   - The status updates through `"queued"` and `"processing"` until reaching `"completed"` (or `"failed"`).
5. **State Update & Tab Activation**:
   - The comprehensive transcript payload is assigned to the `transcript` state variable.
   - The UI transitions from the upload/progress state to display a four-tab analysis dashboard.
6. **Downstream AI & Visual Processing**:
   - In Tab 4 ("Reporting and Visualization"), the text transcript is passed to `src/helper/Symmary.js`, which initiates a call to Google Gemini 1.5 Flash requesting a 100–150 word summary.
   - Concurrently, `src/Components/Chart.jsx` calculates sentiment proportions from `transcript.sentiment_analysis_results` and renders the ECharts gauge chart.

---

## System Architecture

```mermaid
flowchart TD
    subgraph Client ["Client Browser (React 18 SPA)"]
        UI["App.jsx (Root Component)"]
        Sidebar["DrawerBox.jsx (Navigation & Controls)"]
        Recorder["react-voice-recorder (Audio Recorder)"]
        FileInput["Native File Input (accept='audio/*')"]
        StatusComp["Status.jsx (Progress Indicator)"]

        subgraph TabsView ["Tabs Container"]
            T1["Results.jsx & Highlights.jsx\n(Annotated Transcription)"]
            T2["Keyword.jsx, Topic.jsx, Entity.jsx\n(Entity & Topic Extraction)"]
            T3["SentimentalAnalysis.jsx\n(Categorized Sentiment Buckets)"]
            T4["Reports.jsx\n(Reports & Visualization)"]
        end

        subgraph VisualAndAI ["Reporting Components"]
            ECharts["Chart.jsx\n(Apache ECharts Multi-Gauge)"]
            GeminiHelper["Summary.jsx & Symmary.js\n(Gemini 1.5 Flash Client)"]
        end
    end

    subgraph AssemblyAI ["AssemblyAI Cloud Platform"]
        AA_Upload["POST /v2/upload"]
        AA_Transcript["POST /v2/transcript"]
        AA_Poll["GET /v2/transcript/:id"]
    end

    subgraph GoogleAI ["Google AI Platform"]
        GeminiAPI["gemini-1.5-flash API"]
    end

    %% Audio Inputs
    Sidebar --> Recorder
    Sidebar --> FileInput
    Recorder -->|Audio Blob| UI
    FileInput -->|Audio File| UI

    %% AssemblyAI Communication
    UI -->|1. Upload Audio| AA_Upload
    AA_Upload -->|Returns upload_url| UI
    UI -->|2. Request Analysis| AA_Transcript
    AA_Transcript -->|Returns transcript ID| UI
    UI -->|3. Poll Status (every 5s)| AA_Poll
    AA_Poll -->|Returns Completed Payload| UI

    %% UI State Distribution
    UI --> StatusComp
    UI --> TabsView
    TabsView --> T1
    TabsView --> T2
    TabsView --> T3
    TabsView --> T4

    %% Tab 4 Integrations
    T4 --> ECharts
    T4 --> GeminiHelper
    GeminiHelper -->|Generate Summary| GeminiAPI
    GeminiAPI -->|Summary Text| GeminiHelper
```

---

## Technology Stack

### Languages & Runtime
| Technology | Version / Specification | Role in Project |
| :--- | :--- | :--- |
| **JavaScript (ESModules)** | ECMAScript 2020 / ESNext | Primary programming language across all source files |
| **Node.js** | `>= 18.0.0` (Verified on `v22.21.0`) | Development runtime and package execution environment |
| **npm** | `10.9.4` | Package manager |

### Core Frontend Frameworks & Libraries
| Library | Version (package.json) | Role in Project |
| :--- | :--- | :--- |
| **React** | `^18.3.1` | Declarative component architecture and state management |
| **React DOM** | `^18.3.1` | DOM renderer for React components |
| **Vite** | `^5.4.1` | Next-generation frontend build tool and dev server |
| **@vitejs/plugin-react**| `^4.3.1` | Babel-based Fast Refresh plugin for Vite |

### UI Design System & Component Libraries
| Library | Version (package.json) | Role in Project |
| :--- | :--- | :--- |
| **@chakra-ui/react** | `^2.8.2` | Accessible, modular UI component library (Tabs, Alerts, Dialogs, Boxes) |
| **@emotion/react** | `^11.13.3` | CSS-in-JS style engine required by Chakra UI |
| **@emotion/styled** | `^11.13.0` | Styled component primitives for Chakra UI |
| **framer-motion** | `^11.3.31` | Animation library powering Chakra UI transitions |

### Audio Processing & Data Visualization
| Library | Version (package.json) | Role in Project |
| :--- | :--- | :--- |
| **react-voice-recorder**| `^2.1.2` | In-browser audio recording, playback, and WAV blob generation |
| **echarts** | `^5.5.1` | Industrial-grade charting library for multi-ring circular gauge visualization |

### Cloud AI & Machine Learning Services
| Integration / SDK | Version / API Endpoint | Purpose |
| :--- | :--- | :--- |
| **AssemblyAI REST API v2** | `api.assemblyai.com/v2` | Speech-to-text, entity detection, sentiment analysis, IAB topic classification |
| **@google/generative-ai**| `^0.17.1` | Official Google SDK used to prompt the `gemini-1.5-flash` model for summarization |

### Linting & Code Quality
| Tool | Version (package.json) | Role in Project |
| :--- | :--- | :--- |
| **ESLint** | `^9.9.0` | Pluggable linting utility with flat configuration |
| **eslint-plugin-react** | `^7.35.0` | React-specific linting rules |
| **eslint-plugin-react-hooks** | `^5.1.0-rc.0` | Rules of Hooks validation |
| **eslint-plugin-react-refresh** | `^0.4.9` | Hot Module Replacement safety enforcement |

---

## Project Structure

```
speech-intelligence-dashboard/
├── .gitignore                      # Git exclusion rules (node_modules, build artifacts, logs)
├── eslint.config.js                # ESLint 9 flat configuration with React rules
├── index.html                      # Single-page application entry HTML template
├── package-lock.json               # Deterministic dependency tree lockfile (5,935 lines)
├── package.json                    # Project metadata, dependencies, and npm scripts
├── README.md                       # Comprehensive project technical documentation
├── vite.config.js                  # Vite bundler configuration using @vitejs/plugin-react
├── public/
│   └── vite.svg                    # Default Vite application icon asset
└── src/
    ├── App.css                     # Component-level styling overrides (currently 0 bytes)
    ├── App.jsx                     # Root application container, state management, and API polling
    ├── index.css                   # Global CSS reset and styling rules (currently 0 bytes)
    ├── index.jsx                   # React 18 DOM mount point with Chakra ColorModeScript
    ├── Components/
    │   ├── Chart.jsx               # Apache ECharts gauge chart rendering sentiment percentages
    │   ├── DrawerBox.jsx           # Sidebar navigation, audio file selector, and recording dialog
    │   ├── Entity.jsx              # Reusable entity badge with Chakra UI tooltip
    │   └── Summary.jsx             # React component triggering and displaying Gemini summary
    ├── helper/
    │   └── Symmary.js              # Google Generative AI integration helper (calls gemini-1.5-flash)
    └── Services/
        ├── Highlights.jsx          # Highlights transcript sentences with sentiment colors & entity tooltips
        ├── Keyword.jsx             # Extracts unique entities and renders Topic & Entity views
        ├── Reports.jsx             # Tab panel coordinating the summary view and ECharts visual
        ├── Results.jsx             # Tab panel rendering the full annotated transcript
        ├── SentimentalAnalysis.jsx # Tab panel grouping sentences into Positive/Neutral/Negative cards
        ├── Status.jsx              # Visual progress bar and status indicator for upload/transcription
        └── Topic.jsx               # Filters and displays IAB category topics (confidence > 0.6)
```

---

## Technical Implementation

### 1. Root Controller & Polling Engine (`src/App.jsx`)
`App.jsx` acts as the single source of truth for application state:
- Maintains state for audio upload parameters (`audioDetails`, `file`), transcription payload (`transcript`), loading indicators (`isLoading`), and recording dialogs (`isRecording`).
- **Direct File Upload (`handleDirectUpload`)**: Packages the selected file into a `FormData` envelope and posts it directly to AssemblyAI's upload endpoint.
- **Microphone Audio Upload (`handleRecordedAudioUpload`)**: Streams the binary blob directly in the request body to AssemblyAI's upload endpoint.
- **Asynchronous Polling Loop (`getTranscription`)**: After creating the transcription request, executes a loop awaiting 5,000 milliseconds between `GET` polling requests until `statusData.status` equals `"completed"` or `"failed"`.

### 2. Sidebar Navigation & Recording Controller (`src/Components/DrawerBox.jsx`)
- Implements a fixed-position 250px-wide sidebar.
- Contains the brand heading and triggers for recording or uploading audio.
- Integrates `react-voice-recorder` within a Chakra UI `AlertDialog` modal overlay to capture clean audio blobs from the user's microphone.
- Manages local modal state, cancel refs, and triggers the parent upload handlers upon completion.

### 3. Sentence Sentiment & Entity Highlighting (`src/Services/Highlights.jsx` & `Results.jsx`)
- Evaluates each sentence in `transcript.sentiment_analysis_results`.
- Uses a sentiment color palette:
  - `POSITIVE`: `'lightgreen'`
  - `NEGATIVE`: `'pink'`
  - `NEUTRAL`: `'lightgray'`
- Constructs a dynamic regular expression `new RegExp('(${entityText.join("|")})', 'g')` to isolate recognized entities from normal sentence prose.
- Wraps matching entities in bold `<Text>` components inside Chakra UI `<Tooltip>` wrappers configured to reveal the entity's classification (`entity_type`) on hover.

### 4. Keyword & Topic Categorization (`src/Services/Keyword.jsx` & `Topic.jsx`)
- Extracts unique entities using a JavaScript `Set` over `transcript.entities.map(e => e.text)`.
- Matches each entity back to its contextual sentence sentiment and entity type.
- In `Topic.jsx`, parses AssemblyAI's `iab_categories_result.summary` dictionary.
- Applies a strict confidence filter: `transcript.iab_categories_result.summary[topic] > 0.6`.
- Extracts the leaf taxonomy category via string splitting (`topic.split('>').pop()`) and renders styled Chakra UI `Tag` pills.

### 5. Categorized Sentiment Auditing (`src/Services/SentimentalAnalysis.jsx`)
- Partitions all transcribed sentences into three distinct arrays: `POSITIVE`, `NEUTRAL`, and `NEGATIVE`.
- Renders three styled visual cards:
  - Positive card (`bg="green.100"`)
  - Neutral card (`bg="gray.100"`)
  - Negative card (`bg="red.100"`)
- Renders each sentence as a distinct list item with left-aligned callouts.

### 6. Dynamic Gauge Visualization (`src/Components/Chart.jsx`)
- Initializes an Apache ECharts instance against the `#sentiment-chart` DOM container.
- Calculates exact sentiment ratios:
  $$\text{Percentage} = \left(\frac{\text{Count}}{\text{Total}}\right) \times 100$$
- Visualizes the proportions using a 3-tier gauge series:
  - Outer ring: Positive (Green, offset `['0%', '-40%']`)
  - Middle ring: Neutral (Gray, offset `['0%', '0%']`)
  - Inner ring: Negative (Red, offset `['0%', '40%']`)
- Features animated value indicators with 2 decimal precision (`{value}%`) and cleans up memory on component unmount via `myChart.dispose()`.

### 7. LLM Summarization (`src/helper/Symmary.js` & `src/Components/Summary.jsx`)
- Connects to Google's Generative AI API using `@google/generative-ai`.
- Initializes the `gemini-1.5-flash` model.
- Dispatches a prompt:
  ```text
  "${transcription} Make a summary of the transcript given to you in about 100-150 words"
  ```
- Renders the resulting synthesized text inside the "Reporting and Visualization" tab.

---

## Installation & Setup

### Prerequisites
- **Node.js**: Version `18.0.0` or higher (verified on `v22.21.0`).
- **npm**: Version `9.0.0` or higher (verified on `10.9.4`).
- **Supported Browsers**: Google Chrome, Mozilla Firefox, Microsoft Edge, or Apple Safari with microphone access enabled.

### Step-by-Step Installation

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/KrishPatel-AI/speech-intelligence-dashboard.git
   cd speech-intelligence-dashboard
   ```

2. **Install Dependencies**:
   ```bash
   npm install
   ```

---

## Environment Configuration

> [!WARNING]
> In the current codebase, third-party API keys are hardcoded directly inside frontend source files:
> - `src/App.jsx` (`ASSEMBLY_API_KEY`)
> - `src/helper/Symmary.js` (Google Generative AI key)
>
> For production deployments or public hosting, hardcoding credentials in client-side bundles exposes them to the public. It is strongly recommended to refactor these variables into a `.env` file or proxy them through a backend server.

### Recommended Environment Setup (Vite)

1. Create a `.env` file in the project root:
   ```env
   VITE_ASSEMBLY_API_KEY=your_assemblyai_api_key_here
   VITE_GEMINI_API_KEY=your_google_gemini_api_key_here
   ```

2. Update `.gitignore` to ensure `.env` is never committed:
   ```
   .env
   .env.local
   ```

3. Access the environment variables in source code:
   ```javascript
   // In src/App.jsx:
   const ASSEMBLY_API_KEY = import.meta.env.VITE_ASSEMBLY_API_KEY;

   // In src/helper/Symmary.js:
   const genAI = new GoogleGenerativeAI(import.meta.env.VITE_GEMINI_API_KEY);
   ```

---

## How to Run

### Development Mode
Start the Vite local development server with Hot Module Replacement (HMR):
```bash
npm run dev
```
The application will be accessible at:
```
http://localhost:5173/
```

### Production Build
Compile and bundle the production-ready static assets:
```bash
npm run build
```
The compiled output is emitted to the `dist/` directory.

### Preview Production Build
Serve the local `dist/` build locally to verify production behavior:
```bash
npm run preview
```

### Linting
Run ESLint to inspect source files for code style and hook rules:
```bash
npm run lint
```

---

## Usage Instructions

1. **Launch the Web Application**:
   Run `npm run dev` and navigate to `http://localhost:5173`.
2. **Provide Audio**:
   - **Option A (File Upload)**: Click "Choose File" in the left sidebar, select a compatible audio file (such as `.mp3`, `.wav`, or `.m4a`), and click **Upload**.
   - **Option B (Microphone Recording)**: Click **Record Audio** in the left sidebar. In the dialog, start recording, speak into the microphone, stop recording when finished, and click **Upload**.
3. **Monitor Ingestion**:
   - The status bar will transition through `"uploading..."`, displaying an indeterminate progress bar while the file is processed and transcribed.
4. **Explore Analysis Tabs**:
   - **Transcription Engine**: Read through the sentence-by-sentence transcription. Sentences are color-coded by sentiment. Hover over bold words to inspect detected entities.
   - **Keyword Extraction**: Review detected IAB category tags and view all unique entities recognized throughout the dialogue.
   - **Sentiment Analysis**: Review three segmented cards listing all sentences categorized as Positive, Neutral, or Negative.
   - **Reporting and Visualization**:
     - Read the 100–150 word executive summary generated by Google Gemini 1.5 Flash.
     - View the multi-gauge circular chart illustrating the exact percentage breakdown of sentiments.

---

## Example Input and Output

### Sample Input
- **File Format**: `customer_feedback.mp3` (or live microphone recording).
- **Spoken Content**:
  > *"Good morning. I am calling from London to talk about our experience with Acme Corporation. The service delivered by John yesterday was fantastic and very fast. However, the billing system is quite confusing and slow to update."*

### Sample AssemblyAI Response Structure
```json
{
  "id": "7b8f9e2a-1234-4abc-9def-1234567890ab",
  "status": "completed",
  "text": "Good morning. I am calling from London to talk about our experience with Acme Corporation. The service delivered by John yesterday was fantastic and very fast. However, the billing system is quite confusing and slow to update.",
  "sentiment_analysis_results": [
    {
      "text": "Good morning.",
      "sentiment": "NEUTRAL"
    },
    {
      "text": "I am calling from London to talk about our experience with Acme Corporation.",
      "sentiment": "NEUTRAL"
    },
    {
      "text": "The service delivered by John yesterday was fantastic and very fast.",
      "sentiment": "POSITIVE"
    },
    {
      "text": "However, the billing system is quite confusing and slow to update.",
      "sentiment": "NEGATIVE"
    }
  ],
  "entities": [
    { "text": "London", "entity_type": "location" },
    { "text": "Acme Corporation", "entity_type": "organization" },
    { "text": "John", "entity_type": "person" },
    { "text": "yesterday", "entity_type": "date" }
  ],
  "iab_categories_result": {
    "summary": {
      "BusinessAndFinance>Business>CustomerService": 0.89,
      "Technology>Software": 0.65
    }
  }
}
```

### Resulting Output in Speech Intelligence Dashboard
- **Transcription Tab**: Renders sentences with corresponding background tints. Hovering over `"London"` reveals a tooltip reading `location`; hovering over `"John"` reveals `person`.
- **Keyword Tab**:
  - Topics Detected: `CustomerService`, `Software`
  - Entities Listed: `London` (neutral), `Acme Corporation` (neutral), `John` (positive), `yesterday` (positive).
- **Sentiment Tab**:
  - **Positive (1 sentence)**: *"The service delivered by John yesterday was fantastic and very fast."*
  - **Neutral (2 sentences)**: *"Good morning."*, *"I am calling from London..."*
  - **Negative (1 sentence)**: *"However, the billing system is quite confusing and slow to update."*
- **Reporting & Visualization Tab**:
  - **Chart**: Multi-gauge displays:
    - Positive: `25.00%`
    - Neutral: `50.00%`
    - Negative: `25.00%`
  - **Gemini Summary**: A cohesive 100–150 word summary synthesizing the praise for individual staff performance alongside constructive criticism of the billing infrastructure.

---

## API Documentation

The application integrates with two external REST/SDK endpoints.

### 1. AssemblyAI v2 REST API

#### Endpoint: Upload Audio
- **URL**: `https://api.assemblyai.com/v2/upload`
- **Method**: `POST`
- **Headers**:
  - `authorization`: AssemblyAI API Key
- **Request Body**: Binary audio payload or `FormData` containing the audio file.
- **Response**:
  ```json
  {
    "upload_url": "https://cdn.assemblyai.com/upload/..."
  }
  ```

#### Endpoint: Submit Transcription Request
- **URL**: `https://api.assemblyai.com/v2/transcript`
- **Method**: `POST`
- **Headers**:
  - `authorization`: AssemblyAI API Key
  - `Content-Type`: `application/json`
- **Request Body**:
  ```json
  {
    "audio_url": "https://cdn.assemblyai.com/upload/...",
    "sentiment_analysis": true,
    "entity_detection": true,
    "iab_categories": true
  }
  ```
- **Response**:
  ```json
  {
    "id": "transcript_id_string",
    "status": "queued"
  }
  ```

#### Endpoint: Poll Transcription Status
- **URL**: `https://api.assemblyai.com/v2/transcript/{transcriptId}`
- **Method**: `GET`
- **Headers**:
  - `authorization`: AssemblyAI API Key
- **Response**: Object containing `status` (`"queued"`, `"processing"`, `"completed"`, or `"failed"`), `text`, `sentiment_analysis_results`, `entities`, and `iab_categories_result`.

### 2. Google Gemini Generative AI API

- **Library**: `@google/generative-ai`
- **Model**: `gemini-1.5-flash`
- **Method**: `model.generateContent(prompt)`
- **Prompt**:
  ```text
  "${transcription} Make a summary of the transcript given to you in about 100-150 words"
  ```
- **Response Format**: Markdown / plain text string summarizing the transcript.

---

## Database & Data Management

- **Persistence Layer**: None. The project operates entirely as a stateless client-side single-page application.
- **State Management**:
  - Uses standard React component state hooks (`useState`) located at the root in `App.jsx`.
  - Audio files, recorded audio blobs, and JSON transcription responses exist purely in browser memory during the active session.
  - Resetting or reloading the browser page clears all audio data and results.
- **Data Privacy**: Audio files and text transcripts are transmitted directly from the user's browser to AssemblyAI and Google AI servers. No intermediate application server or database stores the payloads.

---

## Testing & Verification

- **Automated Test Suites**: Currently, there are **no automated test suites** (Jest, Vitest, Cypress, or Playwright) configured in this repository.
- **Linting Verification**: Configured via ESLint 9 (`eslint.config.js`). Source files can be verified using:
  ```bash
  npm run lint
  ```
- **Manual Verification**:
  - The application architecture was validated through static code analysis of all 14 source files in `src/`.
  - Node.js runtime compatibility verified with Node.js `v22.21.0` and npm `10.9.4`.

---

## Actual Project Metrics

These metrics are calculated directly from the project repository:

| Metric Category | Verified Value | Calculation Basis / Source |
| :--- | :--- | :--- |
| **Total Project Files** | 24 files | `Get-ChildItem -Recurse -File` (excluding `.git`) |
| **Source Files (`src/`)** | 16 files | 14 `.jsx`/`.js` source files, 2 empty `.css` files |
| **Source Lines of Code (SLOC)** | 843 lines | Cumulative line count of all 14 JavaScript/JSX files in `src/` |
| **Largest Source File** | `src/App.jsx` (211 lines, 7,338 bytes) | Line and byte count analysis |
| **UI Components Count** | 4 components | `src/Components/` (`Chart`, `DrawerBox`, `Entity`, `Summary`) |
| **Service Views Count** | 7 views/services | `src/Services/` (`Highlights`, `Keyword`, `Reports`, `Results`, `SentimentalAnalysis`, `Status`, `Topic`) |
| **Helper Modules Count** | 1 module | `src/helper/Symmary.js` |
| **AssemblyAI Polling Interval** | 5,000 ms (5 seconds) | Hardcoded `setTimeout(resolve, 5000)` in `App.jsx` |
| **Topic Confidence Threshold** | `> 0.6` (60%) | Hardcoded filter in `src/Services/Topic.jsx` |
| **Gemini Summary Target** | 100–150 words | Defined in prompt string in `src/helper/Symmary.js` |

---

## Current Implementation Status

### Fully Implemented & Functional
- [x] In-browser microphone audio recording with playback preview and reset controls.
- [x] Direct audio file upload mechanism (`accept="audio/*"`).
- [x] Asynchronous AssemblyAI upload, transcription trigger, and status polling.
- [x] Sentence-level sentiment color-coding (`lightgreen`, `pink`, `lightgray`).
- [x] Dynamic entity identification with hover tooltips showing entity types.
- [x] IAB topic classification filtered to entries with confidence `> 0.6`.
- [x] Categorized sentiment lists (positive, neutral, negative cards).
- [x] Animated multi-ring sentiment gauge chart powered by Apache ECharts.
- [x] Automated 100–150 word transcript summarization using Google Gemini 1.5 Flash.

### Partially Implemented / Identified Inconsistencies
- [x] **Regex Entity Highlighting**: `Highlights.jsx` uses `new RegExp('(${entityText.join("|")})', 'g')`. If entity names contain unescaped regex special characters (such as parentheses or brackets), a syntax error can occur.
- [x] **Error State Typing**: In `App.jsx`, when an error occurs during upload or transcription, `transcript` is assigned a string (e.g. `"Error occurred during file upload."`). However, downstream components expect `transcript` to be an object (`transcript.status`), causing `status` in `Status.jsx` to be undefined.
- [x] **File Naming & Spelling**: `src/helper/Symmary.js` is misspelled (intended as `Summary.js`), `hanldeReset` is misspelled in `App.jsx` and `DrawerBox.jsx`, and `SentimentalAnalysis.jsx` uses non-standard phrasing for sentiment analysis.
- [x] **Commented-out Code**: In `Results.jsx`, `<Topic transcript={transcript} />` is commented out (topics are displayed inside the Keyword Extraction tab instead).

### Planned / Not Yet Implemented
- [ ] Automated testing suite (unit tests with Vitest / React Testing Library, E2E tests with Playwright).
- [ ] Secure backend API proxy or serverless functions to safeguard third-party API credentials.
- [ ] Export functionality (saving transcripts and summaries as PDF, Markdown, or JSON).
- [ ] Responsive layout adaptations for mobile viewports (e.g., collapsible drawer).

---

## Limitations & Known Issues

1. **Client-Side Credential Exposure**:
   - AssemblyAI and Google Gemini API keys are hardcoded in client-side code. Any user inspecting network requests or bundle source code can extract these credentials.
2. **Infinite Polling Vulnerability**:
   - The polling loop in `App.jsx` continues indefinitely until AssemblyAI returns `"completed"` or `"failed"`. There is no maximum timeout or retry limit, which could cause prolonged background execution if an API request stalls.
3. **No Automated Test Coverage**:
   - The project currently lacks automated unit or integration tests, meaning regressions must be identified through manual testing.
4. **Desktop-Centric Layout**:
   - The layout uses a fixed sidebar width (`w="250px"`) and an offset main content margin (`ml={250}` with `width={1000}`), which causes horizontal overflow on mobile or small viewports.
5. **No Long-Term Storage**:
   - Transcripts, summaries, and audio recordings are not persisted to a database or local storage; refreshing the page clears all data.

---

## Future Improvements

- **Security Hardening**:
  - Move API keys to environment variables and route requests through a lightweight backend API (e.g., Node/Express or Next.js API routes) to keep API keys secure.
- **Robust Error Handling & Timeout Guards**:
  - Implement an exponential backoff strategy and a maximum timeout (e.g., 5 minutes) for the AssemblyAI polling loop.
- **Transcript Export**:
  - Add download buttons to export transcripts, entity lists, and Gemini summaries as PDF, CSV, or formatted Markdown.
- **Responsive Layout Overhaul**:
  - Replace the fixed desktop sidebar with a responsive Chakra UI drawer that collapses into a hamburger menu on smaller screens.
- **Testing Infrastructure**:
  - Introduce Vitest and React Testing Library to test component rendering, regex entity splitting, and chart initialization.

---

## Contributing

Contributions are welcome. To maintain code quality and stability, please adhere to the following workflow:

1. **Fork the Repository** on GitHub.
2. **Create a Feature Branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. **Commit Your Changes**:
   Ensure meaningful commit messages:
   ```bash
   git commit -m "feat: implement responsive drawer navigation"
   ```
4. **Validate Code Quality**:
   Run ESLint before pushing:
   ```bash
   npm run lint
   ```
5. **Push and Open a Pull Request**:
   Push your branch to your fork and submit a PR to `main` with a clear explanation of changes, test steps, and screenshots if applicable.

---

## License

This project currently has no explicit open-source license file (`LICENSE`) in the repository. The `package.json` designates the project as private:
```json
"private": true
```
All rights are reserved by the original project authors. Consult the repository owner before reusing or distributing this code.

---

## Acknowledgements

- **[AssemblyAI](https://www.assemblyai.com/)**: For providing speech-to-text, entity detection, sentiment analysis, and IAB topic categorization APIs.
- **[Google Generative AI](https://ai.google.dev/)**: For the Gemini 1.5 Flash model powering the automated summarization engine.
- **[Apache ECharts](https://echarts.apache.org/)**: For the charting library powering the sentiment gauge visualization.
- **[Chakra UI](https://chakra-ui.com/)**: For accessible React UI components and theming primitives.
- **[Vite](https://vitejs.dev/)**: For fast local development and optimized build tooling.
