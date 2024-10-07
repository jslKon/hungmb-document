# Transactional Outbox

## Context

Update to DBs and messages to broker MUST be atomic.

Sending a message in middle of transaction -> not reliable -> no guarantee transaction commit.
Send message after commit -> no guarantee no crash

## Solution

First store the messages -> then a separate process sends message to message broker