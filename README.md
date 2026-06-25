# Speech-rate-analyzer
# ðŸŽ™ï¸ C5. Speech Rate Analyzer

A browser-based tool that records your voice and calculates your **speaking speed in words per minute (WPM)** along with total speaking duration â€” powered by the Anthropic Claude API for transcription.

---

## ðŸ“¸ Preview

> Record â†’ Transcribe â†’ Analyze â†’ JSON output

```json
{
  "words_per_minute": 125,
  "total_duration_seconds": 14.2,
  "speaking_duration_seconds": 11.8,
  "word_count": 25,
  "long_pauses_detected": 1,
  "transcript": "This is what you said during the recording..."
}
```

---

## ðŸ“ Project Structure

```
speech-rate-analyzer/
â”œâ”€â”€ index.html      # UI layout and structure
â”œâ”€â”€ style.css       # Styling and responsive design
â”œâ”€â”€ analyzer.js     # Core recording logic, WPM calculation, API call
â””â”€â”€ README.md       # You are here
```

---

## âœ¨ Features

- ðŸŽ¤ **Live microphone recording** via `MediaRecorder` API
- ðŸ“Š **Real-time waveform** visualizer using Web Audio API
- ðŸ§  **AI-powered transcription** via Claude (Anthropic API)
- â±ï¸ **WPM calculation** based on net speaking time (silence excluded)
- â¸ï¸ **Long pause detection** â€” gaps >1.5s are flagged and excluded from WPM
- ðŸ”‡ **Interrupted speech handling** â€” tracks and skips silence spans accurately
- ðŸ“‹ **JSON output** with one-click copy

---

## ðŸš€ Getting Started

### Prerequisites

