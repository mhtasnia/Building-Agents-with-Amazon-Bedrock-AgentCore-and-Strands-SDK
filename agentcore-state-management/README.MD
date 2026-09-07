# WanderBot — Short-Term Memory Management with Bedrock AgentCore

An implementation of short-term memory management for an AI travel assistant (**WanderBot**) utilizing the **Bedrock AgentCore Starter Toolkit**, **Strands SDK**, and **Amazon Bedrock**. This project successfully solves context persistence across multi-turn user conversations through custom hook-based memory injection.

---

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
Implementation Checklist & Task StatusAll core requirements and exercises for short-term memory management have been successfully completed:[x] Create a memory resource using agentcore memory create WanderBot and store the resource ID into MEMORY_ID.[x] Import hook classes (HookProvider, HookRegistry, AgentInitializedEvent, MessageAddedEvent) from strands.hooks.[x] Configure hook class attributes in __init__ to retain memory_client, memory_id, and last_k_turns.[x] Bind events in register_hooks() mapping AgentInitializedEvent to on_agent_initialized and MessageAddedEvent to on_message_added.[x] Build the Agent in the entrypoint with hooks=[ShortTermMemoryHookProvider(memory_client, MEMORY_ID)] and pass explicit session/actor context via state={"session_id": session_id, "actor_id": actor_id}.[x] Run a continuous multi-turn conversation ensuring WanderBot seamlessly retains the destination context across turns without re-prompting the user.How Short-Term Memory WorksClient-Driven Session Continuity: The client supplies a stable session_id and actor_id in the invocation payload. This guarantees that user context persists across page refreshes or device switches.Hook-Based Injection:AgentInitializedEvent queries the Bedrock AgentCore Memory data plane via MemoryClient to retrieve the last $K$ conversational turns.Retrieved turns are formatted and appended directly to the agent's system_prompt dynamically upon initialization.Event Persistence: MessageAddedEvent captures new messages as they occur and writes them asynchronously back to the memory store.Execution ScreenshotsSession Execution Step 1 (Initial Setup & Turn 1)Session Execution Step 2 (Multi-turn Context Retention & Tool Invocation)Running the Agent Locally1. Start the Local Development ServerLaunch the application via the Bedrock AgentCore CLI runtime interface or direct ASGI server:PowerShellpython -m uvicorn demo:app --port 8080 --reload
2. Execute the Multi-Turn ConversationIn a separate terminal window, define a persistent session variable and run the three-turn sequence to test memory retention:PowerShell$sessionId = "barcelona_trip"

# Turn 1: Establish destination and intent
agentcore invoke --dev --session-id $sessionId '{"message": "I want to plan a trip to Barcelona on a budget", "actor_id": "alice"}'

# Turn 2: Filter hotels by budget (Agent retains 'Barcelona' from turn 1)
agentcore invoke --dev --session-id $sessionId '{"message": "What hotels are available under $150 per night?", "actor_id": "alice"}'

# Turn 3: Compare specific attributes like cancellation policies
agentcore invoke --dev --session-id $sessionId '{"message": "Which one has the best cancellation policy?", "actor_
