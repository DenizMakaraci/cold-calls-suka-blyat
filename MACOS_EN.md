# Liniya for macOS — English edition

A local voice support assistant for Apple Silicon, including MacBook Pro M5 Pro.
Talk to it in your browser, test answers in chat, or replace the demo knowledge
with your own TXT/Markdown documents. This is a browser conversation app;
telephone numbers, SIP, incoming phone calls and call recording are not included.

## Start

1. Extract **Liniya-macOS-EN.zip** into a writable folder such as Downloads or Applications in your home folder. Do not run it inside the ZIP.
2. Open **START_MACOS.command**. The first launch downloads verified native tools, Python packages and roughly 4.8 GB of model weights, then builds the interface. Allow several minutes and at least 10 GB of free disk space.
3. Your default browser opens **http://127.0.0.1:3000** after all services are ready. Choose **Voice conversation → Start voice conversation** and allow microphone access.
4. Open **STOP_MACOS.command** when finished. Closing the browser or Terminal window does not stop background services.

If Finder blocks a downloaded command, use macOS **Open Anyway** in Privacy & Security only after verifying the source. Alternatively, open Terminal, type `cd `, drag the extracted folder into Terminal, press Enter, then run:

```bash
bash scripts/macos.sh start
```

No administrator password, Homebrew, Docker, Rosetta or paid API key is needed.
The installer keeps Node, uv and Python inside the app folder. Internet is needed
for installation; default conversations run locally after setup.
Do not move the folder after installation: Python environments use absolute paths.
To relocate, stop it, move it, delete only `voice/.venv` and `.local/installed`,
then start again. Keep `.env.local` and the rest of `.local`.

## English and Russian versions

Both editions have independent interfaces, demo documents, assistant prompts,
speech recognition language, browser storage keys, server credentials and
temporary knowledge storage. The English edition speaks English. The Russian
edition speaks Russian. Stop one edition before starting the other: local
service ports are shared, and the launcher reports conflicts instead of
connecting to another installation.

## M5 Pro implementation

- Native ARM64 llama.cpp uses Metal and offloads Qwen2.5 7B Q4_K_M to the Apple GPU. This model fits the M5 Pro with 24 GB of unified memory and improves answer quality over the original 1.5B model. Check answers against your business documents; citations do not guarantee factual accuracy.
- Multilingual faster-whisper base uses native ARM64 CPU int8 inference with four threads. It does **not** use CUDA, the Neural Engine or Metal.
- LiveKit provides local WebRTC audio, turn detection and interruptions. Silero handles voice activity detection.
- Local speech uses the installed macOS **Samantha** voice. Set `MACOS_VOICE` in `.env.local` to another installed English voice if desired. `say -v '?'` lists voices. Install missing voices in **System Settings → Accessibility → Read & Speak** (called Spoken Content on older macOS).
- macOS 15+, Apple Silicon and at least 16 GB of unified memory are required (24 GB recommended). The M5 Pro target has been chosen explicitly; there is no x86 emulation.

For microphone access, check both browser site permissions and **System Settings
→ Privacy & Security → Microphone**. Select the MacBook microphone in your
browser and macOS Sound settings. Use headphones to reduce feedback. Safari and
Chromium browsers can prompt you to enable answer audio; press the audio button
if it appears. A physical microphone conversation is still needed to assess your
room acoustics, timing and interruptions.

## Knowledge and privacy

The 18 TechnoLine FAQ entries are fictional examples, not instructions for real
equipment. Use **Knowledge base → Upload file** to replace them with `.txt` or
`.md`; Markdown sections start with `## Question`, followed by the answer.
Limits: 100 entries, 200 characters per title, 4,000 per answer, 180,000 total.
Importing replaces the current knowledge and clears the chat. Export `.md` to
back up your knowledge. Text chat and knowledge persist in this browser under
the edition-specific storage key. Voice transcripts are temporary; audio is
not recorded by this app. Voice session knowledge is kept in local SQLite with
a one-hour TTL.

Services bind to loopback by default. This is a single-user local MVP, not a
public or multi-tenant service. If you enable cloud Boson or an external model,
the configured provider receives the relevant text.

## Optional Boson voice

To use **Boson for both reasoning and speech**, set `LLM_PROVIDER=boson` in
`.env.local` alongside your `BOSON_API_KEY` and `BOSON_MODEL=higgs-tts-3`, then
stop and restart this edition. The interface shows **Boson reasoning**.
Higgs Realtime generates answers over a server-side WebSocket; Higgs TTS 3
speaks them. Qwen is no longer started or downloaded in this mode. Existing
weights are retained for switching back. Internet and Boson account credit are
required. Set `LLM_PROVIDER=local` to return to local reasoning.

Boson receives the question, recent conversation history and company knowledge
(up to 20,000 characters of complete source records per turn). Small knowledge
bases are sent in full, so paraphrases are not rejected by a word-overlap rule.
Larger bases are ranked and bounded. Answers still need valid citations for
company facts; missing facts should produce a specific explanation or clarification.
Microphone capture, Whisper recognition and LiveKit remain local.

To enable **general knowledge and live public web search**, also set
`BOSON_WEB_SEARCH=1` and restart. The interface shows **Boson + Web**.
Boson can answer general questions without a matching local FAQ and use its
built-in web search for current information. Web links appear in chat and
under voice answers; they must occur in actual search results before the app
accepts them. Answers without a fresh lookup are labelled **General knowledge**.
Company document citations remain separate. This does not provide access to
private accounts or paywalled databases. Search queries are processed by
Boson and its search service and can incur additional API charges. Use
`BOSON_WEB_SEARCH=0` to return to company-knowledge mode.

Copy `.env.example` to `.env.local`, add `BOSON_API_KEY` there, and restart.
Never put the key into browser code. The default local macOS voice works without
an account. Boson is optional; its live API and account limits require a separate
real-provider check. Automated local checks refuse to make paid Boson calls.

## Troubleshooting and verification

```bash
bash scripts/macos.sh status
bash scripts/macos.sh stop
bash scripts/macos.sh setup
voice/.venv/bin/python scripts/macos-check.py
```

The last command requires running services and performs real local speech
synthesis → Whisper → retrieval → Qwen → spoken answer checks for this edition.
It also checks that unknown device models are rejected. It is not a physical
microphone or browser WebRTC test.

Logs are in `.local/logs`; model revisions and checksums are in
`voice/model-lock.json` and `.local/models/manifest.json`. Native downloads are
pinned and checksum-verified; Python and npm dependencies are locked. If setup
is interrupted, rerun it. An abandoned `.local/launcher.lock` can be removed
after confirming no installation is running.

If a port is busy, stop the other edition or the app using that port. The app
uses 3000, 7880, 7881, 8001, 8081, 11434 and UDP 50000–50100. Sleep or network
changes can interrupt a call: end it and start again after waking the Mac.

## Development

The parent source project contains both editions. Run
`python3 scripts/package-macos.py` to regenerate `dist/Liniya-macOS-RU.zip` and
`dist/Liniya-macOS-EN.zip`. `locales/en.json` is the English build-time catalog.
The packager translates source text once, preserves interpolation, and rejects
untranslated Cyrillic in the UI and demo knowledge. `edition.json` selects
recognition, retrieval, HTML language and storage isolation. Do not edit
generated editions as the source of truth.
