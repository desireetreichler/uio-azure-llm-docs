# Using LLMs for Coding via UiO's own solutions

This guide shows three practical ways to get started with LLMs for coding through UiO's GPT-UiO and Azure-based setups:

1. Direct API access from your own code
2. Extensions to VS Code (Zoo/Roo code, Codex)
3. Agent use through the Codex CLI/app (the client is universal, the app only for Windows/Mac only at the time of writing)

At UiO, the Azure route is useful when you want LLM access through an institutionally managed setup rather than a personal public API account, and don't want limited tokens (your project pays per token). This allows you to use LLMs on green and yellow data, and to bill associated costs to a project rather than paying personally. The route through GPT UiO also works for red data and offers open access models in addition to several GPT models (up to 5.4), and has token limits.

The guide is based on workshop notes, personal try and error, and tested setups for Windows, Linux and Mac OS in spring/summer 2026 (but not all setups were tested on all platforms). Rapid development in tools means that some approaches may soon/already be outdated.

## Requirements

For this to work, you will need an API key pointing to your model of choice. UiO offers two solutions:

A) access to **Azure AI Foundry**, for this you need to go through [UiO's Foundry ordering process](https://www-int.uio.no/tjenester/it/ki/foundry/hjelp/bestill-api/api.html) (requires a project that can be billed. The project PI can request access for several users in the project. Previously this could take days to months to approve.)

B) **personal API keys for GPT-UiO***, this you have to ask for by explaining your needs in an [email to gpt-drift@usit.no](https://www.uio.no/tjenester/it/ki/gpt-uio/hjelp/api-nokler.html) (gives you API access to various models, including several ones running locally that are OK to use with red data, but also GPT-5.4)

For any model (also personal subscriptions), you need:
- the model to be deployed (for Azure/Foundry projects, see guide below)
- the deployment name
- the endpoint or base URL
- your API key

## GPT UiO personal API keys Setup
Once your API key access has been granted, log on to gpt.uio.no, then click on your profile name and select My API keys. Generate a key, then retrieve API key, Base URL etc. by clicking on your model of choice. 

## Azure Setup

In Azure AI Foundry:

1. Sign in to [UiO azure](https://ai.azure.com/)
2. Open your project.
3. Go to the models and endpoints area (scroll down left to My assets -> models + endpoints -> new model)
4. Deploy the model you want to use. At the time of writing, gpt-5.4 was the one we got to work smoothly. gpt-5.5 seems to work as well but costs considerably more. We also got Claude to work (that seemed considerably more expensive). 
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
If you want to be able to switch between several models/API keys/endpoints, you can set up individual environments for these and store the keys in each their environment file instead. 

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

The [Codex CLI](https://developers.openai.com/codex/cli) is a terminal-based program with the same functionality. It is similar to Claude Code and available for all platforms.

The [Codex app ](https://developers.openai.com/codex/quickstart?setup=app) is a good fit for users who want a standalone coding agent rather than editor-only integration. This is fairly hands-off any code, and possibly more suitable for (small) stand-alone tasks rather than explorative coding in big projects. The app is only available for Windows/Mac OS at the time of writing. 

UiO-targeted setup instructions for Codex are available [here](https://pages.github.uio.no/alexajo/agent-skolen/setup_codex.html) (access requires UiO account login). This resource also provides a guide on how to make Codex connect to tools with API access such as Canvas or Nettskjema.

### Codex app (one model/profile)
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

### Codex CLI 
The setup for the **Codex CLI** can be done by providing it with the same information in the `config.toml` file,
stored in the same location. Same applies to the VSCode extension.
When you use either of them on the same machine, they share configuration and sessions.

If you want to be able to switch between several models/API keys/endpoints, you can set up individual environments for these and store the keys in each their environment file instead. 
It is possible to install Codex CLI on a server with multiple models/profiles (OpenAI family only) for different purposes, for this follow the setup below.

## Codex CLI Profile Setup on UNIX with several profiles
This setup uses one shared user-level Codex config plus one profile file per recurring model/provider combination.

### Files
User-level config:

- `~/.codex/config.toml`

One file per profile/model, for example:

- `~/.codex/gpt54uio.config.toml`     # GPT-UiO model accessible through personal API key
- `~/.codex/gpt5miniuio.config.toml`  # Another GPT-UiO model
- `~/.codex/azure54.config.toml`      # A model deployed from UiO Azure/Foundry


Environment files:

- `~/.config/codex/env/gptuio.env`    # GPT-UiO models share the same environment (access point, API key)
- `~/.config/codex/env/azure54.env`


Shell helpers:

- `~/.config/codex/codex-profiles.sh` # Helps loading the right model/key for the different profiles

### Structure

`~/.codex/config.toml` contains:

- shared defaults such as `personality` and `model_reasoning_effort`
- provider definitions under `[model_providers.<name>]` -- for this example, you need to define provider informations for the GPT UiO models (shared) and separate ones for models deployed on Azure.
- project trust settings (added later when you choose to trust folders)

Example shared config file:

```toml
# Optional: Define a default setup when no profile is chosen.
model = "gpt-5.4"
model_provider = "azure"
model_reasoning_effort = "medium"

[model_providers.gptuio]
name = "GPT UiO"
base_url = "https://gpt.uio.no/api/v1"
env_key = "YELLOW_UIO_API_KEY"
wire_api = "responses"

[model_providers.azure]
name = "Azure OpenAI"
base_url = "https://fdry-uio-mn-geo-geohyd-snowdepth.cognitiveservices.azure.com/openai/v1"
env_key = "AZURE_OPENAI_API_KEY"
wire_api = "responses"
```

Each profile cofig file should only contain the settings that differ. For the GPT UiO models, this is only the model name. For example gpt54uio.config.toml:

```toml
model = "gpt-5.4"
model_provider = "gptuio"
```

### Environment Variables
Keep API keys out of `config.toml` and store them in environment variables instead. This example requires two environments, one for GPT UiO (shared for all models there) and one for the Azure model. Even if you have further profiles, no extra env file is needed when the provider endpoint and API key are shared.


Example:

```bash
# ~/.config/codex/env/gptuio.env
export YELLOW_UIO_API_KEY='...'
```

Restrict permissions:

```bash
chmod 700 ~/.config/codex ~/.config/codex/env
chmod 600 ~/.config/codex/env/*.env
```

### Shell Wrappers
Wrappers source the matching env file and then start Codex with the matching profile.

Example:

```bash
codex-gpt54uio() {
  set -a
  . "$HOME/.config/codex/env/gptuio.env"
  set +a
  codex --profile gpt54uio "$@"
}
```

Load the wrappers in each shell, or add that line to `~/.bashrc`:

```bash
source ~/.config/codex/codex-profiles.sh
```


### Usage
Start Codex CLI with the profile name defined in the shell wrapper above. Examples:

```bash
codex-azure54
codex-gpt5miniuio
```

If you define a default model in config.toml, this will be loaded when you start codex without specifying a profile.
```bash
codex     # loads default model (if defined)
```


## Resources
- [LLM workshop tutorial from January 2026](https://lexnederbragt.github.io/dsc26-llm-code/tutorial.html)
- [Codex setup (agent-skolen for UiO)](https://pages.github.uio.no/alexajo/agent-skolen/setup_codex.html) (access requires UiO account login).
- [Information about UiO's GPT access / personal API access](https://www.uio.no/tjenester/it/ki/gpt-uio/)
- [Link to the Azure ordering process](https://www-int.uio.no/tjenester/it/ki/foundry/hjelp/bestill-api/api.html)
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
