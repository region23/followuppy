# Task 1: Core Telegram Bot Implementation

## Overview

Implement the Telegram bot component that handles user file uploads, processes commands, and communicates with the API gateway for task queuing.

## Requirements

- Accept audio/video files from users via Telegram
- Handle user commands: /enroll, /link, /privacy, /delete
- Send progress messages during processing
- Support file size management (convert large files to documents)
- Communicate with the API gateway for task queuing

## Technical Details

### Framework and Dependencies

- Python 3.10+
- aiogram 3.x (Telegram Bot framework)
- FastAPI client for API communication
- Docker support for deployment

### Core Features

1. **File Handling**
   - Accept audio and video files up to 50-100MB
   - Convert large files to document type (not audio)
   - Validate file types and sizes
   - Handle file upload progress notifications

2. **Command Processing**
   - `/enroll` - Add voice profile (30-60 second sample)
   - `/link Speaker 2 = Иван Петров` - Manual speaker mapping
   - `/privacy` - Show privacy policy and retention times
   - `/delete <meeting_id>` - Delete artifacts

3. **Progress Reporting**
   - Send progress messages during processing stages:
     - "Extracting audio from video..."
     - "Recognizing speech..."
     - "Generating summary..."
   - Update messages with current status

4. **API Integration**
   - Communicate with API gateway to queue processing tasks
   - Handle responses from the processing pipeline
   - Return results to users in appropriate formats

### Implementation Plan

#### Phase 1: Basic Bot Setup

- Initialize aiogram bot with proper configuration
- Set up command handlers for all required commands
- Implement basic file receiving functionality
- Create progress message system

#### Phase 2: API Integration

- Implement communication with FastAPI gateway
- Create task queuing mechanism
- Handle responses from processing pipeline
- Implement result delivery to users

#### Phase 3: User Experience Enhancements

- Add proper error handling and user feedback
- Implement file size management logic
- Add progress tracking with status updates
- Test with various file types and sizes

### File Structure

```
telegram_bot/
├── bot.py                 # Main bot initialization and configuration
├── handlers/              # Command and message handlers
│   ├── commands.py        # Command handlers (/enroll, /link, etc.)
│   ├── files.py           # File upload and processing handlers
│   └── progress.py        # Progress message handling
├── services/              # Service layer for API communication
│   ├── api_client.py      # FastAPI client implementation
│   └── task_queue.py      # Task queuing logic
├── models/                # Data models for bot operations
│   ├── file_info.py       # File metadata models
│   └── user_commands.py   # Command data models
└── config/                # Configuration files
    └── bot_config.py      # Bot configuration settings
```

### Key Implementation Details

#### File Size Management

- Files larger than 50-100MB should be converted to document type
- Implement size validation and conversion logic
- Provide user feedback about file size limitations

#### Progress Messages

- Send periodic progress updates during processing
- Use Telegram's message editing to update status
- Implement proper timing for progress updates

#### Error Handling

- Handle file upload failures gracefully
- Provide clear error messages to users
- Implement retry logic where appropriate

### Integration Points

1. **API Gateway**: Communicate with FastAPI endpoint to queue tasks
2. **Storage**: Receive file IDs or URLs from API for processing
3. **User Profiles**: Handle /enroll command to create voice profiles

### Testing Requirements

- Test with various file types (MP3, WAV, MP4, MOV)
- Verify progress message updates
- Test command handling with different inputs
- Validate file size management logic

### Success Criteria

- Bot successfully accepts audio/video files
- All commands are properly handled
- Progress messages are sent during processing
- Results are returned to users after processing
