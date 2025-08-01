# MVP Development Phases

## Overview

This document outlines the phased development approach for building the meeting transcription and summarization Telegram bot, based on the specification's MVP roadmap.

## Phase 1: Core Audio Processing and Basic ASR

**Timeline**: 1-2 weeks
**Focus**: Audio extraction, basic transcription, and minimal speaker identification

### Key Deliverables

- Video to audio conversion using ffmpeg
- Whisper-based ASR with timecodes
- Basic speaker diarization (clustered but not named)
- File deduplication by SHA256
- Telegram bot with core functionality

### Technical Components

1. ffmpeg pipeline for audio extraction and normalization
2. faster-whisper implementation (with Metal support on Mac)
3. Basic speaker clustering without name mapping
4. SHA256-based file deduplication
5. Telegram bot with aiogram framework

### User Features

- Send audio/video files to bot
- Progress messages during processing
- Receive basic transcript and summary

## Phase 2: Enhanced Summarization and Output Formats

**Timeline**: 1-2 weeks
**Focus**: Improved summarization, better output formats, and content-based deduplication

### Key Deliverables

- Qwen3-7/14B summarization engine (LM Studio)
- Markdown report generation
- SRT/VTT transcript formats
- Content hash-based deduplication
- Enhanced Telegram bot UI

### Technical Components

1. LM Studio integration for Qwen summarization
2. JSON and human-readable summary generation
3. Content hash calculation (PCM mono/16kHz)
4. Enhanced file deduplication logic
5. Improved Telegram bot with better UX

### User Features

- Enhanced summary output (JSON + human-readable)
- Transcript in multiple formats (SRT, VTT, MD)
- Content-based file deduplication
- Better progress reporting

## Phase 3: User Profile Management and Manual Mapping

**Timeline**: 1 week
**Focus**: Speaker name mapping, user enrollment, and manual assignment

### Key Deliverables

- User profile enrollment (/enroll command)
- Manual speaker name mapping (/link command)
- Improved speaker identification accuracy
- Enhanced Telegram bot with user management

### Technical Components

1. User profile database (PostgreSQL)
2. Speaker name mapping system
3. Manual assignment via Telegram commands
4. Enhanced Telegram bot with user management features

### User Features

- /enroll command for voice profile creation
- /link command for manual speaker mapping
- Improved speaker identification accuracy

## Phase 4: Production-Ready Summarization and Advanced Diarization

**Timeline**: 2 weeks
**Focus**: Production-grade summarization, improved speaker identification, and GPU support

### Key Deliverables

- vLLM integration with Qwen3-30B
- Improved speaker diarization pipeline
- GPU support for ASR and LLM processing
- Horizontal scaling capabilities

### Technical Components

1. vLLM integration for Qwen3-30B
2. Enhanced speaker diarization with ECAPA-TDNN embeddings
3. CUDA support for GPU acceleration
4. Horizontal worker scaling capabilities

### User Features

- Production-quality summarization
- Improved speaker identification accuracy
- Better performance with large files

## Phase 5: Complete System with Monitoring and Security

**Timeline**: 1-2 weeks
**Focus**: Full system integration, monitoring, security, and deployment

### Key Deliverables

- Complete API gateway with authentication
- Full monitoring and logging system
- Security enhancements (mTLS, encryption)
- Production deployment configuration

### Technical Components

1. FastAPI-based API gateway
2. Prometheus + Grafana monitoring
3. Sentry/Opentelemetry tracing
4. Security and privacy features (encryption, mTLS)
5. Docker/Kubernetes deployment configuration

### User Features

- Secure and private processing
- Full audit trail and access control
- Production-ready deployment

## Development Approach

### Development Environment

- Local development on Mac (M4 Pro with 64GB RAM)
- Docker Compose for local testing
- ffmpeg, whisper.cpp/faster-whisper with Metal support

### Production Environment

- GPU servers (Ubuntu with NVIDIA GPUs)
- NVIDIA Container Toolkit and CUDA 12.x
- vLLM with Qwen3-30B (tensor parallel)
- ASR on CUDA (ctranslate2 GPU or NeMo)

### Integration Points

1. Telegram bot communicates with API gateway
2. API gateway queues tasks via Celery/Redis
3. Audio processing pipeline uses ffmpeg
4. ASR system with faster-whisper or NeMo
5. Speaker diarization with Silero + ECAPA-TDNN
6. Summarization engine with Qwen3-30B via vLLM/LM Studio
7. Storage systems (MinIO, PostgreSQL)
8. Monitoring and logging (Prometheus, Grafana, Sentry)

### Testing Strategy

- Unit tests for each component
- Integration tests for data flow
- Performance testing with large files
- Security and privacy validation
- End-to-end user experience testing

### Quality Assurance

- Idempotency support for retry mechanisms
- Content-based deduplication to avoid reprocessing
- Configurable retention policies
- Full offline processing capability
