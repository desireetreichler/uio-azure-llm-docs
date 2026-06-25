# Using LLMs for Coding via UiO's Azure Solution

This guide shows three practical ways to get started with LLMs for coding through UiO's Azure-based setup:

1. Direct API access from your own code
2. Extensions to VS Code (Zoo/Roo code, Codex)
3. Agent use through the Codex app/client (Windows/Mac, no release for Linux at the time of writing)

At UiO, the Azure route is useful when you want LLM access through an institutionally managed setup rather than a personal public API account. This allows you to use LLMs on green and yellow data, and to bill associated costs to a project rather than paying personally.

The guide is based on workshop notes, personal try and error, and tested setups for Windows, Linux and Mac OS in spring 2026 (but not all setups were tested on all platforms). Rapid development in tools means that some approaches may soon/already be outdated.

## Requirements

For this to work, you will need:

- access to Azure AI Foundry, for this you need to go through [UiO's GPT/Azure ordering process](https://www.uio.no/tjenester/it/ki/gpt-uio/) ("bestill API-tilgang", requires a project that can be billed, the project PI can request access for several users in the project. Experience shows that this can take days to months to approve.)
- a deployed model in your Azure project (see guide below)
- the deployment name
- the endpoint or base URL
- your API key

## Azure Setup

In Azure AI Foundry:

1. Sign in to [UiO azure](https://ai.azure.com/)
2. Open your project.
3. Go to the models and endpoints area (scroll down left to My assets -> models + endpoints -> new model)
4. Deploy the model you want to use. At the time of writing, gpt-5.4 was the one we got to work smoothly. gpt-5.5 seems to work as well but costs considerably more.
5. Note: If the model shows a lock, you have to request access by filling in the provided form. It took ca. one hour to be granted access (and access was granted to a series of gpt 5 models)
6. You will need the deployment name (the model name), endpoint information (Target URI), and API key. Endpoint information is shared for all models. Note: delete everything starting from /responses... and replace with /v1, so that the URL looks like this: "https://<your_project>.openai.azure.com/openai/v1"

This information is referred to as the following later in the guide: 

- `AZURE_OPENAI_BASE_URL`
- `AZURE_OPENAI_API_KEY`
- `AZURE_OPENAI_MODEL`

Important:

- do never paste API keys into documentation, they are secret
- keep deployment names exactly as defined in Azure
- the Target URI you see when you deploy a model (and click on it) is different from the URL you see in your overview on Azure. It seems the one from the model view is needed (in a truncated form)

## Option 1: Direct API Access From Code

This is the suggested way if you want to build your own scripts, notebooks, or applications. 

### Optional: store credentials as environment variables

This step is mainly for direct code usage. VSCode extensions and the Codex app may instead let you enter the same values in their own settings. 

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
Source your .bashrc file or restart your terminal/re-connect to the server to make the changes persistent.

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


## Option 2: Extensions in VS Code

There are extensions that can be installed for VS Code that run locally or in an SSH server session. This is probably the most convenient route for people who want LLM support directly inside VS Code. 
We have tested Zoo (previously Roo) Code and the Codex extension.

### Codex extension
Codex is OpenAI's agent solution and designed to work with OpenAI's models provided through Azure (as described above) or [through gpt.uio.no](https://pages.github.uio.no/alexajo/agent-skolen/setup_codex.html#models-made-available-through-gpt.uio.no) (by invitation only at the time of writing). 

Setup:
- Rather than setting environment variables as described above it seems necessary (possibly on Linux only?) to create a .env file and set the API key there in your HOME/.codex folder (on Linux/Mac OS: ~/.codex/.env):
```bash
AZURE_OPENAI_API_KEY="REPLACE_WITH_REAL_KEY"
```
- Install the Codex extension in VSCode and open it (the icon is on the top/right, not in the left sidebar)
- **Do not sign in**, but choose "Use API Key". Put in a dummy string and click continue until you seem to be logged in.
- Go to settings and open the config.toml. Enter the information as shown below for option 3. **Important:** do not paste your key but keep the reference to the environment variable "AZURE_OPENAI_API_KEY" as set above.
- Close and re-open the extension

The extension shares its settings with the app/client versions described in option 3, and sessions should be shared between them on the same machine. Detailed setup instructions for Codex are available [here](https://pages.github.uio.no/alexajo/agent-skolen/setup_codex.html) (access requires UiO account login).

### Zoo Code extension
[Zoo Code](https://www.zoocode.dev/) is a community-driven version of Roo, who discontinued their extension. The extension works with VS Code and lets you use your own model (instead of the built-in Copilot setup). It is not limited to OpenAI models.

Setup:
- Install the Zoo extension
- Click on the Zoo (Zebra) icon in the left sidebar, then open the settings.
- You can store several model setups here by creating a new Configuration Profile for each. 

**What to enter in Zoo**
OpenAI setup that worked in June 2026 for Zoo (and March 2026 for Roo):
- config profile: a custom name, call it the same as your deployed model on Azure to avoid confusion.
- Provider: `OpenAI`
- check `use custom base URL`, and paste the URL/endpoint from Azure. Note: truncate starting from /responses and replace this with /v1, so that it takes the form "https://YOUR-RESOURCE.openai.azure.com/openai/v1/"    
- OpenAPI key -> paste your Azure API key from Azure
- service tier: standard
- model: gpt-5.4 (Note: this has to be deployed in Azure first. Use your Azure deployment name here.)
- reasoning effort: kept empty (none selected)
- verbosity: medium
Anthropic setup that worked in June 2026 for Zoo:
- When using Anthropic as provider, note that the URL/endpoint should be without /v1 : "https:/YOUR-RESOURCE.ai.azure.com/anthropic/". 


Treat this as a tested guidance rather than a guarantee that every menu label will look the same in your version/at the time you try this.

Experience shows that Zoo/Roo and the UiO Azure setup changed rapidly during spring 2026 and the [LLM workshop tutorial from January 2026](https://lexnederbragt.github.io/dsc26-llm-code/tutorial.html) are already outdated. Some noted changes/differences that may help with debugging:
- older Roo versions used a `3rd party provider` path
- in newer versions you may need to use `OpenAI Compatible` or `OpenAI`
- different people got different models to work at different times
- Azure URLs changed between January and March


## Option 3: Codex App or Codex CLI

The [Codex app ](https://developers.openai.com/codex/quickstart?setup=app) is a good fit for users who want a standalone coding agent rather than editor-only integration. This is fairly hands-off any code, and possibly more suitable for (small) stand-alone tasks rather than explorative coding in big projects. The app is only available for Windows/Mac OS at the time of writing. 

The [Codex CLI](https://developers.openai.com/codex/cli) is a terminal-based program with the same functionality. It is similar to Claude Code and available for all platforms.

UiO-targeted setup instructions for Codex are available [here](https://pages.github.uio.no/alexajo/agent-skolen/setup_codex.html) (access requires UiO account login). This resource also provides a guide on how to make Codex connect to tools with API access such as Canvas or Nettskjema.

In March, the workflow to configure the Codex App looked like this:
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
base_url = "YOUR CUSTOM URL"     # in the form: "https://YOUR-RESOURCE.openai.azure.com/openai/v1/"
env_key = "AZURE_OPENAI_API_KEY"
wire_api = "responses"
```

This information is stored in the file `~/.codex/config.toml` on Mac OS or Linux. On Windows, the standard file path is `C:\Users\"USERNAME"\.codex\config.toml`. 

The setup for the **Codex CLI ** can be done by providing it with the same information in the `config.toml` file,
stored in the same location. Same applies to the VSCode extension.
When you use either of them on the same machine, they share configuration and sessions.

## Resources
- [LLM workshop tutorial from January 2026](https://lexnederbragt.github.io/dsc26-llm-code/tutorial.html)
- [Codex setup (agent-skolen for UiO)](https://pages.github.uio.no/alexajo/agent-skolen/setup_codex.html) (access requires UiO account login).
- [Information about UiO's GPT access, Link to the Azure ordering process](https://www.uio.no/tjenester/it/ki/gpt-uio/)
- [UiO Foundry/Azure Model Deployment](https://ai.azure.com/)
- [UiO Azure Portal (monitor your usage/costs)](https://portal.azure.com)
- [Codex](https://developers.openai.com/codex/)


## Troubleshooting

### Authentication fails

Check:

- the key matches the same Azure resource as the endpoint
- the deployment name is exact
- the client expects a base URL, not a full request URL
- your environment variable is actually loaded in the current shell

### Python imports fail

You are probably using a different environment than the one where `openai` was installed.

### Zoo or Codex cannot find the model

Try the Azure deployment name exactly as it appears in Azure AI Foundry. Try a different model. 

## Security Checklist

- never commit API keys
- use environment variables or a secret manager
- remove secrets from screenshots and shared notes
- rotate any key that has been exposed in plain text
