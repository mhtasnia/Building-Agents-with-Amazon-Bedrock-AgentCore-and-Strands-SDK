# Lesson 2: Introduction to Strands SDK and AgentCore Runtime

Builds a minimal Strands agent, deploys it to Amazon Bedrock AgentCore Runtime, and plugs in a built-in Strands tool.

The agent is WanderBot, a travel assistant for a fictional company called Horizon Travel. It answers travel questions and uses the built-in `calculator` tool to handle arithmetic, so trip cost questions come back with real numbers instead of whatever the model guesses.

## Files

```
demo.py            Agent definition and entrypoint
requirements.txt   Python dependencies
```

`.bedrock_agentcore.yaml` and `.bedrock_agentcore/` appear after your first `agentcore configure`. Both are generated, and both are gitignored.

## What the code does

`BedrockAgentCoreApp` wraps the agent in the HTTP contract AgentCore Runtime expects, so the same file runs locally and in the deployed container without changes. The model is Nova 2 Lite through a cross-region inference profile:

```python
MODEL_ID = "us.amazon.nova-2-lite-v1:0"
model = BedrockModel(model_id=MODEL_ID)
```

The `calculator` import from `strands_tools` is the built-in tool. Passing it in the `tools` list is all the wiring needed. Strands generates the schema, hands it to the model, and runs the function when the model calls it.

## Prerequisites

Python 3.12+, a virtualenv, and AWS credentials that resolve. Credential setup is covered in the [root README](../README.md).

Nova 2 Lite needs to be enabled in your account before it will answer. Bedrock console, Model access, request access, then wait for it to show as granted. A model that is not enabled fails at invoke time with an access denied error rather than anything about the model itself.

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

## Run it locally

```powershell
agentcore configure
agentcore deploy --local
```

Local runs skip CodeBuild and ECR entirely, which makes iterating on the system prompt much faster. Use this while you are still editing `demo.py`.

## Deploy to AgentCore Runtime

```powershell
agentcore configure
agentcore deploy
```

The default path builds an ARM64 container in CodeBuild, pushes it to ECR, and creates the runtime. No local Docker needed. First deploy takes several minutes because it creates the ECR repository and two IAM roles, one for the runtime and one for CodeBuild.

`agentcore deploy --local-build` sits between the two: builds on your machine, deploys to the cloud. Useful if CodeBuild is unavailable in your account.

## Invoke

```powershell
$payload = @{ message = "How do I contact Horizon Travel customer support?" } | ConvertTo-Json -Compress
agentcore invoke $payload
```

A prompt that exercises the calculator tool:

```powershell
$payload = @{ message = "A flight costs `$349. Hotel is `$175/night for 4 nights. Total?" } | ConvertTo-Json -Compress
agentcore invoke $payload
```

Backticks escape the dollar signs so PowerShell does not try to expand them as variables. Add `--dev` to invoke a locally running agent instead of the deployed one.

## Troubleshooting

`No AWS credentials found` from `agentcore configure`. Credentials are not resolving. Test with `python -c "import boto3; print(boto3.client('sts').get_caller_identity())"` and read the actual exception, which is more specific than the toolkit's message. `NoRegionError` counts as a credential failure here, so check that `region` is set in `~/.aws/config`.

`ConfigParseError: Unable to parse config file`. The file was found but is malformed. Usual causes are a missing `[default]` section header or an invisible byte order mark on the first line. Check for the BOM with `Format-Hex $env:USERPROFILE\.aws\config | Select-Object -First 1` and look for `EF BB BF`. Rewrite with `Out-File -Encoding ascii`.

`Got unexpected extra argument(s)` from `agentcore invoke`. PowerShell 5.1 re-parses quotes when calling native programs and splits the JSON on spaces. Use the `ConvertTo-Json` form above, or `--%` before the payload to stop PowerShell parsing.

Build stops during PROVISIONING with status `STOPPED`. `STOPPED` means something called the CodeBuild StopBuild API, not that the build failed. If it reproduces when you start a build directly with boto3, outside the toolkit, the cause is in the account rather than the CLI. Look for a StopBuild event in CloudTrail to see the caller, and check that the CodeBuild concurrent build quota is not zero. New accounts pending verification sometimes have builds killed this way with no error surfaced, which needs a Support case. `agentcore deploy --local` is a workable fallback in the meantime.

Deprecation banner on every command. Expected. The pip starter toolkit is no longer supported and the npm CLI (`npm install -g @aws/agentcore`) is the replacement. `AGENTCORE_SUPPRESS_RECOMMENDATION=1` silences it.
