# Task 6: Plan Task Queue Implementation

## Objective

Design and plan the Celery-based task queue system for managing asynchronous processing jobs.

## Task Queue Overview

The system uses Celery with Redis as the message broker to manage asynchronous processing tasks. This allows for:

- Distributing work across multiple workers
- Handling long-running processing operations
- Implementing retries and idempotency
- Monitoring task progress and status

## Core Tasks

### 1. File Processing Task

```python
def process_file_task(file_id: str, meeting_id: str):
    """
    Main processing pipeline for a file:
    - Extract audio using ffmpeg
    - Normalize audio
    - Run ASR
    - Perform speaker diarization
    - Generate summary
    - Store artifacts
    - Update database status
    """
```

### 2. Deduplication Check Task

```python
def check_deduplication_task(file_path: str, content_fp: str):
    """
    Check if file or content already exists in system
    Return cached results if found
    """
```

### 3. Audio Processing Task

```python
def process_audio_task(file_path: str, output_path: str):
    """
    Extract and normalize audio using ffmpeg:
    - Extract audio from video
    - Convert to mono/16kHz
    - Apply RMS/LUFS normalization
    """
```

### 4. ASR Task

```python
def asr_task(audio_path: str, language: str):
    """
    Run automatic speech recognition on audio file:
    - Use WhisperX/faster-whisper/NeMo
    - Generate text with timecodes
    """
```

### 5. Speaker Diarization Task

```python
def diarize_speakers_task(audio_path: str, asr_result: dict):
    """
    Identify and separate speakers:
    - VAD segmentation
    - Speaker embedding extraction (ECAPA-TDNN)
    - Clustering of speakers
    - Mapping to user profiles
    """
```

### 6. Summarization Task

```python
def summarize_task(transcript: str, meeting_id: str):
    """
    Generate structured summary using LLM:
    - Use Qwen3-30B via vLLM/LM Studio
    - Follow specific prompt structure
    - Return JSON and human-readable version
    """
```

### 7. Artifact Generation Task

```python
def generate_artifacts_task(meeting_id: str, summary_data: dict, transcript_data: dict):
    """
    Generate all output artifacts:
    - transcript.md (with speakers and timestamps)
    - transcript.vtt/.srt (subtitle files)
    - summary.json (structured data)
    - Optional: actions.csv/ICS
    """
```

## Task Flow Architecture

### Processing Pipeline

```
[File Upload] → [API Gateway] → [Task Queue] → [Worker Process]
                              ↓
                    [Deduplication Check]
                              ↓
                    [Audio Processing]
                              ↓
                    [ASR Processing]
                              ↓
                    [Speaker Diarization]
                              ↓
                    [Summarization]
                              ↓
                    [Artifact Generation]
                              ↓
                    [Storage & Database Update]
```

## Task Management Features

### 1. Idempotency

- Use content fingerprint (`content_fp`) as idempotency key
- Prevent reprocessing of identical content
- Handle task retries gracefully

### 2. Retries and Error Handling

- Automatic retry on failures with exponential backoff
- Maximum retry limits to prevent infinite loops
- Error logging and notification system
- Task failure cleanup procedures

### 3. Progress Tracking

- Store intermediate progress in database
- Update task status during processing
- Allow API gateway to check progress

### 4. Task Prioritization

- High priority for user-facing tasks
- Lower priority for background maintenance
- Queue management based on system load

## Redis Integration

### Configuration

- Redis as message broker for Celery
- Redis database for caching intermediate results
- Redis for task result storage

### Key Patterns

- Task queue management using Redis lists
- Result storage with TTL for temporary data
- Cache invalidation strategies

## Worker Architecture

### Worker Types

1. **ASR Workers** - Handle speech recognition tasks
2. **Diarization Workers** - Handle speaker identification
3. **Summarization Workers** - Handle LLM processing
4. **Storage Workers** - Handle file storage operations

### Scaling Considerations

- Horizontal scaling of workers based on load
- GPU workers for ASR and summarization tasks
- CPU workers for preprocessing and postprocessing
- Load balancing between worker types

## Task Serialization and Data Flow

### Input/Output Formats

- All task data should be serializable (JSON, Pickle)
- Use UUIDs for all identifiers to ensure uniqueness
- Store intermediate results in cache with TTL
- Pass minimal required data between tasks

### Data Validation

- Validate input parameters before task execution
- Check file integrity and format
- Verify database consistency
- Handle missing or corrupted data gracefully

## Monitoring and Management

### Task Monitoring

- Track task execution time and performance
- Monitor queue length and worker status
- Log task success/failure rates
- Generate metrics for system optimization

### Health Checks

- Worker health monitoring
- Queue health checks
- Database connection status
- External service availability (ffmpeg, ASR models)

## Implementation Steps

1. Set up Celery configuration with Redis broker
2. Define all required task functions with proper signatures
3. Implement task routing and worker assignment
4. Add idempotency and retry mechanisms
5. Create progress tracking system
6. Implement error handling and logging
7. Set up monitoring and metrics collection
8. Test task queue functionality end-to-end
9. Configure scaling strategies for different worker types
10. Document task interfaces and expected behavior
