Global Tech Stack & Repo Architecture
Use this exact stack and directory structure across all versions so the coding agent never invents conflicting files:

Framework & Styling: Vite + React + TypeScript + Tailwind CSS v4 (@tailwindcss/vite plugin, @import "tailwindcss"; in src/index.css — no tailwind.config.js or PostCSS config) + lucide-react

Audio & Visuals: Native Web Audio API (AudioContext, OfflineAudioContext, AnalyserNode, WaveShaperNode) + HTML5 2D Canvas

Local Agent Server: Custom Vite Dev Server middleware (server/agentBridge.ts) running on http://localhost:4400 using HTTP REST + Server-Sent Events (SSE) for zero-dependency real-time browser streaming (plus an automatic static fallback for hosted hackathon deployment)

Plaintext
pulsedeck/
├── PLAN.md
├── vite.config.ts               # Port 4400 + mounts local Agent Bridge middleware
├── server/
│   └── agentBridge.ts           # GET /agent, POST /agent/action, GET /agent/stream (SSE)
└── src/
    ├── types/
    │   └── studio.ts            # SongProject, Track, Step, Section, SonicDNA, AgentAction
    ├── music/
    │   ├── scales.ts            # Phrygian, Minor, Dorian, Pentatonic frequency tables
    │   ├── groove.ts            # Euclidean rhythm generator + MPC swing math
    │   └── presets.ts           # 3 studio-grade starter tracks (Phonk, Lo-Fi, Synthwave)
    ├── audio/
    │   ├── AudioEngine.ts       # Lookahead 16th-note scheduler + Sidechain + Master Tape Bus
    │   ├── voices/
    │   │   ├── drums.ts         # Kick, multi-pulse Clap/Snare, velocity Hi-Hats
    │   │   ├── titan808.ts      # Click + pitch drop + portamento glide + tanh distortion
    │   │   ├── metallicHook.ts  # 1:1.4815 TR-808 cowbell & FM lead synth
    │   │   └── atmosPad.ts      # Detuned polyphonic chords + low-pass filter
    │   └── wavExporter.ts       # OfflineAudioContext .WAV encoder + Remotion beats.json
    ├── dna/
    │   └── sonicDna.ts          # Taste scoring, localStorage persistence, persona morphing
    ├── agent/
    │   ├── actionDispatcher.ts  # Executes AgentActions with Ghost Producer animation queue
    │   └── demoAgent.ts         # Built-in autonomous demo script for hosted mode / judges
    ├── components/
    │   ├── TopBar.tsx           # Brand, Agent status badge, Sonic DNA meter, Export button
    │   ├── VisualStage.tsx      # 60fps Canvas visualizer + floating plain-English agent pills
    │   ├── GhostCursor.tsx      # Animated glowing cursor that moves to knobs/pads on actions
    │   ├── TrackMixer.tsx       # 4-track visual pads (Mute, Solo, Keep DNA, Mutate)
    │   ├── MacroRack.tsx        # 4 rotary hardware knobs (auto-labeled per Sonic DNA persona)
    │   └── ArrangementBar.tsx   # Multi-section blocks (Intro → Drop → Break → Outro)
    └── App.tsx
Version 0.1 — Project Scaffold, Music Theory Core & Studio Shell
Objective: Initialize the project cleanly on port 4400, define the single source of truth for all TypeScript data models, and build the music theory utilities so neither humans nor agents can ever generate invalid or out-of-key beats.

Tasks for the Agent
Scaffold Vite (react-ts), install tailwindcss, @tailwindcss/vite, and lucide-react, and configure vite.config.ts to run on port: 4400 with strictPort: true.

Create src/types/studio.ts defining:

ScaleName: 'D_phrygian' | 'F_minor' | 'A_dorian' | 'C_minor_pentatonic'

PersonaId: 'phonk' | 'lofi' | 'synthwave'

