# WhatsApp AI Chatbot Workflow (n8n)

An automated WhatsApp chatbot built entirely on **n8n**, capable of handling **text, audio, and image** messages. Incoming messages are classified, routed through an AI agent, and answered back in the format the user requested — text or voice.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture / Workflow Logic](#architecture--workflow-logic)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Environment Variables / Credentials](#environment-variables--credentials)
- [Usage](#usage)
- [Customization](#customization)
- [Troubleshooting](#troubleshooting)
- [Roadmap](#roadmap)
- [API & Resource Links](#api--resource-links)
- [Author](#author)

---

## Overview

This repository contains a complete n8n workflow (`whatsapp_chatbot.json`) that turns a WhatsApp Business number into an AI-powered assistant. The bot listens for incoming messages via the WhatsApp Trigger node, determines the message type, processes it accordingly (transcription for audio, vision analysis for images), passes the extracted content to an AI Agent node, and replies back to the user in the appropriate format.

![Workflow Screenshot](https://github.com/user-attachments/assets/abedff14-8b04-4bc2-b5f5-b138090c6df1)

---

## Features

- 📩 **Text message handling** — direct conversational responses via an LLM-powered AI agent.
- 🎙️ **Audio message support** — incoming voice notes are transcribed (e.g., via Whisper/OpenAI) before being processed.
- 🖼️ **Image message support** — images are analyzed/described using a vision-capable model before generating a response.
- 🔀 **Automatic input classification** — a routing node detects the message type and sends it down the correct processing branch.
- 🗣️ **Flexible output format** — replies can be sent back as plain text or converted to audio, depending on what the user requested or the workflow configuration.
- 🔧 **Fully visual, no-code workflow** — built entirely in n8n, so it's easy to inspect, extend, or fork.

---

## Architecture / Workflow Logic

```
WhatsApp Trigger
      │
      ▼
Message Type Classifier
      │
   ┌──┼──────────────┐
   ▼  ▼              ▼
 Text Audio         Image
   │   │              │
   │   ▼              ▼
   │  Transcription  Vision Analysis
   │   │              │
   └───┴──────┬───────┘
              ▼
         AI Agent Node
      (prompt generation +
        response logic)
              │
              ▼
     Format Response
     (Text or Audio)
              │
              ▼
      WhatsApp Send Node
```

1. **Trigger** — The WhatsApp Trigger node listens for new incoming messages (text, audio, or image attachments).
2. **Classification** — A Switch/If node inspects the message payload and routes it to the correct branch.
3. **Media Processing**
   - Audio → sent to a transcription node (e.g., OpenAI Whisper) to convert speech to text.
   - Image → sent to a vision/analysis node to generate a text description of the image content.
4. **Prompt Construction** — The extracted/transcribed text is merged into a prompt template for the AI Agent.
5. **AI Agent** — Generates a contextual response using the configured LLM (OpenAI, or any compatible provider).
6. **Response Formatting** — Depending on the requested/expected output, the reply is either sent as plain text or converted to audio (text-to-speech) before sending.
7. **Delivery** — The final response is sent back to the user via the WhatsApp Send node.

---

## Prerequisites

Before importing this workflow, make sure you have:

1. **n8n** — self-hosted or cloud instance (v1.0+ recommended).
2. **WhatsApp Business API access** — via Meta Cloud API, Twilio, or another n8n-supported WhatsApp integration.
3. **AI provider account** — OpenAI (or another compatible LLM provider) with an active API key.
4. **Audio transcription access** — if using Whisper via OpenAI, this is covered by the same API key; otherwise configure your chosen transcription service.
5. **Vision-capable model access** — for image analysis (e.g., GPT-4 Vision or equivalent).

---

## Installation

1. **Clone or download** this repository:
   ```bash
   git clone <repo-url>
   cd whatsapp-n8n-chatbot
   ```
2. **Open your n8n instance** and navigate to the **Workflows** section.
3. Click the **three-dot menu** (top right) → **Import from File**.
4. Select and upload `whatsapp_chatbot.json` from this repository.
5. Review each node once imported — some nodes will show a warning icon until credentials are attached (see [Configuration](#configuration) below).
6. Once credentials are set, click **Activate** to turn the workflow live.

---

## Configuration

### 1. WhatsApp Trigger Node
- Attach your WhatsApp credentials (Meta Cloud API token, phone number ID, webhook verification token, etc.).
- Set the webhook URL in your WhatsApp Business/Meta developer dashboard to point to your n8n webhook endpoint.

### 2. AI Agent Node
- Attach your OpenAI (or alternative LLM) API credentials.
- Adjust the system prompt to match your desired chatbot persona/tone.
- Configure model parameters (temperature, max tokens, etc.) as needed.

### 3. Audio Transcription Node
- Attach transcription service credentials (e.g., OpenAI Whisper).
- Confirm the input audio format matches what WhatsApp sends (typically `.ogg`/Opus) — add a conversion step if your transcription service requires a different format.

### 4. Image Analysis Node
- Attach vision-model credentials.
- Adjust the analysis prompt (e.g., "describe this image in detail" vs. "extract text from this image") based on your use case.

### 5. Response Formatting / Text-to-Speech Node
- If audio replies are enabled, attach TTS credentials and select a voice.
- Configure the logic that decides text vs. audio output (e.g., based on keywords, user preference, or always defaulting to text).

---

## Environment Variables / Credentials

| Credential | Used By | Required |
|---|---|---|
| WhatsApp API Token | WhatsApp Trigger / Send nodes | Yes |
| WhatsApp Phone Number ID | WhatsApp Trigger / Send nodes | Yes |
| Webhook Verify Token | WhatsApp Trigger | Yes |
| OpenAI API Key (or equivalent) | AI Agent, Transcription, Vision | Yes |
| Text-to-Speech API Key | Audio response formatting | Only if audio replies enabled |

> Store all credentials using n8n's built-in **Credentials Manager** — never hardcode API keys inside node parameters.

---

## Usage

Once activated:

1. Send a **text message** to your connected WhatsApp number → the bot classifies it, generates an AI response, and replies as text.
2. Send a **voice note** → the bot transcribes it, feeds the text to the AI agent, and replies (text or audio, per configuration).
3. Send an **image** → the bot analyzes the image content, generates a relevant response, and replies accordingly.

---

## Customization

- **Change the AI persona** — edit the system prompt inside the AI Agent node.
- **Add new input types** — extend the classifier node with additional branches (e.g., documents, location pins).
- **Multi-language support** — add a language-detection step before the AI Agent node and adjust prompts dynamically.
- **Logging/analytics** — insert a database or Google Sheets node after the AI Agent to log conversations.
- **Rate limiting** — add a wait/throttle node to prevent abuse from repeated rapid messages.

---

## Troubleshooting

| Issue | Likely Cause | Fix |
|---|---|---|
| Webhook not receiving messages | Incorrect webhook URL or verify token | Re-check Meta/WhatsApp dashboard webhook config |
| Audio transcription fails | Unsupported audio format | Add an audio-conversion node before transcription |
| AI Agent returns empty response | Missing/invalid API key or rate limit hit | Verify credentials and API usage limits |
| Image analysis errors out | Vision model not enabled on your API plan | Confirm your OpenAI (or provider) plan supports vision |
| Workflow not triggering | Workflow not activated | Toggle the **Active** switch in n8n |

---

## Roadmap

- [ ] Add support for document/PDF messages
- [ ] Add conversation memory/context persistence across sessions
- [ ] Add a fallback human-handoff flow
- [ ] Add multi-language auto-detection and response

---

## API & Resource Links

- [n8n Documentation](https://docs.n8n.io/)
- [WhatsApp Business Cloud API (Meta)](https://developers.facebook.com/docs/whatsapp/cloud-api)
- [Twilio WhatsApp API](https://www.twilio.com/docs/whatsapp)
- [OpenAI API Documentation](https://platform.openai.com/docs)
- [OpenAI Whisper (Speech-to-Text)](https://platform.openai.com/docs/guides/speech-to-text)
- [OpenAI Text-to-Speech](https://platform.openai.com/docs/guides/text-to-speech)
- [OpenAI Vision (Image Understanding)](https://platform.openai.com/docs/guides/vision)
- [n8n WhatsApp Trigger Node Docs](https://docs.n8n.io/integrations/builtin/trigger-nodes/n8n-nodes-base.whatsapptrigger/)
- [n8n AI Agent Node Docs](https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.agent/)

---

## Author

Built by **Jay Jogarajiya**.
