# Robot Build Log

Append-only progress tracking. Agents and humans write here freely from any device.

---

## Feb 16, 2026

### Wake Word Switch
- [x] Switched from custom "Hey Wall-E" to built-in "Hey Jarvis" (hey_jarvis_v0.1)
- Reason: More reliable detection out of the box

### Phase 3 - Kid Features (reworked)
- [x] Reorganized 19 features by interaction type:
  - **Interactive (5):** animal_quiz*, counting*, magic_word*, story_time*, rhythm_game* ← uses Vosk voice recognition
  - **Sound (4):** silly_voice, simon_says, knock_knock, lullaby
  - **LED (4):** heartbeat, lightning, dance_led, goodnight_fade
  - **Combined (3):** demon_hunter (K-pop theme), dance_party, countdown
  - **Camera (3):** peekaboo, smile_detect, copycat
- *Interactive features use Vosk to listen for kid responses

---

## Feb 15, 2026

### Phase 3 - Kid Features ✓
- [x] bot-00 paired as OpenClaw node to alex-gtx orchestrator
- [x] 20 kid-friendly features built in /home/alex/robot/features/:
  - **Sound (6):** animal_quiz, counting, silly_voice, simon_says, knock_knock, lullaby
  - **LED (5):** heartbeat, lightning, dance_led, color_quest, goodnight_fade
  - **Combined (3):** demon_hunter (K-pop theme), dance_party, countdown
  - **Camera (3):** peekaboo, smile_detect, copycat
  - **Games (3):** treasure_hunt, magic_word, story_time
- [x] Robot skill updated with 20 new @robot actions
- [x] Exec-approvals configured on alex-gtx and bot-00
- [x] **Note:** OpenClaw `nodes run` / `system.run` is macOS-only (requires companion app). Use SSH for bot-00 commands.

### Phase 2 progress
- [x] Mic startup health check (kills stale arecord, retries 3x)
- [x] Trained "Hey Wally" wake word model (hey_walle.onnx)
- [x] Hybrid STT pipeline: Silero VAD → Remote Whisper → Vosk fallback
- [x] Whisper hallucination filter (blocks "thanks for watching" etc.)

### Notes
- Vosk replaced Whisper for STT — Whisper hallucinated YouTube phrases ("thanks for watching") on silence; Vosk returns empty string
- Vosk model preloaded at startup to avoid per-request loading latency
- Mic health check at wakeword startup — kills stale arecord processes that can block the mic device

---

## Feb 14, 2026

### Phase 1 - Brain ✓
- [x] Install OpenClaw, configure Claude API
- [x] OpenClaw Gateway on bot-00 (ws://bot-00.local:18789)
- [x] eSpeak installed (TTS working)
- [x] ~~Whisper installed~~ → Replaced with Vosk (no hallucinations)
- [x] Voice → OpenClaw integration: wakeword.py listens always, routes to OpenClaw, displays on TFT

### Phase 2 - Senses / Intelligence
- [x] Camera connected (Sunplus Full HD at /dev/video0)
- [x] Motion detection → wave greeting
- [x] Rock Paper Scissors game
- [x] Voice control with wake word ("Hey Wall-E")
- [x] Always-listening wake word via OpenWakeWord ("Hey Wall-E" custom trained)
- [x] VAD-based recording (Silero VAD — stops when you stop talking)
- [x] ~~Remote Whisper fallback~~ → Removed, Vosk runs locally
- [x] Cancel phrase detection ("never mind", "stop", etc.)
- [x] Silence detection via RMS threshold (skips transcription if too quiet)
- [x] Post-response cooldown (4s before listening again)
- [x] Fix STT hallucination on silence → Switched to Vosk (returns "" on silence, skips OpenClaw)
- [x] Train custom "Hey Wall-E" wake word (trained on alex-gtx RTX 4080)
- [x] alex-gtx firewall open for Whisper (port 8787)
- [x] Fix alex-gtx CUDA in WSL2 (added /usr/local/lib/ollama/cuda_v12 to ldconfig)
- [x] TFT display showing animated face
- [x] Ollama installed on alex-gtx (v0.16.1)
- [x] ~~Whisper server on alex-gtx~~ → No longer needed, using local Vosk
- [x] Voice → OpenClaw routing (unknown commands go to Claude, response on TFT)
- [x] Text display on TFT face (shows responses, auto-clears after 8s)
- [x] OpenAI-compatible chat completions API enabled on Gateway
- [x] OpenClaw "robot" skill for triggering actions (rps, wave, emotions)
- [x] LED feedback: PULSE (async triple-blink during recording)

### Notes
- Face service: pygame.mixer.init() wrapped in try/except to handle audio device not ready at boot
- Pi 5 runs hotter than Pi 4 — may need heatsink/fan for sustained operation

---

## Feb 13, 2026

### Phase 1 - Brain
- [x] Purchase Pi 5, Arduino, peripherals
- [x] Flash Pi 5 with Raspberry Pi OS (bot-00 at 10.0.0.56)
- [x] Arduino serial handshake (PING/PONG)

---

## Backlog

### Phase 2 - Remaining
- [ ] Vision queries to Claude working
- [ ] Pull Llama 3 8B/70B model (in progress)
- [ ] Route simple commands to local LLM
- [ ] Achieve <2 second response times

### Chassis - Physical Build
- [ ] Acquire chassis, motors, arm
- [ ] Wire motors to L298N to Arduino
- [ ] Wire servos to PCA9685 to Arduino
- [ ] Implement movement commands in Arduino sketch
- [ ] Test voice → movement end-to-end

### Fleet
- [ ] Design modular base platform
- [ ] Build Unit 2 (lawn mower brain)
- [ ] Shared memory/coordination between units

---

## Technical Notes (accumulated learnings)

- Face service: pygame.mixer.init() wrapped in try/except to handle audio device not ready at boot
- Pi 5 runs hotter than Pi 4 — may need heatsink/fan for sustained operation
- Vosk replaced Whisper for STT — Whisper hallucinated YouTube phrases ("thanks for watching") on silence; Vosk returns empty string
- Vosk model preloaded at startup to avoid per-request loading latency
- Mic health check at wakeword startup — kills stale arecord processes that can block the mic device
