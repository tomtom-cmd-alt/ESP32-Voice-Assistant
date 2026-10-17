# Home Assistant AI Voice Assistant Setup Guide

This guide outlines the system architecture, configuration changes, and hardware setup required to use your **Elegoo ESP32 Super Starter Kit** as a Wi-Fi-controlled smart actuator driven by an AI voice pipeline running on **Home Assistant**.

---

## 🏗️ System Architecture Flow
1. **Audio Input:** Your smartphone or computer microphone captures audio upon hearing a wake word.
2. **Transcription (STT):** Audio data streams to **Deepgram** (running the low-latency `nova-2` model).
3. **Inference (LLM Engine):** The text transcript triggers the **NVIDIA NIM API** to invoke the open-weights `deepseek-ai/deepseek-v3` architecture.
4. **Speech Synthesis (TTS):** The structural text reply converts back to real-time audio via the keyless **Puter.js** proxy library layout.
5. **Hardware Execution:** Home Assistant sends a light webhook command over Wi-Fi to your **Elegoo ESP32** via an internal ESPHome configuration to trigger kit components (like servos or LEDs).

---

## 🛠️ Phase 1: API Configuration & Initialization

### 1. Speech-To-Text (Deepgram)
1. Sign up for a free developer account at the [Deepgram Console](https://console.deepgram.com).
2. Generate an API Key within your project space. Ensure permissions include `Usage: Read` and `API: Write`.
3. Save the key string safely.

### 2. LLM Engine (NVIDIA NIM)
1. Navigate to [build.nvidia.com](https://build.nvidia.com) and register a free developer account.
2. Search the text directory for **`deepseek-ai/deepseek-v3`**.
3. Select the model and click **Fetch API Key** to secure your baseline API credentials.

### 3. Text-To-Speech (Puter.js Engine Proxy)
* Puter operates as a keyless public proxy infrastructure. No explicit API token or billing header configuration is required to invoke standard generic endpoints.

---

## 🏡 Phase 2: Integrating Engines into Home Assistant

Open your main Home Assistant instance directory and paste these integrations directly into your core `configuration.yaml` file:

```yaml
# Home Assistant configuration.yaml configuration

# 1. Register Speech-To-Text Engine (Deepgram)
stt:
  - platform: deepgram
    api_key: "YOUR_DEEPGRAM_API_KEY_HERE"
    model: "nova-2-general"

# 2. Register LLM Engine (NVIDIA NIM / DeepSeek V3)
llm:
  - platform: openai
    name: "DeepSeek_V3_NIM"
    api_key: "YOUR_NVIDIA_NIM_API_KEY_HERE"
    base_url: "https://integrate.api.nvidia.com/v1"
    model: "deepseek-ai/deepseek-v3"

# 3. Register Text-To-Speech Engine (Puter Proxy Integration)
tts:
  - platform: generic
    name: "Puter_TTS"
    url: "https://api.puter.com/v1/tts"
    query_params:
      text: "{{ text }}"
      voice: "en-US-Neural-Male"
```

*Save changes and reboot your Home Assistant Core server.*

---

## 🤖 Phase 3: Provisioning the Elegoo ESP32 via ESPHome

Your Elegoo kit board lacks specialized native audio hardware. It will act as a network-controlled node running **ESPHome**.

1. Connect your Elegoo ESP32 board to your computer using a standard micro-USB or USB-C programming data cable.
2. Open your Home Assistant **ESPHome Dashboard** add-on page.
3. Create a new device named `elegoo-voice-actuator`, select **ESP32** as the target chip, and paste the following node schematic configuration:

```yaml
esphome:
  name: elegoo-voice-actuator
  friendly_name: "Elegoo AI Actuator"

esp32:
  board: esp32dev
  framework:
    type: arduino

# Establishes network communication loops back to the Home Assistant dashboard
api:
  encryption:
    key: "GENERATED_ESPHOME_API_ENCRYPTION_KEY_HERE"

logger:
  baud_rate: 115200

wifi:
  ssid: "YOUR_HOME_WIFI_NAME"
  password: "YOUR_HOME_WIFI_PASSWORD"
  fast_connect: true

# Configures output pins for components included in your Elegoo starter kit
servo:
  - id: kit_servo
    output: servo_pwm

output:
  - platform: ledc
    pin: GPIO13
    id: servo_pwm
    frequency: 50Hz

light:
  - platform: binary
    name: "Kit Indicator LED"
    output: led_pin

output:
  - platform: gpio
    pin: GPIO12
    id: led_pin
```

4. Click **Install** in the upper right-hand corner and choose **Plug into this computer via WebUSB** to build and flash the node firmware autonomously.

---

## 🎙️ Phase 4: Final Voice Assistant Pipeline Orchestration

1. In the Home Assistant Dashboard, navigate to **Settings ➡️ Voice Assistants** and click **Add Assistant**.
2. Map your customized execution properties:
   * **Speech-to-Text:** Select your configured `Deepgram STT` profile hook.
   * **Conversation Agent:** Choose `Deepseek_V3_NIM`.
   * **Text-to-Speech:** Route outputs through your `Puter_TTS` service engine.
3. Open the **Home Assistant Companion App** on your mobile phone or peripheral system setup.
4. Navigate to **App Settings ➡️ Voice Assistant**, check the newly initialized custom pipeline engine, and toggle **Persistent Background Wake Word Detection** to lock the system into a constant-listening operational status.
5. Create standard Home Assistant Automation macros linking intents (e.g., *"Activate servo motor"*) to target hardware executions targeting your `elegoo-voice-actuator` pin lines.
