# Task 12: Plan Caching and Deduplication Implementation

## Objective

Design and plan the caching and deduplication system that prevents reprocessing of identical content and improves system performance.

## Caching and Deduplication Overview

The system implements both file-level deduplication and content-based caching to:

- Prevent reprocessing of identical files
- Improve response times for repeated content
- Reduce computational overhead
- Optimize storage usage through content reuse

## Deduplication Strategies

### 1. File Hash Deduplication

- **SHA256 hash** of original file as primary identifier
- Check against existing files in database before processing
- Immediate return of cached results when match found

### 2. Content Fingerprinting

- **Content fingerprint** after audio decoding (PCM mono/16kHz, fixed gain)
- **Chromaprint** acoustic fingerprinting as alternative
- Covers cases where same content exists in different containers/bitrates

## Caching Architecture

### Cache Layers

1. **Database Cache** - Store processed results with TTL
2. **Redis Cache** - Fast access for intermediate processing data
3. **File System Cache** - Local caching for frequently accessed artifacts

### Cache Keys and Structure

```
Cache Key Format:
- File-level: sha256:{file_sha256}
- Content-level: content_fp:{content_fingerprint}
- Processing-step: step:{meeting_id}:{step_name}
- Artifact: artifact:{meeting_id}:{artifact_type}
```

## Implementation Details

### Deduplication Process Flow

```
[File Upload] → [Hash Calculation] → [Database Check] → [Content Fingerprint] → [Cache Check] → [Process/Return]
     ↓              ↓              ↓              ↓              ↓              ↓
   Validate      Calculate      Check          Calculate      Check         Process
   File           SHA256        Existing       Content        Cached        or Return
   Hash           Hash         Files          Fingerprint    Results       Results
```

### Detailed Deduplication Steps

#### Step 1: File Hash Calculation

```python
def calculate_file_hash(file_path: str) -> str:
    """
    Calculate SHA256 hash of the original file
    Returns: Hex string of SHA256 hash
    """
```

#### Step 2: Database Deduplication Check

```python
def check_file_deduplication(file_sha256: str) -> Optional[Dict]:
    """
    Check if file with same SHA256 already exists in database
    Returns: Meeting record if found, None otherwise
    """
```

#### Step 3: Content Fingerprint Calculation

```python
def calculate_content_fingerprint(audio_path: str) -> str:
    """
    Calculate content fingerprint after audio normalization
    Returns: Hex string of content fingerprint (PCM mono/16kHz)
    """
```

#### Step 4: Content-based Cache Check

```python
def check_content_cache(content_fp: str) -> Optional[Dict]:
    """
    Check if content with same fingerprint already processed
    Returns: Cached results if found, None otherwise
    """
```

## Cache Management

### Cache Types and Purging

#### Database Cache

- Store complete meeting results with TTL (e.g., 7 days)
- Key format: `cache:meeting:{meeting_id}`
- Automatic cleanup based on TTL values

#### Redis Cache

- Store intermediate processing data
- Short TTL for temporary results (e.g., 1 hour)
- Key format: `cache:intermediate:{key}`

#### Artifact Cache

- Store generated artifacts with metadata
- Long TTL for frequently accessed content
- Key format: `cache:artifact:{meeting_id}:{type}`

### Cache Invalidation Strategy

1. **Automatic**: Based on TTL values
2. **Manual**: When user deletes meeting or updates profiles
3. **Event-based**: When system detects content changes
4. **Periodic**: Regular cleanup of expired entries

## Performance Optimization

### Cache Hit Strategies

- Prioritize cache lookups before processing
- Implement parallel cache checks where possible
- Use efficient data structures for fast lookups
- Pre-warm cache with frequently accessed content

### Memory Management

- Monitor Redis memory usage
- Implement LRU eviction policies
- Cache only essential data to avoid memory pressure
- Optimize cache key sizes for efficiency

## Integration with Processing Pipeline

### Before Processing

1. Check file hash in database
2. If found, return cached results immediately
3. If not found, proceed to content fingerprinting
4. Check content fingerprint in cache
5. If found, return cached processing results
6. If not found, start full processing pipeline

### During Processing

1. Cache intermediate results for recovery
2. Store partial results if processing is interrupted
3. Update cache with final results upon completion
4. Handle errors by cleaning up cache entries

### After Processing

1. Store complete meeting results in database cache
2. Create artifact references in database
3. Update cache with all generated artifacts
4. Set appropriate TTL values for different cache types

## Error Handling and Recovery

### Cache Failure Scenarios

1. **Cache Unavailable**: Fall back to full processing
2. **Cache Inconsistency**: Validate cached data before use
3. **Cache Corruption**: Remove invalid entries and reprocess
4. **Cache Exhaustion**: Implement proper eviction policies

### Deduplication Error Handling

1. Handle hash calculation failures gracefully
2. Manage database connection issues during checks
3. Provide fallback mechanisms for cache misses
4. Log deduplication attempts for monitoring

## Monitoring and Metrics

### Cache Performance Metrics

- Cache hit ratio (successful vs total requests)
- Cache miss rate
- Average response time with/without cache
- Memory usage statistics
- Cache eviction rates

### Deduplication Metrics

- Deduplication rate (files processed vs deduplicated)
- Time saved through deduplication
- Storage space saved
- System resource utilization improvements

## Implementation Steps

1. Implement file hash calculation utilities
2. Create database deduplication check functions
3. Develop content fingerprinting system
4. Set up Redis cache infrastructure
5. Implement cache storage and retrieval mechanisms
6. Add cache invalidation and cleanup procedures
7. Integrate deduplication checks into processing pipeline
8. Add monitoring and metrics collection
9. Test deduplication scenarios with various inputs
10. Optimize cache performance based on usage patterns
11. Document cache management policies and procedures
12. Implement error handling for cache failures

## Security Considerations

### Cache Data Protection

- Ensure sensitive data is not cached inappropriately
- Implement proper access controls for cached content
- Encrypt cache entries where necessary
- Regular security audits of caching mechanisms

### Deduplication Security

- Prevent malicious users from exploiting deduplication
- Validate file integrity before caching
- Handle edge cases in hash comparisons securely
- Monitor for unusual deduplication patterns that might indicate abuse
