Robot Fleet Project - AI-powered robots with the kids

# Robot Fleet Project

Building AI-powered robots with the kids. Started Feb 2026.

## Helpful Shortcuts

```bash
# Sync this file to GitHub Gist
alias robot-sync="gh gist edit 569220949b12e7f147386b1f708d1e52 -f ~/robot/robot.md"
```

## Vision

A fleet of modular robots around the house, each with the same core architecture: Pi brain running OpenClaw (Claude-powered agent), Arduino body for real-time hardware control. They share intelligence, learn from the family, and coordinate tasks.

The kids don't program robots — they teach them.

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Home Network                            │
│                                                                 │
│  ┌─────────────────┐          ┌─────────────────────────────┐  │
│  │   Unit 1        │          │       alex-gtx              │  │
│  │   (Robot)       │          │       10.0.0.40             │  │
│  │                 │          │                             │  │
│  │  ┌───────────┐  │          │  - RTX 4080 (16GB VRAM)     │  │
│  │  │  Pi 5     │◄─┼──LAN────►│  - Ollama (Llama 3 70B)     │  │
│  │  │  OpenClaw │  │          │  - (STT now local via Vosk) │  │
│  │  └─────┬─────┘  │          │  - Future: embeddings,      │  │
│  │        │USB     │          │    image gen, vision        │  │
│  │  ┌─────▼─────┐  │          └─────────────────────────────┘  │
│  │  │  Arduino  │  │                                           │
│  │  │  Mega2560 │  │          ┌─────────────────────────────┐  │
│  │  └─────┬─────┘  │          │       Pi 4 (existing)       │  │
│  │        │        │          │       10.0.0.53             │  │
│  │  Motors/Servos  │          │                             │  │
│  │  Sensors        │          │  - OpenClaw orchestrator    │  │
│  │  Camera         │          │  - Gateway WebSocket        │  │
│  │  Speaker/Mic    │          │  - Messaging hub            │  │
│  └─────────────────┘          └─────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Network Map

| Host | IP | Role | Hardware |
|------|----|------|----------|
| alex-gtx | 10.0.0.40 | **OpenClaw orchestrator**, Gateway + WhatsApp channel, local inference | Windows 11, RTX 4080 16GB, WSL2 Ubuntu |
| Pi 4 | 10.0.0.51 (eth) / 10.0.0.53 (wifi) | OpenClaw node (connects to alex-gtx) - NOT the robot | Raspberry Pi 4 |
| **bot-00** | `bot-00.local` → **10.0.0.54** (WiFi) | **WALL·E robot brain** + local OpenClaw Gateway | Pi 5 8GB + Arduino Mega 2560 |

**IMPORTANT:**
- **bot-00** = The robot (Pi 5) = `ssh alex@bot-00.local` or `ssh alex@10.0.0.54`
- **Pi 4** = Separate device, NOT the robot = `ssh alex@10.0.0.51` (eth) or `ssh alex@10.0.0.53` (wifi)
- Ethernet (.56) is currently down, use WiFi (.54) or mDNS (bot-00.local)
- **OpenClaw node limitation:** `system.run` via nodes is macOS-only. Use SSH for bot-00 commands.

## Unit 1 - The Prototype

**Purpose:** Wheeled companion robot with arm/claw. Voice-controlled. The kids' robot.

### Hardware (Purchased)

- Raspberry Pi 5 8GB
- Pi 5 27W USB-C PSU
- NVMe HAT for Pi 5
- NVMe SSD (TBD - need to acquire)
- Arduino Mega 2560
- 3.5" TFT display (face/eyes)
- Adafruit speaker
- Blue Microphone (USB, card 1)
- Breadboard + power supply module
- Jumper wires

### Hardware (To Acquire)

- 2WD/4WD chassis kit with DC motors
- L298N motor driver
- PCA9685 servo driver
- Robot arm/claw kit with servos (MG996R)
- USB webcam or Pi Camera
- Battery pack for mobile operation

### Software Stack

| Layer | Component | Purpose |
|-------|-----------|---------|
| AI Brain | OpenClaw + Claude API | Reasoning, planning, conversation, skill execution |
| Speech-to-Text | Vosk (local on Pi) | Voice input — no hallucinations on silence |
| Text-to-Speech | eSpeak | Voice output |
| Body Controller | Arduino sketch | Real-time motor/servo control |
| Communication | USB Serial | Pi ↔ Arduino command protocol |

### Serial Protocol (Pi → Arduino)

Simple text commands over USB serial at 9600 baud:

