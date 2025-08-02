# Task 10: Plan Summarization Implementation

## Objective

Design and plan the LLM-based summarization system that creates structured meeting summaries from transcribed audio.

## Summarization Overview

The summarization system uses Qwen3-30B (via vLLM in production, LM Studio in dev) to generate structured summaries from meeting transcripts. The system follows a specific prompt structure to ensure consistent output format.

## Core Requirements

### Output Structure

The system must produce both JSON and human-readable summaries following this structure:

```json
{
  "summary": "краткое резюме в 5-8 предложений",
  "decisions": [{"who": "Имя", "what": "Решение", "when": "дата/срок или null"}],
  "action_items": [{"assignee":"Имя","task":"...", "due":"YYYY-MM-DD|null", "priority":"low|med|high"}],
  "risks": ["..."],
  "open_questions": ["..."],
  "timeline": [{"t":"00:12:05","topic":"..."}],
  "participants": [{"name":"Имя","speakers":["S1","S3"]}]
}
```

### Prompt Engineering

The system uses a carefully crafted prompt that:

- Instructs the LLM to be concise and focused on key outcomes
- Specifies the required JSON structure
- Provides rules for speaker mapping and timecode references
- Ensures proper handling of edge cases

## Implementation Architecture

### Core Components

#### 1. Prompt Engineering Module

```python
def generate_summarization_prompt(transcript: str, language: str) -> str:
    """
    Generate the complete prompt for summarization including system and user parts
    Returns: Complete prompt string ready for LLM input
    """
```

#### 2. LLM Interface Module

```python
def run_summarization(prompt: str, max_new_tokens: int = 2000) -> Dict[str, Any]:
    """
    Run summarization using Qwen3-30B via vLLM/LM Studio
    Returns: Structured summary data (JSON + text)
    """
```

#### 3. Output Processing Module

```python
def process_summarization_output(raw_output: str) -> Dict[str, Any]:
    """
    Parse and validate LLM output into structured format
    Returns: Clean, validated summary structure
    """
```

### Data Flow Pipeline

```
[Transcript Input] → [Prompt Engineering] → [LLM Processing] → [Output Validation] → [Structured Result]
     ↓              ↓                ↓               ↓              ↓
   Transcript    Generate         Run LLM        Parse &       Final
   with          Prompt           Summarization   Validate      Output
   Speakers      (System + User)  with Qwen3-30B  JSON          Format
```

## Prompt Design

### System Prompt

```
Ты помощник по протоколированию встреч. На входе — хронологический транскрипт с таймкодами и спикерами.
Задача: структурированно и кратко оформить итоги. Пиши на русском, делай выводы, не пересказывай лишнее.
```

### User Prompt Template

```
Дай JSON и краткий текст.
JSON-структура:
{
  "summary": "краткое резюме в 5-8 предложений",
  "decisions": [{"who": "Имя", "what": "Решение", "when": "дата/срок или null"}],
  "action_items": [{"assignee":"Имя","task":"...", "due":"YYYY-MM-DD|null", "priority":"low|med|high"}],
  "risks": ["..."],
  "open_questions": ["..."],
  "timeline": [{"t":"00:12:05","topic":"..."}],
  "participants": [{"name":"Имя","speakers":["S1","S3"]}]
}
Правила:
- Ссылайся на таймкоды важных моментов.
- Объединяй одного человека с несколькими кластерами, если контекст совпадает.
- Если имени нет — оставь S#.
Текстовую выжимку дай после JSON.
<ТРАНСКРИПТ ЗДЕСЬ>
```

## Model Configuration

### Development Environment (LM Studio)

- Qwen3-X models (7-14B, GGUF format)
- Metal acceleration on Mac
- Lower token limits for faster processing

### Production Environment (vLLM)

- Qwen3-30B model with tensor parallelism
- PagedAttention for efficient memory usage
- FP16 (~60-65 GB VRAM) or 4-bit QLoRA/GPTQ (~15-20 GB)
- Context window: 16-32k tokens
- Temperature: 0.2-0.4
- Max new tokens: 1-2k

## Handling Long Meetings

### Chunking Strategy

- For very long meetings (>60 minutes), process in chunks
- Generate intermediate summaries every 30-45 minutes
- Combine chunked summaries into final comprehensive summary
- Maintain temporal coherence across chunks

### Fallback Strategies

1. **Chunking fallback**: Break large transcripts into smaller pieces
2. **Prompt simplification**: Reduce complexity of prompt for long inputs
3. **Token limit handling**: Truncate or summarize very long inputs
4. **Quality degradation**: Accept slightly lower quality for very long meetings

## Integration with Processing Pipeline

### Input Requirements

- Transcript with speaker information and timestamps
- Language specification
- Context about meeting length and complexity

### Output Requirements

- Structured JSON data
- Human-readable summary text
- Properly formatted action items and decisions
- Timecode references where appropriate

### Error Handling

- Handle LLM output parsing failures
- Manage timeouts during processing
- Gracefully handle incomplete responses
- Provide fallback summaries when needed

## Quality Assurance

### Evaluation Metrics

1. **ROUGE scores** - For text quality comparison
2. **QA accuracy** - Test if key information is preserved
3. **Consistency** - Check that action items and decisions are properly extracted
4. **Completeness** - Ensure all required fields are populated

### Validation Steps

1. Validate JSON structure against schema
2. Check for required fields in all sections
3. Verify timecode references are valid
4. Confirm speaker mappings are consistent
5. Validate that action items have proper assignees and due dates

## Performance Considerations

### Resource Management

- Monitor VRAM usage in production
- Implement batching for multiple short meetings
- Cache frequently used prompts or templates
- Optimize token usage to reduce processing time

### Processing Optimization

- Pre-process transcripts to remove noise
- Use appropriate model sizes for different meeting lengths
- Implement efficient prompt construction
- Handle parallel processing where possible

## Implementation Steps

1. Set up LLM interface with vLLM and LM Studio compatibility
2. Implement prompt engineering framework
3. Create structured output parsing system
4. Add validation and error handling
5. Implement chunking strategy for long meetings
6. Test with various meeting types and lengths
7. Optimize performance for both dev and prod environments
8. Integrate with task queue system
9. Add fallback strategies for edge cases
10. Document summarization interfaces and expected behavior
11. Create testing framework for quality assurance
12. Implement monitoring for processing times and success rates
