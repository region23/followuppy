# Task 1: Analyze Project Requirements and Architecture

## Objective

Understand the complete project requirements, architecture, and technical specifications for the Telegram meeting transcription and summarization system.

## Description

This task involves thoroughly analyzing the project specification document to extract all requirements, architectural components, workflows, and technical details needed for implementation.

## Key Requirements Identified

- Telegram bot for audio/video processing with aiogram framework
- Offline processing with no internet connectivity required
- Multi-step pipeline: audio extraction → ASR → speaker diarization → summarization
- Support for Russian and English languages
- Caching/deduplication using file hashes and content fingerprints
- Storage in SeaweedFS (S3-compatible) and PostgreSQL
- Telegram Passport for access control
- Progress notifications during processing
- Support for large files (>50MB) via document upload

## Components to Implement

1. Telegram Bot (Python, aiogram)
2. API Gateway (FastAPI)
3. Task Queue (Celery + Redis)
4. Video Processing Pipeline (ffmpeg)
5. ASR System (WhisperX/faster-whisper)
6. Speaker Diarization (VAD + ECAPA-TDNN + clustering)
7. Summarization (Qwen3-30B via vLLM/LM Studio)
8. Storage Layer (SeaweedFS + PostgreSQL)
9. Caching/Deduplication
10. Monitoring (Sentry/Opentelemetry)
11. Access Control (Telegram Passport)

## Technical Specifications

- Dev environment: Mac Mini M4 Pro with 64GB RAM
- Production: GPU server with Ubuntu and CUDA
- ASR models: WhisperX, faster-whisper, NeMo ASR
- Diarization models: Silero VAD, ECAPA-TDNN, pyannote (with fallback to NeMo)
- LLM: Qwen3-30B via vLLM in production, LM Studio in dev
- File processing: 60-120s chunks with 0.2-0.5s overlap
- Audio normalization: RMS/LUFS target -23
- Storage: SeaweedFS for media files, PostgreSQL for metadata

## Deliverables

- Complete understanding of system architecture
- List of all required components and their interactions
- Technical stack mapping
- Implementation priority matrix