```
PING              → PONG
PULSE             → (no response) LED: 70ms on/off x3 - visual "listening" cue
ATTENTION         → OK (flash LED 3x fast)
WAVE              → OK (flash LED 3x slow)
FORWARD <ms>      → OK
BACK <ms>         → OK
LEFT <deg>        → OK
RIGHT <deg>       → OK
CLAW_OPEN         → OK
CLAW_CLOSE        → OK
ARM_UP            → OK
ARM_DOWN          → OK
```

Arduino acknowledges each command. WAKEUP returns OK immediately before LED pattern (non-blocking). Pi sends commands async via fire-and-forget threads.

## Future Units

### Home Maintenance

| Unit | Name | Purpose |
|------|------|---------|
| 3 | GutterScout | Gutter inspection, debris clearing |
| 4 | ShopKeeper | Workshop inventory, tool retrieval |
| 5 | PipeWatch | Plumbing leak/freeze monitoring |
| 6 | FilterMinder | HVAC filter monitoring |
| 7 | CrawlBot | Crawlspace/attic inspection |
| 8 | PanelGuard | Electrical panel monitoring |
| 9 | SumpSentry | Sump pump monitoring + backup |
| 10 | VentBot | Dryer vent inspection |

### Family / Other

| Unit | Name | Purpose |
|------|------|---------|
| 11 | Garden Sentinel | Garden bed monitoring, targeted watering |
| 12 | Chicken Coop Manager | Auto door, egg alerts, feed monitoring |
| 13 | Package Sentinel | Porch delivery reception, locking bin |
| 14 | Wake-Up Ranger | Personalized kid wake-ups |
| 15 | Scavenger Hunt Master | Interactive treasure hunts |
| 16 | Pet Patrol | Automated feeding, play, monitoring |
| 17 | Greenhouse Tender | Climate control, seedling care |
| 18 | Fitness Buddy | Workout tracking, rep counting |

## Build Progress

**See [robot-log.md](robot-log.md) for detailed progress tracking and session notes.**

Current status: Phase 3 complete (kid features). Next: chassis/physical build.

## Development Setup

**Primary dev on bot-00.local** (use mDNS hostname - works on eth or wifi). alex-gtx WSL is the orchestrator with WhatsApp. Pi 4 (10.0.0.51) is a node (currently disconnected).

### Pi File Locations

```
/home/alex/robot/
├── .venv/                # Python virtual environment
├── face/
│   └── face.py           # Animated face display (pygame)
├── features/             # Kid-friendly feature scripts (19 total)
│   ├── animal_quiz.py    # Animal sounds quiz (voice interactive)
│   ├── counting.py       # Count together (voice interactive)
│   ├── magic_word.py     # Polite words game (voice interactive)
│   ├── story_time.py     # Branching story (voice interactive)
│   ├── rhythm_game.py    # Clap along to LED beats (voice interactive)
│   ├── silly_voice.py    # Funny eSpeak voices
│   ├── simon_says.py     # Simon Says game
│   ├── knock_knock.py    # Knock knock jokes
│   ├── lullaby.py        # Soft bedtime story
│   ├── heartbeat.py      # Calm LED pulse
│   ├── lightning.py      # Fast LED burst
│   ├── dance_led.py      # Dance LED patterns
│   ├── goodnight_fade.py # LED fade out
│   ├── demon_hunter.py   # LED + voice (K-pop theme)
│   ├── dance_party.py    # DJ mode
│   ├── countdown.py      # 3-2-1 BLAST OFF
│   ├── treasure_hunt.py  # Verbal clues game
│   ├── peekaboo.py       # Face detection game
│   ├── smile_detect.py   # Smile celebration
│   └── copycat.py        # Expression matching
├── health_check.py       # Boot health check (camera verification)
├── motion.py             # Motion detection → triggers wave
├── override.py           # Bypass wake word - run actions directly
├── sketch/
│   └── sketch.ino        # Arduino firmware
├── stt.py                # Speech-to-text (legacy, unused)
├── listen.py             # Voice command listener (trigger-based)
├── wakeword.py           # Always-listening wake word + VAD + Vosk STT pipeline
├── models/               # hey_jarvis_v0.1.onnx (built-in) + Vosk model
├── train_wakeword/       # Scripts to train custom wake words on GPU (archived)
└── test_arduino.py       # Serial communication test
```

### Face + Motion System

All services auto-start on boot via systemd:

| Service | Description |
|---------|-------------|
| `robot-health` | Boot health check (camera verification) |
| `robot-face` | Animated face display |
| `robot-motion` | Motion detection → wave |
| `robot-wakeword` | Always-listening wake word detection |

```bash
# Service control
sudo systemctl start|stop|restart robot-face
sudo systemctl start|stop|restart robot-motion

# View motion log
tail -f /tmp/motion.log

# Manual wave trigger
echo 'wave' > /tmp/robot_emotion

# Available emotions: happy, sad, surprised, excited, sleepy, angry, wave

# Rock Paper Scissors game
echo 'rps' > /tmp/robot_emotion
```

