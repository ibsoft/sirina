# Sirina

Sirina is a local speech toolkit for AI agents. It provides text-to-speech, speech-to-text, microphone capture, voice
activity detection, model download/check commands, and a small Python API.

Sirina is designed for agents that need a reusable local speech layer:

- Speak responses through a local TTS model.
- Transcribe WAV files through local STT models.
- Listen to the microphone using VAD instead of a wake word.
- Build always-listening assistant loops.
- Use a CLI for quick testing and a Python API for integration.

## Features

- Sirina/Piper TTS voice, selected with `voice="sirina"`
- Kokoro TTS voices, listed with `sirina voices`
- Parakeet TDT STT, selected with `engine="tdt"`
- Parakeet CTC STT, selected with `engine="ctc"`
- Silero VAD microphone utterance capture
- Automatic mono conversion and resampling for WAV transcription
- `SIRINA_MODEL_DIR` support for external model directories

## Install

From this checkout:

```bash
cd /mnt/data/Desktop/Sirina
python3 -m pip install -e .
```

For development:

```bash
python3 -m pip install -e ".[dev]"
```

For CUDA ONNX Runtime:

```bash
python3 -m pip install -e ".[cuda]"
```

## Models

Sirina needs model files before real TTS/STT can run.

Download all models:

```bash
sirina download --group all
```

Download only TTS models:

```bash
sirina download --group tts
```

Download only STT/VAD models:

```bash
sirina download --group asr
```

Check model files and checksums:

```bash
sirina check-models --group all
```

Sirina resolves model files in this order:

1. `SIRINA_MODEL_DIR`, if set
2. `models/` in this source checkout
3. bundled metadata under `sirina.assets`
4. `../models` when Sirina is inside a nested checkout
5. `~/.sirina/models`

Use a custom model directory:

```bash
export SIRINA_MODEL_DIR=/path/to/models
sirina check-models --group all
```

The default Sirina TTS model path is:

```text
models/TTS/sirina.onnx
```

Large `.onnx` and `.bin` files are intentionally not tracked by git. Small YAML/JSON/pickle metadata files are included.

## CLI Quick Start

List available voices:

```bash
sirina voices
```

## Voices And Languages

Sirina currently supports English speech workflows.

Text-to-speech language support:

- `sirina`: English, configured as `en-us`, 22050 Hz output
- Kokoro voices: English voices using Sirina's `en_us` phonemizer, 24000 Hz output

Speech-to-text language support:

- `tdt`: Parakeet TDT ASR, 16000 Hz internal sample rate
- `ctc`: Parakeet CTC ASR, 16000 Hz internal sample rate

The current Sirina API does not expose a language selector. Both Sirina/Piper and Kokoro TTS call the phonemizer with
`en_us`, and STT uses the bundled Parakeet model configs. WAV files passed to `sirina transcribe` may use other sample
rates or stereo channels; Sirina converts them to mono 16 kHz before STT.

Built-in Sirina voice:

| Voice | Language | Notes |
| --- | --- | --- |
| `sirina` | English (`en-us`) | Default Sirina/Piper voice |

Kokoro voice prefixes:

| Prefix | Meaning |
| --- | --- |
| `af_` | American English, feminine voice |
| `am_` | American English, masculine voice |
| `bf_` | British English, feminine voice |
| `bm_` | British English, masculine voice |

Kokoro voices included in the current voice bundle:

| Voice | Language/Accent | Presentation |
| --- | --- | --- |
| `af_alloy` | American English | Feminine |
| `af_aoede` | American English | Feminine |
| `af_bella` | American English | Feminine |
| `af_jessica` | American English | Feminine |
| `af_kore` | American English | Feminine |
| `af_nicole` | American English | Feminine |
| `af_nova` | American English | Feminine |
| `af_river` | American English | Feminine |
| `af_sarah` | American English | Feminine |
| `af_sky` | American English | Feminine |
| `am_adam` | American English | Masculine |
| `am_echo` | American English | Masculine |
| `am_eric` | American English | Masculine |
| `am_fenrir` | American English | Masculine |
| `am_liam` | American English | Masculine |
| `am_michael` | American English | Masculine |
| `am_onyx` | American English | Masculine |
| `am_puck` | American English | Masculine |
| `bf_alice` | British English | Feminine |
| `bf_emma` | British English | Feminine |
| `bf_isabella` | British English | Feminine |
| `bf_lily` | British English | Feminine |
| `bm_daniel` | British English | Masculine |
| `bm_fable` | British English | Masculine |
| `bm_george` | British English | Masculine |
| `bm_lewis` | British English | Masculine |