TrackId: 'drums' | 'bass808' | 'hook' | 'pad'

StepEvent: { step: number; active: boolean; degree?: number; velocity: number; glide?: boolean; accent?: boolean }

SongSection: { id: string; name: string; bars: number; activeTracks: TrackId[]; filterTargetHz: number; driveTarget: number }

SongProject: { title: string; bpm: number; scale: ScaleName; swing: number; sidechain: number; drive: number; filterHz: number; tracks: Record<TrackId, StepEvent[]>; sections: SongSection[] }

SonicDNA: weighted scores for phonk, lofi, and synthwave (0 to 100), favorite BPM range, preferred scale, and locked pattern memory.

AgentAction: discriminated union for set_stage, sculpt_drums, lay_bassline, add_hook, add_pad, set_macros, arrange_timeline, transport, and narrate.

Create src/music/scales.ts mapping scale degrees (including negative octaves for sub-bass) to exact Hz frequencies.

Create src/music/groove.ts implementing a Bjorklund Euclidean rhythm generator (generateEuclidean(pulses, steps)) and an MPC swing offset calculator (getSwingOffsetSec(stepIndex, bpm, swingAmount)).

Create src/music/presets.ts containing 3 complete, genuinely musical 16-step multi-section starter tracks (Dark 134 BPM Phonk, 86 BPM Dusty Lo-Fi, 120 BPM Cyber Synthwave).

Render a clean dark placeholder hardware layout in src/App.tsx displaying the loaded preset metadata.

Completion Goal (v0.1 Definition of Done)
Running npm run dev opens PulseDeck at http://localhost:4400 with zero TypeScript or console errors.

Tailwind v4 styles render properly.

Switching between the 3 starter presets updates the displayed BPM, scale, and step arrays accurately, and unit-checking scales.ts confirms all scale degrees resolve to valid frequencies.

Version 0.2 — High-Punch Web Audio DSP Engine & Lookahead Sequencer
Objective: Build the zero-dependency procedural audio engine so the 3 starter tracks sound punchy, warm, and studio-mixed before designing the heavy visuals.

Tasks for the Agent
Build src/audio/voices/drums.ts:

Kick: Tight sine pitch drop (150Hz to 42Hz over 45ms) layered with a 12ms high-passed click transient.

Clap / Snare: 3 staggered micro-bursts of bandpass-filtered noise spaced 14ms apart over a 185Hz punch body.

Hi-Hat: High-pass filtered noise (8500Hz) with velocity-sensitive decay (30ms closed, 160ms open) and slight pitch randomization for realism.

Build src/audio/voices/titan808.ts:

Layer a 15ms 1.4kHz attack click + triangle/sine sub body with exponential pitch drop.

Implement true portamento pitch glide when step.glide is true.

Route through a custom asymmetric tanh WaveShaperNode (controlled by the drive macro) followed by an 850Hz low-pass filter to keep sub-bass aggressive yet clean.

Build src/audio/voices/metallicHook.ts:

Implement the classic dual-square metallic cowbell/pluck using the exact 1 : 1.4815 frequency ratio routed through a resonant bandpass filter (Q = 4.5) with a snappy decay envelope (plus a softer FM bell/EP mode for Lo-Fi and sawtooth lead mode for Synthwave).

Build src/audio/voices/atmosPad.ts:

3-voice chord synthesizer pulling triad/7th intervals directly from scales.ts, routed through a gentle low-pass filter and stereo delay/chorus feel.

Build src/audio/AudioEngine.ts:

Implement a rock-solid Web Audio Lookahead Scheduler (25ms timer scheduling notes 100ms ahead using audioCtx.currentTime) so timing never stutters even during heavy UI rendering.

Build the Automated Sidechain Ducking Bus: route bass808, hook, and pad through sidechainGain; every time a Kick triggers at time t, automate sidechainGain down to 1 - (sidechain * 0.85) in 4ms and recover smoothly over 160ms.