IPC via `/tmp/robot_emotion` - write emotion name or command to trigger.

Motion threshold: 8000 pixels (moderate movement).

### Voice Control

Say **"Hey Jarvis"** then any of these commands:

| Category | Say This | What Happens |
|----------|----------|--------------|
| **Games** | "rock paper scissors" | RPS game |
| | "simon says" | Simon Says game |
| | "animal quiz" | Animal sounds quiz |
| | "treasure hunt" | Verbal clue treasure hunt |
| | "peek a boo" | Face detection game |
| **Sound** | "counting" / "count" | Count to 5 together |
| | "silly voice" | Funny robot voices |
| | "joke" / "knock knock" | Knock knock jokes |
| | "story" | Interactive bedtime story |
| | "lullaby" / "bedtime" | Soft bedtime mode |
| | "magic word" | Learn please/thank you |
| **LED** | "flash" / "lightning" | Exciting LED burst |
| | "calm" / "heartbeat" | Slow calming pulse |
| | "rhythm" | Clap-along rhythm game |
| | "goodnight" | LED fade out |
| **Combined** | "demon hunter" | K-pop alert mode! 🔥 |
| | "dance" / "party" | DJ mode with lights |
| | "countdown" / "blast off" | 3-2-1 BLAST OFF! |
| **Camera** | "smile" | Smile detector |
| | "copycat" / "copy" | Expression matching |
| **Emotions** | "happy" / "sad" / "angry" | Change face emotion |
| | "wave" / "hello" | Wave animation |

```bash
# Override wake word - run actions directly
ssh alex@bot-00.local "python3 /home/alex/robot/override.py demon_hunter"
ssh alex@bot-00.local "python3 /home/alex/robot/override.py countdown"
ssh alex@bot-00.local "python3 /home/alex/robot/override.py --list"  # Show all

# Watch wake word listener in real time
journalctl -u robot-wakeword -f
```

### OpenClaw Gateway (on bot-00)

Gateway runs on bot-00 for agent orchestration and messaging. Device paired ✓ **Feb 14, 2026**

| Setting | Value |
|---------|-------|
| WebSocket | `ws://bot-00.local:18789` |
| Dashboard | `http://bot-00.local:18789` |
| Config | `~/.openclaw/openclaw.json` |
| Auth Mode | `token` |
| Auth Token | *(see ~/.openclaw/openclaw.json)* |
| Password | *(see local config)* |

```bash
# Service control (user service)
systemctl --user start|stop|restart openclaw-gateway

# View logs
journalctl --user -u openclaw-gateway -f

# Test gateway
curl http://bot-00.local:18789/

# CLI status
openclaw status
openclaw health
```

### Robot Skill

Custom OpenClaw skill at `~/.openclaw/workspace/skills/robot/` for triggering robot actions.

```bash
# Available actions
@robot rps        # Rock Paper Scissors game
@robot wave       # Wave animation
@robot happy      # Happy face
@robot sad        # Sad face
@robot angry      # Angry face
@robot surprised  # Surprised face
@robot excited    # Excited face
@robot sleepy     # Sleepy face
```

Claude can invoke these via the skill when processing voice commands.

### Arduino CLI (on bot-00)

Installed at `/home/alex/bin/arduino-cli`. No laptop needed for Arduino development.

```bash
# Compile
~/bin/arduino-cli compile --fqbn arduino:avr:mega /home/alex/robot/sketch/

# Upload (Arduino must be connected via USB)
~/bin/arduino-cli upload -p /dev/ttyACM0 --fqbn arduino:avr:mega /home/alex/robot/sketch/

# Test communication
python3 /home/alex/robot/test_arduino.py
```

### Serial Port

Arduino appears at `/dev/ttyACM0` when connected to Pi via USB.

## Local Stores

| Store | Location | Good For |
|-------|----------|----------|
| Micro Center | Westmont (~30 min) | Pi, Arduino, electronics, cameras |
| American Science & Surplus | Geneva | Motors, wires, mechanical bits, sundries |
| HobbyTown | Batavia (~5 min) | Chassis kits, servos, RC parts |
| Trossen Robotics | Downers Grove | Serious robot arms, Dynamixel servos |

## Technical Notes

- Arduino Mega has 54 digital pins — plenty of room for expansion
- alex-gtx RTX 4080 can run Llama 3 70B quantized at ~30-40 tok/sec
- All units share the same core architecture — build once, replicate

**See [robot-log.md](robot-log.md) for troubleshooting notes and learnings.**
