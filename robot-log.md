# Robot Build Log

Append-only progress tracking. Agents and humans write here freely from any device.

---

## Jul 10, 2026

### bot-00 revived + audio stack fixed (project pulled to ~/Desktop/robot-fleet)
- [x] bot-00 IP drifted .54 → **10.0.0.36** (DHCP). Use `bot-00.local`. `robot.md` updated.
- [x] **Root cause of dead wakeword:** Pi was re-flashed to a **PipeWire**-based Raspberry Pi OS. PipeWire monopolized the Blue mic (the only audio device — mic *and* speaker on card 1). Robot code uses raw ALSA (`arecord -D plughw:1,0`) → every capture hit "device busy" → `wakeword.py` read EOF, hit a silent `break`, exited 0; `Restart=on-failure` never restarted it. A duplicate **user** `robot-wakeword` (`Restart=always`) respawned it endlessly under the broken mic = the "orphan" churn.
- [x] **Fix — made bot-00 ALSA-only:**
  - Masked user PipeWire trio (`pipewire`, `pipewire-pulse`, `wireplumber`).
  - Added `/etc/asound.conf` → ALSA `default` = `hw:1,0` (Blue card).
  - `core.py speak()`: `espeak` (dead JACK backend) → `espeak --stdout | aplay -D plughw:1,0`. TTS verified OK + audible.
  - `wakeword.py`: silent EOF `break` → `sys.exit(1)`.
  - Disabled duplicate **system** `robot-wakeword`; kept **user** one (`Restart=always`).
  - `loginctl enable-linger alex` so user services start at boot. All persisted.
- [x] Result: `robot-core` + `robot-wakeword` (user) stable, wakeword listening (NRestarts=0), TTS + mic + camera + face all working.
- [x] **Camera services cleaned up:** both broken-by-design now that `core.py` owns `/dev/video0` exclusively. `robot-health` = one-shot boot check doing direct `cv2.VideoCapture(0)` (always fails, wrong python); `robot-motion` = service pointing at a **missing** `motion.py` (only `motion.py.old` left) → crash-looped on file-not-found. **Both stopped + disabled**, failed state cleared. bot-00 all-green (face + core + wakeword active; nothing failed).
- [x] **Presence greeter shipped:** replaced the old motion-diff `MotionDetector` in `core.py` with `PresenceGreeter` — `detect_faces`-triggered, visual wave + LED pulse, 5s cooldown, arrival-only, idle-gated (quiet during games via `face.last_activity`). Ambient "robot notices you" for the 3 & 5yo. `--no-greet` to disable.
- [x] **Rock Paper Scissors v1 shipped:** rewrote `features/rock_paper_scissors.py` visual-first (audio deprioritized) — 3-2-1-GO countdown + LED, throw revealed as the face RPS graphic. No input capture (kids self-judge); v2 = capture/auto-score. Voice triggers already mapped.
- [x] **Greeter face-detection tuned:** loosened `CameraManager.detect_faces` from `scaleFactor=1.3, minNeighbors=5` → `1.1/3` so kids who get right up in the lens (large/tilted/cropped faces) are detected. Verified against a live capture where 1.3/5 found 0 faces and 1.1/3 found 2.
- [x] **Wakeword troubleshooting:** (a) service had been left stopped → restarted, stable (NRestarts=0, holds mic, listening). (b) Command accuracy is degraded because the **Whisper STT server on alex-gtx (10.0.0.40:8787) is unreachable — that box is powered off** → falls back to small Vosk which mishears. Mitigated: `parse_command` now does exact→keyword(rock/paper/scissors, token-matched)→conservative fuzzy(≥0.7) matching (unit-tested 10/10, e.g. "rocket"→countdown preserved, garbage→no launch). Durable fix = power on alex-gtx OR run local Whisper on the Pi (recommended).
- [ ] **Still parked for strategy pass:** rest of games library (smile_detect typo, stubbed rhythm_game/simon_says/treasure_hunt, no error handling).

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
