JARVIS — Embedded AI Voice Assistant

A fully custom AI voice assistant robot built on Raspberry Pi Zero 2W. Jarvis listens for a wake word, understands natural speech, thinks with a large language model, and responds with a neural voice — all running on a $15 computer the size of a credit card.

What it does


Wake word detection — says "Hey Jarvis" and it instantly responds, powered by OpenWakeWord running fully offline on-device
Natural conversation — stays in conversation mode, remembers what you said earlier in the session, and picks up context naturally
Persistent memory — remembers facts about you between sessions (name, preferences, ongoing projects)
Live web search — looks up current news, weather, sports scores, flight prices, and anything else in real time
Neural TTS — responds in a natural voice using Groq's Orpheus model, falls back to Microsoft Edge TTS when quota runs low
Animated face — a circular pulse ring animation on the TFT screen reacts to each state: idle breathing, listening ripples, speaking pulses, thinking spinner
Proximity awareness — HC-SR04 ultrasonic sensor detects when someone approaches



Hardware

ComponentPartBrainRaspberry Pi Zero 2WDisplayILI9341 2.8" SPI TFT (320×240)MicrophoneINMP441 I2S MEMS micAmplifierMAX98357 I2S Class D ampSpeakers2× small speakersDistance sensorHC-SR04 ultrasonicStorage8GB microSD

Wiring highlights:


Audio runs entirely over I2S — mic and amp share the same clock lines (GPIO 18/19), amp muted via SD_MODE pin (GPIO 16) during listening to eliminate crosstalk noise
Display connected over SPI0 with custom fbtft overlay, face renders directly to /dev/fb0 framebuffer in RGB565
Ultrasonic on GPIO 23 (TRIG) and GPIO 26 (ECHO)



Software stack

wake word       OpenWakeWord (hey_jarvis_v0.1.onnx) — offline, instant
transcription   Groq Whisper large-v3-turbo — English forced, ~0.9s
LLM             Groq llama-3.3-70b-versatile — 1-3 sentence responses
TTS             Groq Orpheus v1 (daniel voice) → Edge TTS fallback
face            Custom framebuffer renderer — RGB565, 5 animated states
memory          JSON-based persistent fact extraction via LLM
search          DuckDuckGo (ddgs) + OpenWeatherMap
audio pipeline  arecord → numpy VAD → sox → Whisper / sox → aplay
amp control     RPi.GPIO → GPIO16 → MAX98357 SD_MODE (hardware mute)
autostart       systemd service with restart-on-failure


Project structure

aria/
├── main.py                  — main conversation loop
├── audio/
│   ├── wakeword.py          — OpenWakeWord wake detection
│   ├── listen.py            — VAD + Whisper transcription
│   ├── speak.py             — Orpheus + Edge TTS with fallback
│   └── amp_control.py       — GPIO16 hardware amp mute
├── brain/
│   └── tools.py             — search, weather, news, time intents
├── face/
│   └── aria_face.py         — framebuffer face renderer
├── memory/
│   ├── memory_manager.py    — fact extraction and persistence
│   └── memory.json          — stored user facts
├── sensors/
│   └── ultrasonic.py        — HC-SR04 distance reading
└── assets/
    ├── beep.wav             — startup sound
    ├── listening.wav        — "yes I'm listening"
    ├── didnt_catch.wav      — "sorry didn't catch that"
    └── goodbye.wav          — goodbye response


How it works

Boot → systemd starts Jarvis → beep
     → OpenWakeWord listens continuously (offline, ~0ms latency)
     → "Hey Jarvis" detected → amp on → "Yes, I'm listening"
     → VAD records until 1s silence → Groq Whisper transcribes
     → intent detection → optional tool call (search/weather/time)
     → Groq LLM generates response → Orpheus TTS speaks it
     → amp off → back to wake word listening
     → "Goodbye" → saves memory → sleep mode


Key engineering challenges solved


I2S crosstalk noise — mic and amp share clock lines; solved by hardware-muting the amp via SD_MODE GPIO during all non-speaking phases
Wake word latency — Vosk-based detection had 10-15s delays; replaced with OpenWakeWord ONNX model for sub-2s detection
Audio format chain — MAX98357 only accepts S32_LE at 48kHz stereo; all audio piped through sox for conversion
RAM constraints — Pi Zero has 512MB; tuned with 1GB swap, MemoryMax systemd limit, and streaming audio pipeline to avoid buffering full responses
Framebuffer rendering — no GPU, no display server; face renders directly to /dev/fb0 in RGB565 with smooth color interpolation between states



APIs used

ServiceUsageFree tierGroqLLM + Whisper STT + Orpheus TTSGenerous daily limitsOpenWeatherMapLive weather1000 calls/dayDuckDuckGoWeb searchUnlimitedMicrosoft Edge TTSTTS fallbackUnlimited
What's next


Tank chassis with ESP32 motor control for physical movement
Music/YouTube playback via yt-dlp + mpv
Alarm and timer system
WiFi hotspot fallback for connecting to new networks
Ultrasonic proximity integration into main pipeline
<img width="1134" height="1493" alt="jarvis from the front, off" src="https://github.com/user-attachments/assets/6db60850-3a59-48cf-bdfe-3824c274a063" />

