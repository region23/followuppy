# Task 2: Telegram Bot Implementation with aiogram

## Objective

Implement the Telegram bot using aiogram framework to handle file uploads, status updates, and result delivery.

## Requirements

- Accept audio/video file uploads via Telegram
- Handle files larger than 50MB appropriately
- Provide progress updates during processing
- Deliver final results with summary and transcripts
- Implement all required bot commands

## Implementation Details

1. **Bot Framework Setup**:
   - Configure aiogram with proper error handling
   - Set up message handlers for different file types
   - Implement rate limiting to handle Telegram HTTP timeouts

2. **File Handling**:
   - Process audio/video files with proper validation
   - Handle files >50MB by converting to document type
   - Implement file size validation and error handling

3. **Command Implementation**:
   - `/list` - Show all meeting summaries with pagination
   - `/enroll` - Add voice profile (30-60 sec sample)
   - `/link Speaker 2 = Иван Петров` - Manual speaker mapping
   - `/privacy` - Show storage and retention policy
   - `/stats` - Show user statistics
   - `/help` - Show command help
   - `/delete <meeting_id>` - Delete artifacts

4. **Progress Reporting**:
   - Send progress messages during processing stages
   - Implement status updates for each pipeline step
   - Handle long-running operations gracefully

5. **Integration Points**:
   - Connect to API gateway for task submission
   - Handle result delivery back to Telegram
   - Implement proper error reporting

## Deliverables

- Complete Telegram bot implementation with aiogram
- All command handlers implemented
- Progress reporting system
- File handling and validation logic
