WanderBot — Short-Term Memory Management with Bedrock AgentCore

An implementation of short-term memory management for an AI travel assistant (WanderBot) utilizing the Bedrock AgentCore Starter Toolkit, Strands SDK, and Amazon Bedrock.

This project demonstrates how context can persist across multi-turn user conversations through custom hook-based memory injection.

## Architecture & Project Structure

```text
agentcore-state-management/
├── .bedrock_agentcore/
├── datasets/
│   └── hotels.json
├── screenshots/
│   ├── session_1.png
│   └── session_2.png
├── .bedrock_agentcore.yaml
├── .dockerignore
├── demo.py
├── memory_setup.py
└── requirements.txt

Execution Screenshots
Session Execution — Step 1: Initial Setup & Turn 1

The first session establishes the user's travel context and destination. WanderBot receives the request to plan a budget trip to Barcelona and stores the conversation context in AgentCore Memory.

<p align="center"> <img src="screenshots/session_1.png" alt="Session 1 — Initial Setup & Turn 1" width="800"/> </p>
Session Execution — Step 2: Multi-Turn Context Retention & Tool Invocation

The second session demonstrates that WanderBot successfully retains the destination context from the previous turn and uses it without requiring the user to repeat the destination.

<p align="center"> <img src="screenshots/session_2.png" alt="Session 2 — Multi-Turn Context Retention & Tool Invocation" width="800"/> </p>
Implementation Checklist & Task Status


The client supplies a stable session_id and actor_id in the invocation payload. This allows user context to persist across page refreshes or device switches.

Hook-Based Memory Injection

AgentInitializedEvent queries the Bedrock AgentCore Memory data plane through MemoryClient to retrieve the last K conversational turns.

The retrieved turns are formatted and appended directly to the agent's system_prompt dynamically during agent initialization.

Event Persistence

MessageAddedEvent captures new messages as they occur and writes them asynchronously back to the memory store.

Running the Agent Locally
1. Start the Local Development Server

Launch the application using the Bedrock AgentCore CLI runtime interface or a direct ASGI server:

python -m uvicorn demo:app --port 8080 --reload

2. Execute the Multi-Turn Conversation

In a separate terminal window, define a persistent session variable and run the three-turn sequence to test memory retention:

$sessionId = "barcelona_trip"

# Turn 1: Establish destination and intent
agentcore invoke --dev --session-id $sessionId '{"message": "I want to plan a trip to Barcelona on a budget", "actor_id": "alice"}'

# Turn 2: Filter hotels by budget
# Agent retains "Barcelona" from turn 1
agentcore invoke --dev --session-id $sessionId '{"message": "What hotels are available under $150 per night?", "actor_id": "alice"}'

# Turn 3: Compare specific attributes like cancellation policies
agentcore invoke --dev --session-id $sessionId '{"message": "Which one has the best cancellation policy?", "actor_id": "alice"}'

Expected Behavior

Across the three turns, WanderBot maintains the conversation context:

The user establishes Barcelona as the destination and specifies a budget-oriented trip.
The user asks for hotels under $150 per night without repeating the destination.
The user asks about cancellation policies without specifying which destination or hotels they are referring to.

The memory hooks allow WanderBot to understand these follow-up requests using the context stored from previous turns.
