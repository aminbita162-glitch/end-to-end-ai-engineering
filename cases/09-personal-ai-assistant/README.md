# Personal AI Assistant

**Status:** `MVP in development`

A personal AI assistant integrating task management, notes, scheduling, and OpenAI-powered responses.

## What It Is

An MVP-stage personal AI assistant that connects task management, note-taking, scheduling, and OpenAI to help with daily personal workflows.

## What It Is Not

- Not a completed product.
- Not a live hosted service (no verified live URL at this time).
- Not a multi-user platform.

## Problem

Personal productivity tools are fragmented. Tasks, notes, and schedules exist in separate apps with no AI layer to connect or reason over them.

## Scope

MVP scope:
- Task management
- Notes
- Schedule integration
- OpenAI-powered responses

Actively in development.

## Architecture

Backend service integrating task, note, and schedule data with OpenAI API for response generation. Architecture documented in source repository.

## Engineering Decisions

- OpenAI integration for assistant responses.
- MVP focus: core personal productivity loop before expanding to additional integrations.
- Backend-first approach: establish data models and API before frontend.

## Stack

| Layer | Technology |
|-------|-----------|
| AI Integration | OpenAI |

Full stack documented in source repository.

## Testing

Refer to source repository for current test coverage.

## Deployment Model

Local development / MVP stage. No verified live URL available at this time.

## Live Links

No verified live URL. Refer to source repository for setup instructions.

## Technical Evidence

Refer to the source repository and:
[`projects/02-ai-assistant-project/`](../../projects/02-ai-assistant-project/)

That folder contains the containerized, API-first backend implementation with Docker Compose setup, authentication, conversation management, message persistence, and OpenAI integration.

## Measurable Results

MVP in development. No production metrics claimed.

## Source Repository

https://github.com/aminbita162-glitch/personal-ai-assistant
