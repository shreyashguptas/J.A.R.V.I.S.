# Raspberry Pi Zero 2 W Voice Assistant

A complete voice assistant project using Raspberry Pi Zero 2 W with INMP441 microphone, MAX98357A amplifier, and Groq API integration.

## 🎯 Project Overview

This project creates a voice assistant that:
- Records audio using INMP441 I2S microphone
- **Press button once to start recording, press again to stop** (no time limit!)
- Transcribes speech using Groq Whisper API
- Processes queries with Groq LLM
- Generates speech responses using Groq TTS
- Plays audio through MAX98357A I2S amplifier
- Controlled via GPIO button

## 🔧 Hardware Requirements

### Components
- **Raspberry Pi Zero 2 W** (mainboard)
- **Google Voice HAT** (AIY Voice HAT v1.0 or compatible)
  - Includes: Built-in dual MEMS microphones
  - Includes: 3W Class D amplifier
  - Interface: I2S digital audio
- **3W 4Ω Speaker** (connects to Voice HAT)
- **Push Button** (for activation)
- **Optional: Jumper wires** (for button connection)

### Wiring Diagram

```
Google Voice HAT:
└── Mounts directly onto Pi's 40-pin GPIO header

Speaker:
├── Red wire   → Voice HAT Speaker + terminal
└── Black wire → Voice HAT Speaker - terminal

Button:
├── Terminal 1 → Pi GPIO17 (Pin 11) - accessible through Voice HAT
└── Terminal 2 → Pi GND (Pin 6) - accessible through Voice HAT

Optional - Amplifier Shutdown Control:
└── Voice HAT SD pin → Pi GPIO27 (Pin 13) [reduces audio clicks]
```

**Note**: The Google Voice HAT is a complete audio solution that includes microphone array and speaker amplifier in one board. It connects to all 40 GPIO pins but passes through unused pins for your button.

## 🚀 Quick Start

### 1. Hardware Setup
1. **Power off** the Raspberry Pi completely
2. **Mount the Google Voice HAT** onto the 40-pin GPIO header (press down firmly)
3. **Connect the speaker** to the Voice HAT speaker terminals (red to +, black to -)
4. **Connect the button** to GPIO17 and GND
5. **Power on** the Raspberry Pi

### 2. Software Configuration

#### Enable I2S Audio
Add to `/boot/firmware/config.txt`:
```bash
sudo nano /boot/firmware/config.txt
```

Then add this at the bottom of the file

```bash
# I2S Configuration for Google Voice HAT
dtparam=i2s=on
dtoverlay=googlevoicehat-soundcard
```

**Then reboot:**
```bash
sudo reboot
```

#### Install Dependencies

- Make sure this repo is git cloned then run the following

```bash
cd J.A.R.V.I.S.
```

**Option 1: Use System Packages (Recommended for Pi Zero 2 W)**

This avoids compilation issues and memory constraints:

```bash
# Install system packages
sudo apt update
sudo apt install -y python3-pyaudio python3-rpi.gpio python3-requests python3-numpy python3-pip

# Create virtual environment with system packages
python3 -m venv --system-site-packages ~/venvs/pi
source ~/venvs/pi/bin/activate

# Install remaining packages
pip install python-dotenv
```

**Option 2: Build from Source (if you have time and patience)**

Only use this if Option 1 doesn't work:

```bash
# Install build dependencies
sudo apt install -y python3-dev portaudio19-dev libatlas-base-dev

# Create virtual environment
python3 -m venv ~/venvs/pi
source ~/venvs/pi/bin/activate

# Install packages (this may take 30-60 minutes on Pi Zero 2 W)
pip install -r config/requirements.txt
```

#### Configure API Key
Copy the environment template and add your API key:
```bash
cp env.example .env
nano .env
```

Edit the `.env` file and replace:
```
GROQ_API_KEY=YOUR_GROQ_API_KEY_HERE
```

### 3. Test Setup
```bash
# Test microphone (records for 5 seconds)
arecord -D plughw:0,0 -c1 -r 16000 -f S16_LE -t wav -d 5 test.wav

# Test speaker
aplay -D plughw:0,0 test.wav

# Run voice assistant
python3 voice_assistant.py
```

## 🎙️ How to Use

