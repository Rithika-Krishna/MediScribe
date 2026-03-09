# 🩺 MediScribe AI

> **Real-time AI-powered telemedicine platform** — connects doctors and patients via WebRTC video call, auto-transcribes the consultation, and generates a prescription using AI.

---

## 📌 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [System Architecture](#system-architecture)
- [Project Structure](#project-structure)
- [How It Works](#how-it-works)
- [Getting Started](#getting-started)
- [Prerequisites](#prerequisites)
- [Installation & Setup](#installation--setup)
- [Running the App](#running-the-app)
- [Future Improvements](#future-improvements)

---

## Overview

MediScribe AI is a browser-based telemedicine application that enables real-time video consultations between doctors and patients. During the call, the doctor's speech is captured, transcribed using Google Speech-to-Text, and processed by an AI model that identifies diseases and medications mentioned in the conversation. At the end of the call, an auto-generated prescription is delivered to the patient.

---

## Features

- 🎥 **Peer-to-peer video calling** via WebRTC — no media server needed
- 🎙️ **Real-time speech capture** from the doctor's microphone using MediaRecorder
- 📝 **Automatic transcription** using Google Speech-to-Text (via Python SpeechRecognition)
- 🤖 **AI-powered prescription generation** — extracts diseases and drugs from transcribed text
- 💊 **Instant prescription delivery** to the patient via Socket.IO at call end
- 🔁 **Real-time signaling** using Socket.IO for WebRTC handshake (offer/answer/ICE)

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | HTML, CSS |
| Real-time Communication | WebRTC (peer-to-peer) |
| Signaling & Events | Node.js + Socket.IO |
| Web Server | Express.js |
| Audio Transcription | Python, SpeechRecognition (Google STT) |
| AI / NLP | Python — keyword-based NER on Disease.csv & Drugs.csv |
| Data | Disease.csv, Drugs.csv |
| Tunneling (Dev) | ngrok |

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        BROWSER                              │
│                                                             │
│   ┌──────────────┐        WebRTC P2P        ┌────────────┐ │
│   │  Doctor Page │◄────────────────────────►│Patient Page│ │
│   │  Doctor.html │                          │Patient.html│ │
│   └──────┬───────┘                          └─────┬──────┘ │
│          │ Socket.IO                              │Socket.IO│
└──────────┼─────────────────────────────────────── ┼────────┘
           │                                        │
           ▼                                        ▼
┌─────────────────────────────────────────────────────────────┐
│                   Node.js Server (server.js)                │
│          Express + Socket.IO — port 3000 via ngrok          │
│                                                             │
│  Routes: /mediscribe  /doc  /pat                            │
│  Events: offer, answer, candidate, audioData, endCall       │
└───────────────────────────┬─────────────────────────────────┘
                            │ Socket.IO (Python client)
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                  Python AI Backend                          │
│                                                             │
│  secondary_model.py  →  Google STT  →  transcribed text     │
│  primary_model.py    →  Disease.csv + Drugs.csv matching    │
│                      →  prescription string generated       │
└─────────────────────────────────────────────────────────────┘
```

---

## Project Structure

```
MediScribe/
│
├── Main_page.html          # Landing page — user selects Doctor or Patient role
├── Main_styles.css
│
├── Doctor.html             # Doctor's UI — start call, video feed, end call
├── Doctor_styles.css
│
├── Patient.html            # Patient's UI — join call, receive prescription modal
├── Patient_styles.css
│
├── server.js               # Node.js + Express + Socket.IO signaling server
├── webrtc_setup.js         # WebRTC logic for the Patient side
├── Signalwebrtcint.js      # WebRTC logic for the Doctor side (offer, ICE, audio send)
│
├── primary_model.py        # AI model — extracts diseases/drugs, generates prescription
├── secondary_model.py      # Audio handler — transcribes audio via Google STT
│
├── Disease.csv             # Dataset of disease names for keyword matching
├── Drugs.csv               # Dataset of drug names for keyword matching
│
├── package.json
└── Resources/
    └── favicon.ico
```

---

## How It Works

### Step-by-step Flow

1. **User visits** `/mediscribe` and selects their role — Doctor or Patient
2. **Doctor** opens `/doc`, browser connects to Socket.IO server
3. **Patient** opens `/pat`, peer connection is initialized, patient waits for offer
4. **Doctor clicks Start Call** → `getUserMedia()` captures video + audio → `RTCPeerConnection` created
5. **WebRTC Offer** is created and sent via `socket.emit('offer')` → server broadcasts to patient
6. **Patient receives offer** → creates answer → sends back → P2P video call established
7. **ICE candidates** are exchanged via Socket.IO to finalize the P2P network path
8. **Audio recording** begins — `MediaRecorder` captures doctor's audio every 1 second and emits it as `audioData`
9. **Python backend** (`secondary_model.py`) receives the audio, transcribes it via Google Speech-to-Text
10. **`primary_model.py`** scans the transcribed text against `Disease.csv` and `Drugs.csv`, extracts matches, and generates a prescription
11. **Doctor clicks End Call** → streams stop → `endCall` event emitted with prescription
12. **Patient's browser** receives the `endCall` event and displays the prescription in a modal

---

## Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) v14+
- [Python](https://www.python.org/) 3.8+
- [ngrok](https://ngrok.com/) account + CLI
- A microphone and webcam

### Python Dependencies

```bash
pip install pyaudio speechrecognition python-socketio pandas
```

### Node.js Dependencies

```bash
cd MediScribe
npm install
```

---

## Installation & Setup

### 1. Start ngrok

```bash
ngrok http 3000
```

Copy the generated HTTPS URL (e.g. `https://xxxx-xxxx.ngrok-free.app`)

### 2. Update the ngrok URL

Replace the ngrok URL in all three files with your current ngrok URL:

- `server.js` — in the CORS `origin` field
- `Signalwebrtcint.js` — in `io.connect(...)`
- `secondary_model.py` — in `sio.connect(...)`

### 3. Start the Node.js Server

```bash
node server.js
```

Server will run on `http://localhost:3000`

### 4. Start the Python AI Backend

```bash
python secondary_model.py
```

---

## Running the App

| Role | URL |
|---|---|
| Landing Page | `https://<ngrok-url>/mediscribe` |
| Doctor | `https://<ngrok-url>/doc` |
| Patient | `https://<ngrok-url>/pat` |

1. Open the **Doctor** page in one browser tab/window
2. Open the **Patient** page in another tab, window, or device
3. Doctor clicks **Start Call** — WebRTC offer is sent to the patient
4. Patient automatically joins the call
5. Doctor speaks — the system transcribes and processes the speech
6. Doctor clicks **End Call** — prescription is auto-generated and shown to the patient

---

## Future Improvements

- 🧠 Replace keyword matching with a proper medical NLP model (spaCy, BioBERT)
- 🔐 Add JWT-based authentication for doctors and patients
- 🗄️ Store consultation history and prescriptions in a database
- 🌐 Replace ngrok with a proper cloud deployment (AWS / GCP / Azure)
- 📱 Build a mobile-friendly UI / React Native app
- 🔊 Handle audio format conversion (WebM/Opus → WAV) for more accurate STT
- 🏥 Add TURN server support for users behind strict NATs/firewalls