Build the Master Tape Bus: WaveShaperNode warm soft-clipper → BiquadFilterNode (master DJ sweep) → DynamicsCompressorNode (limiter) → AnalyserNode (for live visuals) → destination.

Support multi-section playback (automatically advancing through Intro → Drop → Break → Outro, muting inactive tracks and ramping filter/drive targets per section).

Completion Goal (v0.2 Definition of Done)
Clicking Play in the browser plays all 3 presets with zero audio crackling, zero timing drift, and clearly audible kick-driven sidechain pump.

Adjusting BPM, Swing, 808 Drive, Sidechain, and Master Filter changes the live sound smoothly without restarting playback.

The sequencer automatically transitions across song sections (Intro to Drop to Break) with audible filter sweeps and track mutes.

Version 0.3 — Tactile Visual Studio UI & 60fps Audio-Reactive Stage
Objective: Replace all developer test controls with the polished, non-codey hardware deck and 60fps beat-reactive center canvas so any non-technical user or judge immediately understands what is happening.

Tasks for the Agent
Build src/components/VisualStage.tsx:

Full-width HTML5 2D Canvas reading live FFT frequency data and bass energy from AudioEngine.getAnalyserData() at 60fps via requestAnimationFrame.

Render a crisp, hardware-grade beat visualizer (bass shockwave rings + spectrum bars + live active beat step indicator) that reacts violently to kicks and 808 glides.

Overlay a sleek, non-intrusive Live Action Banner Pill in the top-center of the canvas (e.g., "Grooving at 134 BPM • D Phrygian • Section: MAIN DROP").

Build src/components/TrackMixer.tsx:

4 side-by-side tactile instrument cards (01 KICK & PERC, 02 TITAN 808, 03 LEAD HOOK, 04 ATMOS PAD).

Each card shows a visual 16-step illuminated pad strip (users can click/drag to toggle hits, right-click or Shift-click to toggle 808 glide), Mute/Solo LED buttons, a 🎲 Mutate button (generates a fresh scale-locked musical variation for just that track), and a 🔥 Keep button (signals taste preference).

Build src/components/MacroRack.tsx:

4 custom SVG/Canvas Rotary Hardware Knobs with tick marks, indicator LEDs, and vertical click-and-drag interaction (plus double-click to reset).

Wire knobs directly to AudioEngine macros (808 Drive, Sidechain Pump, Groove Swing, DJ Filter) with smooth visual needle animation.

Build src/components/ArrangementBar.tsx:

Horizontal clickable timeline blocks (01 INTRO (4B) → 02 MAIN DROP (8B) → 03 BREAK (4B) → 04 OUTRO (4B)) with a live progress fill sweeping across the active block and one-click section jumping.

Build src/components/TopBar.tsx:

Studio logo, Play/Pause transport, BPM tap/scrub readout, Scale selector, and quick Preset loader.

Completion Goal (v0.3 Definition of Done)
Zero raw JSON or code appears anywhere on the screen.

Users can build, mutate, mute, and tweak a full beat entirely by clicking pads, dragging rotary knobs, and jumping between arrangement blocks.

The center canvas runs at a smooth 60fps and visibly pulses in sync with every kick and melody note.

Version 0.4 — Universal Local Agent Bridge & Visual Ghost Producer
Objective: Enable any AI agent in any harness (OpenCode CLI, Claude Code, Cursor, Python, curl) to connect to http://localhost:4400/agent, discover the studio, and build beats step-by-step while the user watches the Ghost Producer operate the UI live.

Tasks for the Agent
Build server/agentBridge.ts and mount it as a Vite plugin inside vite.config.ts:

GET /agent: Returns a clean, agent-friendly JSON handbook containing:

Current SongProject summary (BPM, scale, active sections)

User's current SonicDNA profile (with plain-English guidance on what genre/vibe the user loves)

