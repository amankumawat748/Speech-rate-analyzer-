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