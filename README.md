# Using LLMs for Coding via UiO's Azure Solution

This guide shows three practical ways to get started with LLMs for coding through UiO's Azure-based setup:

1. direct API access from your own code
2. Roo Code in VS Code
3. the Codex app

The guide is based on workshop notes, and tested setups for Windows and Mac OS in Jan-March 2026. Rapid development in tools means that some approaches may soon/already be outdated.

## Before You Start

You will need:

- access to Azure AI Foundry, for this you need to go through [UiO's GPT/Azure ordering process](https://www.uio.no/tjenester/it/ki/gpt-uio/) ("bestill API-tilgang", requires a project that can be billed, the project PI can request access for several users in the project. Experience shows that this can take days to months to approve.)
- a deployed model in your Azure project
- the deployment name
- the endpoint or base URL
- your API key

At UiO, the Azure route is useful when you want LLM access through an institutionally managed setup rather than a personal public API account. This allows you to use LLMs on green and yellow data, and to bill associated costs to a project rather than paying personally.

## Shared Azure Setup

In Azure AI Foundry:

1. Sign in to [UiO azure](https://ai.azure.com/)
2. Open your project.
3. Go to the models and endpoints area (scroll down left to My assets -> models + endpoints -> new model)
4. Deploy the model you want to use. At the time of writing, gpt-5.4 was the one we got to work. 
5. Note: If the model shows a lock, you have to request access by filling in the provided form. It took ca. one hour to be granted access (and access was granted to a series of gpt 5 models)
6. You will need the deployment name (the model name), endpoint information (Target URI), and API key. Endpoint information is shared for all models. 

This information is referred to as the following later in the guide: 

- `AZURE_OPENAI_BASE_URL`
- `AZURE_OPENAI_API_KEY`
- `AZURE_OPENAI_MODEL`

Important:

- do never paste API keys into documentation, they are secret
- keep deployment names exactly as defined in Azure

## Option 1: Direct API Access From Code

This is the suggested way if you want to build your own scripts, notebooks, or applications. I have not tested this.

### Optional: store credentials as environment variables

This step is mainly for direct code usage. Roo Code and the Codex app may instead let you enter the same values in their own settings.

#### PowerShell

```powershell
$env:AZURE_OPENAI_BASE_URL = "https://YOUR-RESOURCE.openai.azure.com/openai/v1/"
$env:AZURE_OPENAI_API_KEY = "REPLACE_WITH_REAL_KEY"
$env:AZURE_OPENAI_MODEL = "YOUR_DEPLOYMENT_NAME"
```

To persist them in Windows:

```powershell
setx AZURE_OPENAI_BASE_URL "https://YOUR-RESOURCE.openai.azure.com/openai/v1/"
setx AZURE_OPENAI_API_KEY "REPLACE_WITH_REAL_KEY"
setx AZURE_OPENAI_MODEL "YOUR_DEPLOYMENT_NAME"
```

#### macOS / Linux

```bash
export AZURE_OPENAI_BASE_URL="https://YOUR-RESOURCE.openai.azure.com/openai/v1/"
export AZURE_OPENAI_API_KEY="REPLACE_WITH_REAL_KEY"
export AZURE_OPENAI_MODEL="YOUR_DEPLOYMENT_NAME"
```

To persist them, add the `export` lines to your shell startup file such as `~/.bashrc` or `~/.zshrc`.

### Python example

Install the SDK:

```bash
pip install openai
```

Then try:

```python
import os
from openai import OpenAI

def require_env(name: str) -> str:
    value = os.environ.get(name)
    if not value:
        raise RuntimeError(f"Missing required environment variable: {name}")
    return value

client = OpenAI(
    api_key=require_env("AZURE_OPENAI_API_KEY"),
    base_url=require_env("AZURE_OPENAI_BASE_URL"),
)

response = client.responses.create(
    model=require_env("AZURE_OPENAI_MODEL"),
    input="Explain in three bullet points how an LLM coding assistant can help with debugging."
)

print(response.output_text)
```

Notes:

- if you use `conda` or `mamba`, make sure you run the script from the same environment where `openai` is installed
- if commands run in the wrong Python environment, call them explicitly, for example `mamba run -n YOUR_ENV python script.py`
- many clients expect the deployment name, not just the model family name

## Option 2: Roo Code in VS Code

Roo is an extension that can be installed for VS Code. It can run locally or in a SSH server session. This is probably the most convenient route for people who want LLM support directly inside VS Code. 

- install the Roo extension
- click on the Roo (Kangaroo) icon in the left sidebar

### What to enter in Roo
- base URL: your Azure endpoint or base URL
- API key: your Azure API key
- model: your Azure deployment name

Experience shows that Roo and the UiO Azure setup changed since the [LLM workshop tutorial from January 2026] (https://lexnederbragt.github.io/dsc26-llm-code/tutorial.html)
- older Roo versions used a `3rd party provider` path
- in newer versions you may need to use `OpenAI Compatible` or `OpenAI`
- different people got different models to work at different times
- Azure URLs changed between January and March

Setup that worked in March 2026:
- config profile: a custom name
- Provider: `OpenAI`
- check `use custom base URL`, and paste the URL from Azure. Note: I had to delete everything starting from /responses... in the URL, thus only keeping the first half of the URL.
- OpenAPI key -> paste from Azure
- service tier: standard
- model: gpt-5.4 (Note: this has to be deployed in Azure first. I tried other models, but they didn't work)
- reasoning effort: none
- verbosity: medium 

Treat this as a tested guidance rather than a guarantee that every menu label will look the same in your version/at the time you try this.

## Option 3: Codex App

The Codex app is a good fit for users who want a standalone coding agent rather than editor-only integration. This is fairly hands-off any code, and possibly more suitable for (small) stand-alone tasks rather than explorative coding in big projects. 

In March, the workflow to configure Codex looked like this:
- Install the Codex app from Open AI (root access required for this to work properly. Note it will ask for root authorisation only when trying it out first time)
- In a terminal, set the environment variable AZURE_OPENAI_API_KEY as described above 
- In Codex, open Settings, choose Configuration -> set config file
- Replace the first few lines with the following (keep any last lines that look like they are specific to your OS system)
```toml
model = "YOUR_DEPLOYMENT_NAME"
model_provider = "azure"
model_reasoning_effort = "medium"

[model_providers.azure]
name = "Azure OpenAI"
base_url = "your URL, again only keep the first part up to (not including) /responses..."
env_key = "AZURE_OPENAI_API_KEY"
wire_api = "responses"
```


## Troubleshooting

### Authentication fails

Check:

- the key matches the same Azure resource as the endpoint
- the deployment name is exact
- the client expects a base URL, not a full request URL
- your environment variable is actually loaded in the current shell

### Python imports fail

You are probably using a different environment than the one where `openai` was installed.

### Roo or Codex cannot find the model

Try the Azure deployment name exactly as it appears in Azure AI Foundry. Try a different model. 

## Security Checklist

- never commit API keys
- use environment variables or a secret manager
- remove secrets from screenshots and shared notes
- rotate any key that has been exposed in plain text

## Suggested Next Additions

If you want to expand this guide later, useful next additions would be:

- a `python_quickstart.py` example file in the repository
- a short `curl` example
- screenshots from Azure AI Foundry and Roo
- a short FAQ with the most common setup mistakes