Exact schema and examples for POST /agent/action commands

A short quickStartPrompt so any LLM immediately knows how to compose a multi-step beat.

GET /agent/stream: Opens a Server-Sent Events (SSE) channel pushing incoming agent actions to the browser in real time (text/event-stream).

POST /agent/action: Validates an incoming single AgentAction (or batch of actions), updates server-side state, broadcasts it over SSE to the browser, and returns { ok: true, applied: action.type }.

POST /agent/dna: Allows the browser to sync updated SonicDNA back to the local bridge so newly connected agents always read the latest user taste.

Enable permissive local CORS headers so external local scripts never hit browser blocks.

Build src/agent/actionDispatcher.ts and src/components/GhostCursor.tsx:

Maintain an animated visual queue so when an agent sends 5 rapid actions, the browser spaces them out smoothly (~900ms apart) so the user can actually watch the beat come together.

Animate a glowing Ghost Producer Cursor (data-agent-target="knob-drive", data-agent-target="track-bass808", etc.) gliding to the exact pad, knob, or section block being edited.

Display plain-English toast pills on the visual stage ("🤖 Agent connected • Locking D Phrygian at 134 BPM", "🔊 Sculpting gliding 808 bassline", "🎬 Arranging 16-bar drop timeline").

Smoothly animate rotary knobs turning to their target values instead of snapping instantly.

Build src/agent/demoAgent.ts and add a ⚡ Run Demo Agent button + 🔌 Connect Agent modal in TopBar.tsx:

Clicking ⚡ Run Demo Agent runs a choreographed 6-step autonomous producer session directly inside the browser (using the exact same actionDispatcher + GhostCursor), ensuring the hosted hackathon version works identically even without a local terminal.

Clicking 🔌 Connect Agent opens a clean 1-click copy card showing the exact prompt to paste into any agent harness ("Connect to PulseDeck at http://localhost:4400/agent and build a beat matching my Sonic DNA").

Completion Goal (v0.4 Definition of Done)
Running curl http://localhost:4400/agent in a terminal returns the live studio state, Sonic DNA, and action spec.

Sending a POST http://localhost:4400/agent/action request from a terminal (or real AI agent harness) visibly moves the Ghost Cursor on the open webpage, pops up a plain-English status pill, updates the pads/knobs, and changes the live playing music within 1 second.

Clicking ⚡ Run Demo Agent in the browser builds a complete beat step-by-step visually with zero backend dependency.

Version 0.5 — Adaptive "Sonic DNA" Engine (Auto-Morphing Studio Personas)
Objective: Make PulseDeck genuinely adapt to the user over time—morphing its physical hardware skin, visualizer style, macro knob labels, audio engine character, and agent defaults to match the music they love.

Tasks for the Agent
Build src/dna/sonicDna.ts:

Track and persist (localStorage + POST /agent/dna) behavioral signals:

BPM Gravity: Moving average of tempos played for more than 10 seconds.

Macro Bias: Tracks whether the user favors heavy Drive/Sidechain (Phonk/Bass), high Swing/Warm Filter (Lo-Fi), or wide Pads/Medium Tempo (Synthwave).

Explicit Training (🔥 Keep button): Clicking 🔥 Keep on any track boosts the active style's DNA score (+12 points), flashes a subtle DNA pulse animation, and saves that pattern as a reference motif for the agent.

Calculate dominant persona (phonk, lofi, or synthwave) plus percentage confidence (e.g., 🔥 Phonk / Bass DNA • 84%).

Implement 3 full Dynamic Hardware Skins via CSS custom properties (data-persona="phonk" | "lofi" | "synthwave" on the root container) with smooth 500ms transitions:

phonk (Redline Carbon Deck): Matte obsidian #0C0D10, international orange #FF4500 & crimson LEDs, aggressive radial bass shockwave visualizer, macro labels tuned to 808 DRIVE / PUMP / GRIT / CUT.

