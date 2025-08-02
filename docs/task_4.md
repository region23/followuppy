# Task 4: Plan Telegram Bot Implementation

## Objective

Design and plan the Telegram bot component using aiogram framework with all required functionality.

## Bot Features and Commands

### Core Commands

1. **`/start`** - Welcome message and basic instructions
2. **`/help`** - Show available commands and usage instructions
3. **`/list`** - List all processed meetings with pagination
4. **`/enroll`** - Add voice profile (upload 30-60 sec sample)
5. **`/link Speaker 2 = Иван Петров`** - Manually link speaker to user
6. **`/privacy`** - Show privacy policy and data retention
7. **`/stats`** - Show user statistics
8. **`/delete <meeting_id>`** - Delete meeting artifacts

### File Handling

- Accept audio and video files
- Handle files >50MB by converting to document type
- Send progress notifications during processing:
  - "Извлекаю аудио..."
  - "Распознаю речь..."
  - "Разделяю спикеров..."
  - "Суммирую встречу..."
  - "Готово! Смотри вложения."

## Bot Architecture

### Core Components

1. **Message Handler** - Process incoming messages and commands
2. **File Processor** - Handle audio/video file uploads
3. **Command Router** - Route commands to appropriate handlers
4. **Progress Tracker** - Send progress updates during processing
5. **Result Deliverer** - Send final results with artifacts

### Bot Flow

#### File Upload Flow

1. User sends audio/video file
2. Validate file type and size
3. If >50MB, request document upload instead
4. Save file to SeaweedFS storage
5. Create meeting record in database
6. Queue processing task in Celery
7. Send initial progress message
8. Wait for processing completion
9. Deliver results with artifacts

#### Command Flow

1. User sends command
2. Parse command and arguments
3. Validate user permissions
4. Execute appropriate handler
5. Return response to user

## Implementation Details

### File Processing Logic

- Support common audio/video formats (MP3, WAV, MP4, MOV, etc.)
- Validate file size limits (50MB threshold)
- Convert large files to document type if needed
- Generate unique identifiers for files and meetings
- Store file metadata in database

### Progress Notifications

- Send progress updates every 10-30 seconds during processing
- Use Telegram's typing indicators where appropriate
- Provide clear status messages:
  - "Извлекаю аудио..." (Extracting audio)
  - "Распознаю речь..." (Recognizing speech)
  - "Разделяю спикеров..." (Identifying speakers)
  - "Суммирую встречу..." (Summarizing)
  - "Готово! Смотри вложения." (Done! See attachments)

### Result Delivery

- Send summary text as message
- Attach all artifacts:
  - `transcript.md` (with speakers and timestamps)
  - `transcript.vtt/.srt` (subtitle files)
  - `summary.json` (structured data)
- Optional: `actions.csv` / ICS with deadlines

### Error Handling

- Handle file upload failures
- Handle processing errors gracefully
- Provide user-friendly error messages
- Log errors for debugging

## Integration Points

### With API Gateway

- Submit files via API calls
- Check processing status
- Retrieve results when ready

### With Database

- Create file records
- Create meeting records
- Store speaker mappings
- Store artifacts references

### With Task Queue

- Submit processing tasks
- Monitor task progress
- Handle task completion/failure

## Technical Requirements

### aiogram Features to Use

- Message handlers for different message types
- Command handlers for bot commands
- Callback query handlers for inline buttons
- Middlewares for authentication and logging
- File handling utilities

### Security Considerations

- Validate all file uploads
- Implement rate limiting per user
- Protect against malicious files
- Handle user sessions properly

### Performance Considerations

- Efficient message processing
- Asynchronous file handling where possible
- Proper error recovery
- Memory usage optimization

## Implementation Steps

1. Set up aiogram bot instance with proper configuration
2. Implement message handlers for different content types
3. Create command handlers for all required commands
4. Implement file upload and validation logic
5. Add progress notification system
6. Integrate with API gateway for task submission
7. Implement result delivery mechanism
8. Add error handling and logging
9. Test bot functionality end-to-end