Check the exact voices available on a machine with:

```bash
sirina voices
```

Speak through speakers:

```bash
sirina say "System online."
```

Save speech and play it:

```bash
sirina say "System online." --output sirina-test.wav
```

Save speech without playback:

```bash
sirina say "System online." --output sirina-test.wav --no-play
```

Transcribe an audio file:

```bash
sirina transcribe your-audio.wav --engine tdt
```

Record one microphone utterance and transcribe it:

```bash
sirina listen --engine tdt
```

Record one microphone utterance, save it, and transcribe it:

```bash
sirina listen --engine tdt --output test.wav
```

## Audio Device Selection

Sirina uses `sounddevice` for local microphone and speaker access. Device selection is config-backed and supports
autodetect by default.

List available devices:

```bash
sirina audio-devices
```

Example output:

```text
index  in  out  default  name
    0   0    2  output   Built-in Audio Analog Stereo
    1   1    0  input    USB Microphone
    2   2    2  -        USB Headset
```

The columns mean:

- `index`: numeric device ID accepted by config and CLI flags
- `in`: available input channels; use devices with `in > 0` as microphones
- `out`: available output channels; use devices with `out > 0` as speakers
- `default`: system default input/output marker reported by `sounddevice`
- `name`: device name; unique substrings are accepted

Config defaults live in `src/sirina/config.py`:

```python
SIRINA_AUDIO_INPUT_DEVICE = os.getenv("SIRINA_AUDIO_INPUT_DEVICE", "auto")
SIRINA_AUDIO_OUTPUT_DEVICE = os.getenv("SIRINA_AUDIO_OUTPUT_DEVICE", "auto")
```

Use environment variables to select devices without editing code:

```bash
export SIRINA_AUDIO_INPUT_DEVICE=1
export SIRINA_AUDIO_OUTPUT_DEVICE=0
```

You can also use a unique device name substring:

```bash
export SIRINA_AUDIO_INPUT_DEVICE="USB Microphone"
export SIRINA_AUDIO_OUTPUT_DEVICE="Built-in Audio"
```

Autodetect is the default:

```bash
export SIRINA_AUDIO_INPUT_DEVICE=auto
export SIRINA_AUDIO_OUTPUT_DEVICE=auto
```

Autodetect behavior:

1. Try the system default device from `sounddevice`.
2. Verify it has the required channel type: input channels for microphone, output channels for speakers.
3. If the default is unavailable or unsuitable, choose the first device with matching channels.
4. If no matching device exists, return `None` and let `sounddevice` raise the runtime device error.

Override devices per command:

```bash
sirina say "Testing speaker." --output-device 0
sirina say "Testing headset." --output-device "USB Headset"
sirina listen --input-device 1 --engine tdt
sirina listen --input-device "USB Microphone" --engine tdt
```

Python API overrides:

```python
from sirina import TextToSpeech, record_utterance

tts = TextToSpeech(voice="sirina")
tts.play("Testing selected speaker.", output_device="USB Headset")

audio = record_utterance(input_device="USB Microphone")
```

## Speech-To-Text Notes

`sirina transcribe` accepts WAV files. The STT models operate at 16 kHz mono internally, and Sirina converts input WAVs
before transcription:

- Stereo files are mixed down to mono.
- Non-16 kHz files are resampled to 16 kHz.
- Empty files raise a clear error.
- Missing files raise `FileNotFoundError`.

Use `tdt` for better accuracy:

```bash
sirina transcribe meeting.wav --engine tdt
```

Use `ctc` when you want a simpler/faster engine:

```bash
sirina transcribe meeting.wav --engine ctc
```

## Text-To-Speech Notes

The default voice is `sirina`:

```bash
sirina say "Hello from Sirina."
```

Use a Kokoro voice by name:

```bash
sirina voices
sirina say "Hello from Kokoro." --voice af_alloy
```

`sirina say --output file.wav` now plays by default. Use `--no-play` for file-only generation.