1. **Press the button** - Recording starts
2. **Speak your question** - Take as long as you need (no time limit)
3. **Press the button again** - Recording stops
4. **Wait for response** - Processing and playback happens automatically
5. **Listen to the answer** - Response plays through speaker
6. **Repeat** - Press button to ask another question

## 📁 Project Structure

```
voice_assistant/
├── README.md                 # This file
├── voice_assistant.py        # Main application
├── env.example              # Environment variables template
├── config/
│   ├── config.txt.example    # Example Pi configuration
│   └── requirements.txt      # Python dependencies
├── docs/
│   ├── hardware_setup.md     # Detailed hardware guide
│   ├── troubleshooting.md    # Common issues and solutions
│   └── api_setup.md          # Groq API configuration
├── scripts/
│   ├── setup.sh              # Automated setup script
│   ├── test_audio.py         # Audio testing utilities
│   └── install_deps.sh       # Dependency installer
└── examples/
    ├── simple_test.py        # Basic functionality test
    └── mic_test.py           # Microphone testing
```

## 🔍 Troubleshooting

### Microphone Not Working

**Common Issues:**

1. **Hardware Connections**
   - Verify all wiring matches diagram
   - Check for loose breadboard connections
   - Ensure proper power supply

2. **I2S Configuration**
   - Verify `/boot/firmware/config.txt` settings
   - Reboot after configuration changes
   - Check with `arecord -l`

3. **Device Detection**
   ```bash
   # List audio devices
   arecord -l
   aplay -l
   
   # Test recording (5 seconds)
   arecord -D plughw:0,0 -c1 -r 16000 -f S16_LE -d 5 test.wav
   ```

4. **Python Environment**
   - Ensure virtual environment is activated
   - Check PyAudio installation
   - Verify device permissions

### Audio Quality Issues

- **Microphone Too Quiet**: 
  - Increase `MICROPHONE_GAIN` in `.env` file (try 2.0, 3.0, or 4.0)
  - Check ALSA capture volume: `amixer -c 0 set Capture 100%`
  - Verify microphone is close enough (6-12 inches)
- **Speaker Volume Low**: Check speaker connections and amplifier power
- **Distorted Audio**: 
  - Reduce `MICROPHONE_GAIN` if too high
  - Verify sample rate settings (16000 Hz)
- **No Audio**: Test with `aplay` command first

### API Issues

- **Authentication**: Verify Groq API key
- **Network**: Check internet connectivity
- **Rate Limits**: Monitor API usage

## 🛠️ Development

### Testing Audio Components
```bash
# Test microphone recording
python3 scripts/test_audio.py

# Test individual components
python3 examples/mic_test.py
```

### Customization

You can customize settings via the `.env` file:

```bash
# Edit .env file
nano .env
```

**Available Settings:**
- `MICROPHONE_GAIN` - Amplify microphone input (default: 2.0)
  - `1.0` = No amplification
  - `2.0` = Double volume (recommended)
  - `3.0` = Triple volume
  - `4.0` = Quadruple volume (may distort)
- `RECORD_SECONDS` - Recording duration in seconds (default: 5)
- `BUTTON_PIN` - GPIO pin for button (default: 17)
- `SAMPLE_RATE` - Audio sample rate (default: 16000)

**Example `.env`:**
```env
GROQ_API_KEY=your_api_key_here
MICROPHONE_GAIN=2.5
RECORD_SECONDS=7
```

## 📚 API Documentation

### Groq API Endpoints
- **Whisper**: `https://api.groq.com/openai/v1/audio/transcriptions`
- **LLM**: `https://api.groq.com/openai/v1/chat/completions`
- **TTS**: `https://api.groq.com/openai/v1/audio/speech`

### Models Used
- **Whisper**: `whisper-large-v3-turbo` (fastest, most accurate)
- **LLM**: `openai/gpt-oss-20b`
- **TTS**: `playai-tts` with `Chip-PlayAI` voice

## 🤝 Contributing

1. Fork the repository
2. Create feature branch
3. Make changes
4. Test thoroughly
5. Submit pull request

## 📄 License

MIT License - see LICENSE file for details

## 🆘 Support

For issues and questions:
1. Check troubleshooting guide
2. Review hardware setup
3. Test individual components
4. Create GitHub issue with logs

---

**Happy Voice Assisting! 🎤🤖**
