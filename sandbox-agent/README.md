# Hello World sandbox agent

This sample uses the Python OpenAI Agents SDK `SandboxAgent` with
`DockerSandboxClient`. The SDK harness runs on the host; Docker provides the
isolated workspace where the public GitHub repository is cloned and edited.
The model call goes from the host to Ollama's OpenAI-compatible API, so the
sandbox does not need an Ollama or GitHub credential.

## Prerequisites

- Python 3.10 or newer
- Docker Desktop running
- Ollama running with `gpt-oss:20b-cloud` available, or choose a local model
- GitHub access is only needed later for the host-side push and pull request

## Run

From the repository root in PowerShell:

```powershell
py -m venv sandbox-agent\.venv
.\sandbox-agent\.venv\Scripts\Activate.ps1
python -m pip install -r sandbox-agent\requirements.txt
$env:OLLAMA_MODEL = "gpt-oss:20b-cloud"
python sandbox-agent\run_agent.py
```

To use the local model installed on this machine instead:

```powershell
$env:OLLAMA_MODEL = "qwen3:4b-instruct-2507-q4_K_M"
python sandbox-agent\run_agent.py
```

The first run builds a small sandbox image with Git, then the SDK fetches
`gvece1988/hello-world` into a fresh Docker workspace. The agent corrects the
greeting in `program.cs`. The runner copies the changed file and a unified diff
into `sandbox-agent/artifacts/`; review the diff before applying it to the host
checkout.

Set `HELLO_WORLD_REF` to choose a different source branch. Set
`OLLAMA_BASE_URL` if Ollama is listening somewhere other than
`http://localhost:11434/v1`.

## Commit and open a pull request

After reviewing the generated patch, apply it to the host checkout and use
ordinary GitHub authentication outside the sandbox:

```powershell
git apply sandbox-agent\artifacts\program.patch
git diff --check
git diff
git add program.cs README.md .gitignore sandbox-agent
git commit -m "Fix Hello World greeting and add sandbox demo"
git push -u origin demo/docker-sandbox-agent
gh pr create --base master --title "Fix Hello World greeting and add sandbox demo" --body "Adds a Docker sandbox agent example using Ollama and fixes the greeting typo."
```

The sandbox does not receive GitHub credentials and does not commit or push.
