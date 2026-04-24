# API Rate Limiting and Best Practices

## Overview

Learn how to handle API rate limiting and implement best practices for reliable API integration.

## Rate Limits

### Default Limits

Proxmox VE handles rate limiting at multiple levels:

| Level | Limit |
|-------|-------|
| API | 100 req/min |
| WebSocket | 10 conn/min |
| Auth | 5 attempts/min |

### Configuration

```bash
# View current limits
pvesh get /cluster/options

# API rate limit configuration
# File: /etc/pve/datacenter.conf
```

## Error Handling

### Rate Limit Response

```json
{
  "data": null,
  "errors": [{
    "code": 429,
    "message": "rate limit exceeded"
  }]
}
```

### Handle 429 Errors

```python
import time

def api_call_with_retry(func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return func()
        except Exception as e:
            if e.status_code == 429:
                wait_time = 2 ** attempt
                print(f"Rate limited, waiting {wait_time}s")
                time.sleep(wait_time)
            else:
                raise
```

## Retry Logic

### Exponential Backoff

```python
import time

def request_with_backoff(session, url, max_retries=5):
    for attempt in range(max_retries):
        response = session.get(url)
        
        if response.status_code == 200:
            return response
        
        if response.status_code == 429:
            wait = min(2 ** attempt, 60)
            time.sleep(wait)
            continue
            
        response.raise_for_status()
```

## Caching

### Cache API Responses

```python
import functools
import time

@functools.lru_cache(maxsize=100)
def cached_request(url, ttl=60):
    # Implementation
    return response
```

## Keywords

#rate-limiting #performance #optimization #best-practices

## Related Articles

- [[API]]
- [[API-Authentication]]
---

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]

[[index|Back to Proxmox VE]]