lofi (Warm Sampler Deck): Brushed cream #EDE8DF & anodized aluminum #2A2825, amber #E89B2D VU needles, circular vinyl/tape oscilloscope visualizer, macro labels tuned to TAPE WARMTH / DUCK / MPC SWING / LOW-PASS (plus subtle vinyl crackle layer enabled in AudioEngine).

synthwave (Cyber-Analog Deck): Deep midnight #090B1A, cyan #00E5FF & magenta phosphor CRT perspective grid visualizer, macro labels tuned to ANALOG DRIVE / PULSE / GROOVE / RESONANCE.

Add a DNA Status Pill in TopBar.tsx that shows live adaptation (and lets users/judges preview or reset their Sonic DNA in one click so hackathon judges can test all 3 studio transformations immediately).

Feed the live SonicDNA summary directly into GET /agent so any connecting AI agent automatically adopts the user's preferred BPM, scale, and vibe without being told.

Completion Goal (v0.5 Definition of Done)
Clicking 🔥 Keep on Phonk tracks or cranking BPM above 128 + Drive above 65% shifts the Sonic DNA meter toward Phonk and smoothly morphs the entire studio into the Redline Carbon skin and shockwave visualizer.

Lowering BPM to 85 and dialing in Swing morphs the studio into the Warm Sampler skin with circular tape visuals and audible vinyl warmth.

Hitting GET http://localhost:4400/agent reflects the updated Sonic DNA profile immediately.

Version 1.0 — Master .WAV Export, Remotion beats.json Sync & Hackathon Polish
Objective: Deliver real creator utility (exporting mastered instrumentals + video-sync data), lock down zero-friction availability, and polish every micro-interaction for hackathon judging.

Tasks for the Agent
Build src/audio/wavExporter.ts:

Instantiate an OfflineAudioContext matching the exact full length of song.sections (in seconds at 44100Hz stereo).

Reuse the exact same voice synthesizers, sidechain ducking bus, and section filter/drive automation from AudioEngine.ts to render the entire multi-section song faster than realtime.

Encode the resulting AudioBuffer into a standard 16-bit PCM .WAV Blob and trigger download ({title}.wav).

Simultaneously generate and export {title}_beats.json containing:

bpm, fps: 60, durationSec

kicks: exact { timeSec, frame60, velocity }[]

snares: exact { timeSec, frame60 }[]

drops: exact { section, timeSec, frame60 }[] (ready for Remotion camera cuts and text pops).

Add URL State Sharing:

Add a Share Link button that compresses the current SongProject into a URL hash (#s=...) so opening the link loads that exact beat and Sonic DNA persona instantly.

Add Subtle Tactile UI Micro-Audio:

Synthesize ultra-short 8ms mechanical switch clicks when toggling pads, clicking Mute/Solo, or switching sections (with a clean mute toggle).

Hosted Mode & Hackathon Bulletproofing:

Gracefully detect when running as a static hosted site (e.g., Vercel/Netlify for the Freebuff hackathon link) vs. localhost:4400: hide connection errors if /agent/stream is unavailable on static hosting, and highlight ⚡ Run Demo Agent + 🔌 Run Locally for Terminal Agents cleanly.

Add keyboard shortcuts (Space to Play/Pause, 1–4 to mute tracks, M to mutate).

Verify npm run build succeeds with zero TypeScript warnings and zero runtime console errors in an incognito browser window.

Completion Goal (v1.0 Definition of Done)
Clicking Export Master (.WAV + Remotion Sync) renders the full multi-section track in less than 3 seconds and downloads both a crisp, distortion-free .WAV file and a valid beats.json timestamp file.

Opening a shared URL in an incognito tab loads the exact beat immediately with zero login walls.

npm run build passes cleanly, the hosted static preview works flawlessly via the built-in Demo Agent, and running npm run dev locally lets any external AI agent connect and produce beats live on screen.