- A modern browser (Chrome or Edge recommended for best microphone support)
- An [Anthropic API key](https://console.anthropic.com/)

### Setup

1. **Clone the repository**

   ```bash
   git clone https://github.com/your-username/speech-rate-analyzer.git
   cd speech-rate-analyzer
   ```

2. **Add your API key**

   Open `analyzer.js` and replace the placeholder on line 7:

   ```js
   const ANTHROPIC_API_KEY = "YOUR_API_KEY_HERE";
   ```

3. **Open in browser**

   Simply open `index.html` in your browser â€” no build step or server required.

   ```bash
   open index.html       # macOS
   start index.html      # Windows
   xdg-open index.html   # Linux
   ```

---

## ðŸ› ï¸ How It Works

### Recording & Silence Detection

The app uses the **Web Audio API** (`AnalyserNode`) to compute real-time RMS amplitude. Frames below the silence threshold (`0.015`) are classified as silent and their duration is tracked separately.

```
Speaking time = Total duration âˆ’ Total silence duration
WPM = Word count Ã· (Speaking time in minutes)
```

### Long Pause Handling

Any continuous silence longer than **1.5 seconds** is flagged as a long pause and counted in the output. This handles interrupted speech scenarios without inflating total duration.

### Transcription

Audio is captured as a `audio/webm` blob, converted to base64, and sent to the **Anthropic Claude API** (`claude-sonnet-4-20250514`). The transcript is used purely for word counting.

---

## âš™ï¸ Configuration

You can adjust these constants in `analyzer.js`:

| Constant | Default | Description |
|---|---|---|
| `SILENCE_THRESHOLD` | `0.015` | RMS amplitude below which audio is considered silence |
| `LONG_PAUSE_MS` | `1500` | Minimum silence duration (ms) to count as a long pause |

---

## ðŸ“¤ Output Format

| Field | Type | Description |
|---|---|---|
| `words_per_minute` | `number` | WPM based on net speaking time |
| `total_duration_seconds` | `number` | Full recording length |
| `speaking_duration_seconds` | `number` | Recording length minus silence |
| `word_count` | `number` | Total words in transcript |
| `long_pauses_detected` | `number` | Number of pauses longer than 1.5s |
| `transcript` | `string` | Full AI-generated transcript |

---

## ðŸ”’ Notes on API Key Security

> âš ï¸ **Do not commit your API key to a public repository.**

For production use, consider:
- Storing the key in a backend proxy server
- Using environment variables with a build tool (Vite, Webpack)
- Restricting key usage in the [Anthropic Console](https://console.anthropic.com/)

---

## ðŸ§ª Task Spec

| Field | Value |
|---|---|
| Task ID | C5 |
| Difficulty | Moderate |
| Duration | 1 Week |
| Objective | Measure candidate speaking speed |
| Deliverable | Correct WPM calculation |

---

## ðŸ¤ Contributing

Pull requests are welcome! For major changes, please open an issue first to discuss what you'd like to change.

---

## ðŸ“„ License

[MIT](LICENSE)






Streaming vs Batch Transcription Simulator 

INTERN TASK REPORT 

T3. Streaming vs Batch Transcription Simulator 

Intern Name: Aman Kumawat 
Supervisor: vasudha 
Duration: 3 -5 Days




import whisper
import time
import json
•	from pydub import AudioSegment
import sys
import os

def split_audio(file_path, chunk_length_ms=2000):
    """2-sec chunks me audio split karta hai"""
    audio = AudioSegment.from_file(file_path)
    chunks = []
    for i in range(0, len(audio), chunk_length_ms):
        chunk = audio[i:i + chunk_length_ms]
        chunk_path = f"temp_chunk_{i//chunk_length_ms}.wav"
        chunk.export(chunk_path, format="wav")
        chunks.append(chunk_path)
    return chunks, len(audio) / 1000

def run_streaming_simulation(model, chunks):
    """Streaming mode: chunk-by-chunk transcribe + partial results"""
    print("--- STREAMING MODE START ---")
    start_time = time.time()
    full_transcript = ""
    
    for i, chunk_path in enumerate(chunks):
        chunk_start = time.time()
        result = model.transcribe(chunk_path, fp16=False)
        chunk_text = result["text"].strip()
        full_transcript += chunk_text + " "
        print(f"Chunk {i+1}: {chunk_text} | Latency: {time.time() - chunk_start:.2f}s")
        os.remove(chunk_path) # temp file delete
        
    total_time = time.time() - start_time
    print("--- STREAMING MODE END ---")
    print(f"Full Streaming Transcript: {full_transcript.strip()}\n")
    return total_time, len(chunks), full_transcript.strip()

def run_batch_simulation(model, file_path):
    """Batch mode: pura file ek saath transcribe"""
    print("--- BATCH MODE START ---")
    start_time = time.time()
    result = model.transcribe(file_path, fp16=False)
    total_time = time.time() - start_time
    transcript = result["text"].strip()
    print("--- BATCH MODE END ---")
    print(f"Full Batch Transcript: {transcript}\n")
    return total_time, transcript

def main(audio_file):
    if not os.path.exists(audio_file):
        print(f"Error: File {audio_file} not found")
        sys.exit(1)
        
    print("Loading Whisper model 'base'...")
    model = whisper.load_model("base")
    
    chunks, duration = split_audio(audio_file, 2000)
    print(f"Audio duration: {duration:.1f}s, Split into {len(chunks)} chunks\n")
    
    stream_time, num_chunks, _ = run_streaming_simulation(model, chunks)
    batch_time, _ = run_batch_simulation(model, audio_file)
    
    output = {
        "mode": "streaming",
        "chunks": num_chunks,
        "total_time_sec": round(stream_time, 1),
        "mode_batch_time_sec": round(batch_time, 1)
    }
    
    print("--- COMPARISON RESULT ---")
    print(json.dumps(output, indent=2))

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python simulator.py <audio_file_path>")
        sys.exit(1)
    main(sys.argv[1])

How to run 

python simulator.py your_audio.wav

OUTPUT 
Loading Whisper model 'base'...
Audio duration: 12.0s, Split into 6 chunks

--- STREAMING MODE START ---
Chunk 1: The quick brown | Latency: 1.42s
Chunk 2: fox jumps over | Latency: 1.35s
Chunk 3: the lazy dog | Latency: 1.51s
Chunk 4: near the bank | Latency: 1.48s
Chunk 5: of the river | Latency: 1.39s
Chunk 6:. | Latency: 1.32s
--- STREAMING MODE END ---
Full Streaming Transcript: The quick brown fox jumps over the lazy dog near the bank of the river.

--- BATCH MODE START ---
--- BATCH MODE END ---
Full Batch Transcript: The quick brown fox jumps over the lazy dog near the bank of the river.

--- COMPARISON RESULT ---
{
  "mode": "streaming",
  "chunks": 6,
  "total_time_sec": 9.8,
  "mode_batch_time_sec": 4.1
}
