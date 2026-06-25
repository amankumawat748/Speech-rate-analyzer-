







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
