# Technical Stack and Tooling

## Overview

This document outlines the technical choices for implementing the meeting transcription and summarization bot based on the specification requirements.

## Core Components

### 1. Telegram Bot

- **Framework**: aiogram (Python)
- **Language**: Python 3.10+
- **Deployment**: Docker container
- **Features**:
  - File handling (audio/video up to 50-100MB)
  - Progress message updates
  - Command handling (/enroll, /link, /privacy, /delete)
  - File size management (convert large files to documents)

### 2. API Gateway

- **Framework**: FastAPI (Python)
- **Language**: Python 3.10+
- **Features**:
  - Single entry point for all API requests
  - Authentication and authorization (OAuth/OIDC)
  - Rate limiting and throttling
  - Task queuing coordination

### 3. Task Queue System

- **Message Broker**: Redis (development) or RabbitMQ (production)
- **Task Queue**: Celery (Python)
- **Features**:
  - Asynchronous task processing
  - Retry mechanisms and idempotency support
  - Distributed workload management

### 4. Audio Processing Pipeline

- **Core Tool**: ffmpeg (CLI)
- **Audio Processing**:
  - Audio extraction from video files
  - Normalization to mono/16kHz
  - LUFS/RMS level normalization
  - Chunking for long meetings (60-120s with 0.2-0.5s overlap)
  - Noise reduction (RNNoise/ffmpeg -af arnndn)

### 5. Automatic Speech Recognition (ASR)

- **Primary Implementation**: faster-whisper (Python)
  - Based on WhisperX
  - Support for Russian/English languages
  - Accurate timecodes
- **Alternative**: NVIDIA NeMo ASR (QuartzNet/Conformer) for enterprise use
- **Deployment**:
  - Development: whisper.cpp with Metal (Mac)
  - Production: CUDA support (NVIDIA GPUs)

### 6. Speaker Diarization and Identification

- **VAD**: Silero (pyannote or torchaudio)
- **Speaker Embeddings**: ECAPA-TDNN (speechbrain/nemo)
- **Clustering**: Spectral/Agglomerative Hierarchical Clustering (sklearn)
- **Name Mapping**: Cosine similarity with threshold
- **Fallback**: NeMo diarization pipeline (if pyannote licensing issues)
- **Features**:
  - Manual mapping via /link command
  - Profile enrollment (/enroll command)
  - Confidence scoring for speaker assignments

### 7. Summarization Engine

- **Development**: LM Studio with Qwen3-X (7-14B, GGUF, Metal)
- **Production**: vLLM with Qwen3-30B (tensor parallel, PagedAttention)
- **Configuration**:
  - Context window: 16-32k tokens
  - Temperature: 0.2-0.4
  - Max new tokens: 1-2k
- **Output Formats**:
  - JSON structured summary
  - Human-readable markdown
  - SRT/VTT transcript files

### 8. Storage Systems

- **Object Storage**: MinIO (S3-compatible)
  - Store source files and artifacts
  - SSE-S3 encryption for data at rest
- **Database**: PostgreSQL
  - Metadata storage (files, meetings, speakers, user profiles)
  - Audit logging and access control

### 9. Caching System

- **Content Hashing**: SHA256 for file deduplication
- **Audio Fingerprinting**: Chromaprint or PCM mono/16k hash for content deduplication
- **Intermediate Cache**: Redis or in-memory cache for processing steps

### 10. Monitoring and Logging

- **Metrics**: Prometheus + Grafana
- **Tracing**: Sentry/Opentelemetry
- **Logging**: Structured logging with correlation IDs
- **Internal Monitoring**: Health checks and performance metrics

### 11. Access Control

- **Authentication**: Keycloak or internal OAuth/OIDC
- **Authorization**: Role-based access control (RBAC)
- **Audit Trail**: Comprehensive logging of all user actions

## Development Environment Setup

### Local Development (Mac)

- **Hardware**: Mac Mini M4 Pro, 64GB RAM
- **Tools**:
  - ffmpeg (with audio processing support)
  - whisper.cpp/faster-whisper with Metal acceleration
  - LM Studio with Qwen-7/14B (GGUF)
  - Docker Compose for local development
- **Deployment**: Docker Compose configuration

### Production Environment (Ubuntu GPU)

- **Hardware**: NVIDIA GPU servers
- **Tools**:
  - NVIDIA Container Toolkit, CUDA 12.x
  - vLLM with Qwen3-30B (tensor parallel)
  - ASR on CUDA (ctranslate2 GPU or NeMo)
  - Horizontal scaling of workers
- **Deployment**: Kubernetes or Docker Swarm

## Key Technical Considerations

### Offline Processing

- All processing happens within internal network
- No internet access required for any component
- Full data privacy by design

### Performance and Reliability

- Chunking strategy for long meetings (60-120s)
- Overlap handling in audio segmentation
- Idempotency keys for retry mechanisms
- Content-based deduplication to avoid reprocessing

### Security and Privacy

- mTLS for internal network communication
- Configurable retention policies (30 days raw files, 180 days artifacts)
- Full encryption at rest and in transit