## Python API

Basic TTS:

```python
from sirina import TextToSpeech

tts = TextToSpeech(voice="sirina")
tts.play("System online.")
tts.save_wav("Saved to disk.", "sirina-output.wav")
```

Basic STT:

```python
from sirina import SpeechToText

stt = SpeechToText(engine="tdt")
text = stt.transcribe_file("your-audio.wav")
print(text)
```

Microphone capture:

```python
from sirina import SpeechToText, record_utterance

audio = record_utterance()
if audio.size:
    text = SpeechToText(engine="tdt").transcribe(audio)
    print(text)
```

## Always-Listening Agent Loop

Sirina does not require a wake word. The usual pattern is an always-running VAD loop:

1. Open the microphone.
2. Wait until VAD detects speech.
3. Buffer audio until silence.
4. Transcribe the utterance.
5. Send text to your agent or LLM.
6. Speak the response with TTS.
7. Repeat.

Minimal example:

```python
from sirina import SpeechToText, TextToSpeech, record_utterance

stt = SpeechToText(engine="tdt")
tts = TextToSpeech(voice="sirina")

while True:
    audio = record_utterance(
        silence_seconds=0.8,
        max_seconds=20.0,
        speech_start_timeout_s=60.0,
    )
    if audio.size == 0:
        continue

    user_text = stt.transcribe(audio).strip()
    if not user_text:
        continue

    print(f"User: {user_text}")

    # Replace this with your agent/LLM call.
    assistant_text = f"You said: {user_text}"

    print(f"Sirina: {assistant_text}")
    tts.play(assistant_text)
```

Recommended safeguards for no-wake-word agents:

- Do not listen while TTS is playing.
- Ignore empty or very short transcriptions.
- Add a cooldown after speaking if your speakers feed back into the microphone.
- Keep `max_seconds` bounded so one long noise segment cannot block the loop forever.
- Use a wake word or push-to-talk later if false activations become a problem.

## External Program Integration

Any external Python program can import Sirina after installation:

```python
from sirina import SpeechToText, TextToSpeech

def respond_to_audio(path: str) -> str:
    stt = SpeechToText(engine="tdt")
    tts = TextToSpeech(voice="sirina")

    user_text = stt.transcribe_file(path)
    response = f"I heard: {user_text}"
    tts.play(response)
    return response
```

For non-Python programs, shell out to the CLI:

```bash
sirina transcribe input.wav --engine tdt
sirina say "Response text."
```

## Configuration

Core defaults live in:

```text
src/sirina/config.py
```

Useful defaults include:

- `DEFAULT_TTS_VOICE = "sirina"`
- `DEFAULT_STT_ENGINE = "tdt"`
- `INPUT_SAMPLE_RATE = 16000`
- `SIRINA_AUDIO_INPUT_DEVICE = os.getenv("SIRINA_AUDIO_INPUT_DEVICE", "auto")`
- `SIRINA_AUDIO_OUTPUT_DEVICE = os.getenv("SIRINA_AUDIO_OUTPUT_DEVICE", "auto")`
- `LISTEN_SILENCE_SECONDS = 0.8`
- `LISTEN_MAX_SECONDS = 20.0`
- `LISTEN_SPEECH_START_TIMEOUT_SECONDS = 10.0`

Model paths and checksums are also centralized in `src/sirina/config.py`.

## Troubleshooting

If `sirina check-models --group all` says models are missing:

```bash
sirina download --group all
```

If `sirina say` saves a file but you hear nothing:

1. Confirm playback works outside Sirina.
2. Check the system output device and volume.
3. Try direct playback:

```bash
sirina say "Testing audio output."
```

If microphone capture records nothing:

```bash
sirina listen --vad-threshold 0.5 --speech-start-timeout 30
```

If transcription seems wrong:

1. Try `--engine tdt`.
2. Make sure the input is actual speech, not the TTS output you just generated.
3. Test with a clean WAV file.

If an external program cannot import `sirina`:

```bash
cd /mnt/data/Desktop/Sirina
python3 -m pip install -e .
```

## Development

Run tests:

```bash
PYTHONPATH=src python3 -m pytest -q
```

Compile-check imports:

```bash
python3 -m compileall -q src tests
```
