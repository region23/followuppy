# Task 3: Define Database Schema

## Objective

Design the complete database schema for storing meeting metadata, user profiles, artifacts, and cache data.

## Database Design Overview

The system requires a PostgreSQL database to store:

- File metadata and references
- Meeting information and status
- Speaker identification and mapping
- User profiles with voice embeddings
- Artifacts (transcripts, summaries, etc.)
- Cache for intermediate processing results

## Tables Structure

### 1. Files Table

```sql
files (
    id UUID PRIMARY KEY,
    sha256 VARCHAR(64) NOT NULL,           -- SHA256 hash of original file
    content_fp VARCHAR(128),              -- Content fingerprint (PCM mono/16k)
    mime VARCHAR(100),                    -- MIME type of the file
    duration INTEGER,                     -- Duration in seconds
    size BIGINT,                          -- File size in bytes
    owner UUID,                           -- Owner user ID
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
)
```

### 2. Meetings Table

```sql
meetings (
    id UUID PRIMARY KEY,
    file_id UUID REFERENCES files(id),
    status VARCHAR(50) NOT NULL,          -- Processing status (pending, processing, completed, failed)
    language VARCHAR(10),                 -- Detected language
    started_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    finished_at TIMESTAMP WITH TIME ZONE  -- When processing was completed
)
```

### 3. Speakers Table

```sql
speakers (
    id UUID PRIMARY KEY,
    meeting_id UUID REFERENCES meetings(id),
    cluster_id VARCHAR(50),               -- Cluster identifier from diarization
    user_profile_id UUID,                 -- Reference to user profile (can be NULL)
    confidence FLOAT                      -- Confidence score of mapping
)
```

### 4. User Profiles Table

```sql
user_profiles (
    id UUID PRIMARY KEY,
    display_name VARCHAR(255) NOT NULL,   -- User's display name
    embeddings JSONB,                     -- Voice embeddings (array of floats)
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
)
```

### 5. Artifacts Table

```sql
artifacts (
    id UUID PRIMARY KEY,
    meeting_id UUID REFERENCES meetings(id),
    type VARCHAR(50) NOT NULL,            -- Type: transcript.md, transcript.vtt, transcript.srt, summary.json
    s3_key VARCHAR(500),                  -- S3 key for the artifact
    meta JSONB                            -- Metadata about the artifact
)
```

### 6. Cache Table

```sql
cache (
    key VARCHAR(255) PRIMARY KEY,
    value TEXT NOT NULL,                  -- Cached data (JSON or serialized object)
    ttl TIMESTAMP WITH TIME ZONE          -- Time to live for cache entry
)
```

## Relationships and Constraints

1. **Files → Meetings**: One-to-many relationship (one file can have one meeting)
2. **Meetings → Speakers**: One-to-many relationship (one meeting has many speakers)
3. **Speakers → User Profiles**: Many-to-one relationship (speaker maps to user profile)
4. **Meetings → Artifacts**: One-to-many relationship (one meeting has multiple artifacts)
5. **User Profiles**: Self-contained, stores voice embeddings for speaker identification

## Indexes and Performance Considerations

### Primary Indexes

- `files.id`, `meetings.id`, `speakers.id`, `user_profiles.id`, `artifacts.id`
- `files.sha256` (for deduplication)
- `files.content_fp` (for content-based deduplication)

### Additional Useful Indexes

- `meetings.file_id` (for file-based queries)
- `meetings.status` (for status filtering)
- `speakers.meeting_id` (for speaker queries by meeting)
- `speakers.user_profile_id` (for user profile lookups)
- `artifacts.meeting_id` (for artifact retrieval by meeting)

## Data Flow and Usage Patterns

### File Processing Workflow

1. When a file is uploaded, create entry in `files` table
2. Create corresponding entry in `meetings` table with status "pending"
3. After processing:
   - Update `meetings.status` to "completed" or "failed"
   - Store speaker mappings in `speakers` table
   - Store artifacts in `artifacts` table
   - Cache intermediate results in `cache` table

### Speaker Identification Workflow

1. During diarization, create speaker clusters
2. Match clusters to user profiles using cosine similarity
3. Store mapping in `speakers` table with confidence score
4. If no match found, leave `user_profile_id` as NULL for manual assignment

### Caching Strategy

- Cache intermediate processing results by content fingerprint (`content_fp`)
- Cache completed meeting results for quick retrieval
- Set TTL for cache entries to prevent stale data

## Security Considerations

1. All tables should have appropriate access controls
2. User profile data should be protected with proper RBAC
3. File references should be validated before access
4. Timestamps should be stored in UTC for consistency

## Migration Strategy

1. Create all tables with proper constraints and indexes
2. Implement seed data for initial user profiles if needed
3. Set up proper backup and restore procedures
4. Plan for schema evolution as requirements change
