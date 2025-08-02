# Task 2: Create High-Level System Design

## Objective

Design the overall architecture and component interactions for the Telegram meeting transcription and summarization system.

## System Architecture Overview

### Components and Their Responsibilities

1. **Telegram Bot (aiogram)**
   - Receives audio/video files from users
   - Handles bot commands (/list, /enroll, /link, etc.)
   - Sends progress notifications during processing
   - Provides results to users

2. **API Gateway (FastAPI)**
   - Single entry point for all API requests
   - Authentication and authorization using Telegram Passport
   - Rate limiting to prevent abuse
   - Task submission and status checking endpoints

3. **Task Queue (Celery + Redis)**
   - Manages asynchronous processing tasks
   - Distributes work across multiple workers
   - Handles task retries and idempotency
   - Stores task states and results

4. **Video Processing Pipeline**
   - Uses ffmpeg for audio extraction
   - Normalizes audio to mono/16kHz
   - Applies audio normalization (RMS/LUFS)

5. **ASR System**
   - Processes audio to text with timecodes
   - Supports WhisperX, faster-whisper, and NeMo ASR
   - Language detection and model selection

6. **Speaker Diarization**
   - VAD for speech segmentation
   - Speaker embedding extraction (ECAPA-TDNN)
   - Clustering of speakers (spectral/ahc)
   - Mapping to user profiles

7. **Summarization Engine**
   - LLM-based summarization using Qwen3-30B
   - Uses vLLM in production, LM Studio in dev
   - Follows specific prompt structure for structured output

8. **Storage Layer**
   - SeaweedFS for media file storage (S3-compatible)
   - PostgreSQL for metadata and artifacts
   - Caching layer for intermediate results

9. **Caching/Deduplication**
   - File hash (sha256) checking
   - Content fingerprinting for identical content in different containers
   - Cache invalidation strategy

10. **Monitoring**
    - Sentry/Opentelemetry for tracing and error tracking
    - Performance metrics collection

11. **Access Control**
    - Telegram Passport integration
    - Role-based access control (RBAC)

## Data Flow Diagram

```
[User] → [Telegram Bot] → [API Gateway] → [Task Queue]
                              ↓
                    [Storage: SeaweedFS]
                              ↓
                    [Video Processing Pipeline]
                              ↓
                         [ASR System]
                              ↓
                   [Speaker Diarization]
                              ↓
                     [Summarization Engine]
                              ↓
                    [Storage: PostgreSQL]
                              ↓
                    [Caching Layer]
                              ↓
                    [Telegram Bot] → [User]
```

## Component Interactions

1. **Telegram Bot** receives files and forwards to API Gateway
2. **API Gateway** validates requests, stores files in SeaweedFS, and queues tasks
3. **Task Queue** distributes processing jobs to workers
4. **Video Processing Pipeline** extracts audio and normalizes it
5. **ASR System** converts audio to text with timecodes
6. **Speaker Diarization** identifies speakers and maps to profiles
7. **Summarization Engine** creates structured summary from transcript
8. **Storage Layer** persists all artifacts and metadata
9. **Caching Layer** prevents reprocessing of identical content
10. **Telegram Bot** delivers results back to user

## Technology Stack Mapping

| Component | Technology |
|-----------|------------|
| Telegram Bot | Python, aiogram |
| API Gateway | FastAPI |
| Task Queue | Celery + Redis |
| Video Processing | ffmpeg |
| ASR | WhisperX/faster-whisper/NeMo |
| Diarization | Silero VAD, ECAPA-TDNN, pyannote/NeMo |
| Summarization | Qwen3-30B via vLLM/LM Studio |
| Storage | SeaweedFS (S3), PostgreSQL |
| Caching | Redis |
| Monitoring | Sentry/Opentelemetry |
| Access Control | Telegram Passport |

## Deployment Architecture

### Development Environment

- Mac Mini M4 Pro with 64GB RAM
- ffmpeg, whisper.cpp/faster-whisper with Metal support
- LM Studio with Qwen-7/14B (GGUF) for prototyping
- Docker Compose for local development

### Production Environment

- GPU server with Ubuntu and CUDA 12.x
- NVIDIA Container Toolkit
- vLLM with Qwen3-30B (tensor parallelism)
- ASR on CUDA (ctranslate2 GPU or NeMo)
- Horizontal scaling of ASR workers, separate LLM processing

## Security Considerations

- All processing happens offline
- AES256 encryption for files at rest
- TLS for data in transit
- RBAC for access control
- No external network calls during processing
