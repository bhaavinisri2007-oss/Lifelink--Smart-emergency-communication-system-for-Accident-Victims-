# 🚑 Lifelink

**Smart emergency communication for accident victims.**
Lifelink detects a crash automatically, sends the victim's location and critical medical information to the right responders, and keeps working even when the network is down.

> ⚠️ Prototype for educational/demo use. Not a certified medical or emergency device.

---

## ✨ Why Lifelink is different

| Feature | What it does |
|---|---|
| **Automatic crash detection** | Reads accelerometer data and triggers an alert when a sustained impact is detected, even if the victim cannot press a button. |
| **Offline-first delivery** | Tries SMS → internet → nearby mesh (Bluetooth/peer-to-peer) peers in order. If all fail, the alert is queued and retried when connectivity returns. |
| **Medical card in a QR code** | A compact, encoded medical profile (blood type, allergies, conditions, emergency contact) that bystanders or paramedics can scan. |
| **Triage-based routing** | Severity decides which responders are alerted (ambulance, hospital, police, fire) and the nearest unit of each type is selected by distance. |
| **Zero-dependency core** | Pure Python standard library, so it runs on low-cost hardware and older phones. |

---

## 🧠 How it works

```
 Phone sensors ──► CrashDetector ──► EmergencyAlert ──► Triage ──► Router
 (accelerometer)   (sustained g)     (location, g,       (severity)  (nearest responders)
                                      medical card)                     │
                                                                        ▼
                                   ┌─────────── Channel chain ──────────┐
                                   │  SMS  →  Internet  →  Mesh peers   │
                                   └────────────────────────────────────┘
                                        │ all failed?
                                        ▼
                                 Outbox queue → retry on reconnect
```

1. **Detect** – the crash detector requires several consecutive high-g samples, which filters out a phone being dropped or a bump.
2. **Assess** – the alert is assigned a triage level (`CRITICAL`, `SERIOUS`, `MINOR`).
3. **Route** – the nearest responders for that triage level are selected.
4. **Transmit** – a compact payload (typically under 300 bytes) goes out over the first channel that works.
5. **Recover** – unsent alerts stay in the outbox and are flushed automatically.

---

## 📁 Project structure

```
lifelink/
├── README.md        # This file
├── lifelink.py      # Core system: detection, triage, routing, delivery
└── requirements.txt # (empty: standard library only)
```

---

## 🚀 Getting started

**Requirements:** Python 3.9+

```bash
git clone https://github.com/<your-username>/lifelink.git
cd lifelink
python lifelink.py
```

The included demo simulates a sensor stream containing a crash, a medical profile, nearby responders, and unreliable network channels.

Example output (values vary with the random seed):

```
[detector] Crash detected, peak 15.8 g
[triage]   CRITICAL
  [SMS       ] failed (231 bytes)
  [Internet  ] delivered (231 bytes)
  -> City Ambulance 12 (ambulance) 0.4 km away
  -> Central Hospital (hospital) 0.9 km away
  -> Traffic Police Post (police) 1.2 km away
[qr-card]  eyJpZCI6...
```

---

## 🧩 Using the library in your own code

```python
from lifelink import (
    CrashDetector, LifelinkNode, EmergencyAlert, MedicalProfile,
    Location, Responder, SimulatedChannel, assess_triage
)

node = LifelinkNode(
    channels=[SimulatedChannel("SMS", 0.9), SimulatedChannel("Internet", 0.8)],
    responders=[Responder("Unit 7", "ambulance", Location(13.08, 80.27))],
)
```

To connect real hardware or services, replace `SimulatedChannel` with a class that implements `is_available()` and `send(payload)`:

- **SMS:** a telecom or gateway API
- **Internet:** an HTTPS endpoint or MQTT broker
- **Mesh:** Bluetooth LE advertising or LoRa radio

---

## 🗺️ Roadmap

- [ ] Real accelerometer and GPS input from Android/iOS
- [ ] Bluetooth LE mesh relay so nearby phones forward alerts
- [ ] Encrypted payloads (AES-GCM) with per-user keys
- [ ] Web dashboard for responders with live map
- [ ] Two-way messaging so responders can ask the victim questions
- [ ] Unit tests and CI on GitHub Actions

---

## 🔐 Privacy and safety

- Medical data should be encrypted in production and shared only with emergency services.
- Lifelink is an aid, not a replacement for calling emergency numbers. Always call your local emergency number directly when you can.
- Automatic detection can produce false positives. Users should be able to cancel an alert during a short countdown.

---

## 🤝 Contributing

Pull requests are welcome. Please open an issue first to discuss major changes.

## 📄 License

MIT
