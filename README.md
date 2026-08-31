# Building Agents with Amazon Bedrock AgentCore and Strands SDK

Course work and exercises for Course 3: Building Agents with Amazon Bedrock AgentCore and Strands SDK.

Each lesson lives in its own directory with its own README, source, and `requirements.txt`. Lessons are self-contained, so you can run any one of them without having done the others.

## Repo layout

```
.
├── 01-introduction/
├── 02-strands-agentcore-runtime/
│   ├── README.md
│   ├── demo.py
│   └── requirements.txt
├── 03-strands-agentcore-function-calling/
├── ...
├── 22-project-ai-support-agent/
└── README.md
```

## Lessons

| # | Lesson | Topic |
|---|--------|-------|
| 1 | Introduction: Building AI Agents with Bedrock AgentCore | Cloud resources, containerized labs |
| 2 | Introduction to Strands SDK and AgentCore Runtime | Minimal Strands agent, deploy to Runtime, built-in tool |
| 3 | Extending Agents with Tools | Tool integrations for real-time actions |
| 4 | Building Agents with Bedrock AgentCore Function Calling | Python functions as tools, LLM-driven calling |
| 5 | Structured Outputs | Schemas, parsers, responses as actionable JSON |
| 6 | Implementing Structured Outputs with Strands and AgentCore | Pydantic models, schema-conformant tool output |
| 7 | Agent State Management | State machines, tracking input and tool use |
| 8 | Short-Term Agent Memory | Ephemeral context retention in a session |
| 9 | Implementing Agent State Management with AgentCore Memory | HookProviders, lifecycle events, multi-turn |
| 10 | External Tools and APIs | MCP, authenticating agents against external APIs |
| 11 | Integrating APIs with AgentCore Gateway | Lambda and API tools via MCP |
| 12 | Securing Agents with AgentCore Identity | Credential storage, Gateway targets |
| 13 | Web Search Agents | Grounding responses, handling noise |
| 14 | Creating Web Search Agents with AgentCore Browser | Managed headless Chrome sessions |
| 15 | Interacting with Databases | SQL and vector databases |
| 16 | Agentic Retrieval Augmented Generation | Reflection, query reformulation, retry loops |
| 17 | Building RAG Systems using S3 Vector Store and Bedrock API | Knowledge Bases, citation-based answers |
| 18 | Long-Term Agent Memory | Semantic, episodic, and procedural memory |
| 19 | Maintaining Long-Term Agent Memory with DynamoDB | Preferences across sessions |
| 20 | Executing Code with AgentCore Code Interpreter | Python in a sandbox from a Strands tool |
| 21 | Monitoring and Governing Agents with Observability and Policy | CloudWatch GenAI Observability Dashboard |
| 22 | Project: AI Support Agent | End-to-end agent workflow (due September 20, 2026) |

## Setup

Everything here targets Python 3.12+ and assumes a virtual environment per lesson.

```powershell
cd 02-strands-agentcore-runtime
python -m venv .venv
.\.venv\Scripts\Activate.ps1        # macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
```

If `pip install` ever prints "Defaulting to user installation," your venv is not active. Activate it and reinstall, otherwise packages land in `~/.local` and get shared across projects in ways that will bite you later.

## AWS credentials

The lessons all call Bedrock, so credentials need to resolve before anything works. They belong in your home directory, not in the project folder. boto3 does not look in the working directory.

`~/.aws/credentials` (Windows: `C:\Users\<you>\.aws\credentials`):

```ini
[default]
aws_access_key_id = ...
aws_secret_access_key = ...
aws_session_key = ....
```

`~/.aws/config`:

```ini
[default]
region = us-east-1
output = json
```

Verify without installing the AWS CLI:

```powershell
python -c "import boto3; print(boto3.client('sts').get_caller_identity())"
```

An account ID and ARN means you are good. Two things that commonly break these files: a missing `[default]` header, and a byte order mark added by Notepad or a PowerShell `>` redirect. Both produce a `ConfigParseError`. Writing the file with `Out-File -Encoding ascii` avoids the second one.

Model access also needs enabling separately. In the Bedrock console under Model access, request the models a lesson uses before running it. The `us.` prefix on a model ID means a cross-region inference profile covering US regions, so your configured region has to be one of them.


## What is not committed

`.gitignore` covers virtualenvs, bytecode, and generated build artifacts:

```
.venv/
__pycache__/
*.pyc
.bedrock_agentcore/
.aws/
.env
```
