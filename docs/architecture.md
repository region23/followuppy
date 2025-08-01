# Meeting Transcription and Summarization Bot - Architecture

## Overview

This document describes the high-level architecture for a Telegram bot that processes audio/video files to generate transcriptions, speaker diarization, and structured summaries.

## System Components

### 1. Telegram Bot (aiogram)

- Accepts audio/video files from users
- Handles user commands (/enroll, /link, /privacy, /delete)
- Manages progress messages
- Communicates with API gateway

### 2. API Gateway (FastAPI)

- Single entry point for all requests
- Authentication and authorization
- Rate limiting and throttling
- Task queuing coordination

### 3. Task Queue (Celery + Redis/RabbitMQ)

- Asynchronous task processing
- Distributed workload management
- Retry mechanisms and idempotency support

### 4. Audio Processing Pipeline (ffmpeg)

- Extracts audio from video files
- Normalizes audio to mono/16kHz
- Audio level normalization (RMS/LUFS)
- Chunking for long meetings

### 5. Automatic Speech Recognition (ASR)

- WhisperX/faster-whisper for accurate timecodes
- Language auto-detection
- Support for Russian/English languages

### 6. Speaker Diarization and Identification

- Voice Activity Detection (VAD) with Silero
- Speaker embedding extraction (ECAPA-TDNN)
- Clustering and speaker name mapping
- Manual mapping via /link command

### 7. Summarization Engine (Qwen3-30B)

- Structured summary generation via vLLM/LM Studio
- JSON and human-readable output formats
- Custom prompt engineering for meeting summaries

### 8. Storage Systems

- MinIO (S3-compatible) for source files and artifacts
- PostgreSQL for metadata management

### 9. Caching System

- Content hash-based deduplication
- Cache for intermediate processing steps

### 10. Monitoring and Logging

- Prometheus + Grafana for metrics
- Sentry/Opentelemetry for tracing
- Internal logging and audit trails

### 11. Access Control

- Keycloak or internal OAuth/OIDC
- Role-based access control (RBAC)
- Audit logging

## Data Flow

1. User sends audio/video to Telegram bot
2. Bot saves file to MinIO and queues processing task
3. Task is picked up by worker in queue
4. Audio is processed through ffmpeg pipeline
5. ASR extracts text with timecodes
6. Speaker diarization identifies speakers
7. Summarization engine generates structured output
8. Results are stored in MinIO and PostgreSQL
9. Bot returns summary and files to user

## Deployment Architecture

### Development (Mac)

- ffmpeg, whisper.cpp/faster-whisper with Metal support
- LM Studio with Qwen-7/14B (GGUF) for prototyping
- Docker Compose for local development

### Production (GPU Ubuntu)

- NVIDIA Container Toolkit, CUDA 12.x
- vLLM with Qwen3-30B (tensor parallel)
- ASR on CUDA (ctranslate2 GPU or NeMo)
- Horizontal scaling of workers

## Security and Privacy

- Full offline processing (no internet access)
- SSE-S3 encryption in MinIO
- mTLS for internal network communication
- Configurable retention policies
