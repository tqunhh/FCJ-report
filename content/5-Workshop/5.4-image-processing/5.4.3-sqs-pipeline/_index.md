---
title: "SQS Pipeline"
date: 2025-12-08
weight: 3
chapter: false
pre: "<b>5.4.3. </b>"
---

## Purpose

Process images asynchronously to:
- Reduce system load when multiple users upload simultaneously
- Ensure no message loss when Lambda encounters errors
- Separate processing steps (moderation → detect labels)

## SQS Pipeline Architecture

```
S3 Upload Event
    ↓
SQS: ModerationQueue
    ↓
Lambda: Content Moderation
    ↓ (if approved)
SQS: DetectLabelsQueue
    ↓
Lambda: Detect Labels
    ↓
DynamoDB + Gallery
```

## Demo: SQS Queues Overview

The system uses multiple SQS queues to process images in a pipeline:

![SQS Queues Overview](/images/5-Workshop/5.4-image-processing/SQS_QUESUES_OVERVIEW.png)

## Benefits of SQS

### 1. Asynchronous Processing
- Lambda doesn't need to wait for completion
- Users upload images quickly
- Processing happens in background

### 2. Automatic Retry
- If Lambda fails, message returns to queue
- Automatic retry with backoff
- Dead Letter Queue (DLQ) for repeatedly failed messages

### 3. Auto Scaling
- SQS automatically scales with message volume
- Lambda triggers in parallel for multiple messages
- No system overload concerns

### 4. Step Separation
- Content Moderation and Detect Labels are independent
- Easy to debug and monitor each step
- Simple to add new processing steps

## Queue Configuration

### ModerationQueue
- Visibility Timeout: 300s (5 minutes)
- Message Retention: 4 days
- DLQ: ModerationDLQ (after 3 retries)

### DetectLabelsQueue
- Visibility Timeout: 180s (3 minutes)
- Message Retention: 4 days
- DLQ: DetectLabelsDLQ (after 3 retries)

## Monitoring

Track important metrics:
- **ApproximateNumberOfMessagesVisible** - Messages waiting
- **ApproximateAgeOfOldestMessage** - Oldest message age
- **NumberOfMessagesReceived** - Total messages received
- **NumberOfMessagesSent** - Total messages sent

## Conclusion

SQS Pipeline enables the system to:
- Process reliably and stably
- Scale automatically based on demand
- Easy to maintain and extend
