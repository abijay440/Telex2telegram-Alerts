# Telegram Alert Integration for Telex

## Overview

The Telegram Alert Integration is an Output Integration for the Telex platform (telex.im). This integration is designed to route every incoming channel message from Telex to your target URL (hosted on Cloudflare Workers). The worker processes each message by checking for specific alert keywords and, if any are detected, it sends an alert to a designated Telegram chat via a bot.

Telex sends messages in a predetermined JSON format. This integration immediately acknowledges receipt of the payload and then processes the message internally—sending Telegram alerts when required.

## Integration Type

Output Integration:
Routes data from a Telex channel to an external service (Telegram). The integration does not provide additional feedback to Telex beyond an HTTP acknowledgment.

## Features

1. Real-Time Message Processing:
Receives every channel message from Telex at the target URL.

2. Keyword Alert Filtering:
Analyzes the incoming message (after stripping HTML) to check for predefined alert keywords.

3. Telegram Notification:
If a keyword is detected, the integration sends an alert message via Telegram using the Bot API.

4. Immediate Acknowledgment:
Responds to every incoming message with HTTP 200 OK, as required by Telex.

5. Configurable Settings:
All integration settings (bot token, chat ID, keywords, channel ID, and interval) are provided in the JSON payload from Telex.

6. Extensible and Lightweight:
Built as a Cloudflare Worker with minimal dependencies, making it fast and easy to deploy.

## JSON Payload Format

When Telex sends a channel message to your integration, it uses the following JSON format:

```json
{
  "message": "<p>testing help</p>",
  "settings": [
    {
      "default": "your-bot-token", // from botfather
      "label": "telegram_bot_token",
      "required": true,
      "type": "text"
    },
    {
      "default": "you_chat_id",// settings
      "label": "telegram_chat_id",
      "required": true,
      "type": "text"
    },
    {
      "default": "alert, warning, stop, help", // user defined keywords
      "label": "keywords",
      "required": true,
      "type": "text"
    },
  ]
}

```
Note: In this integration, the target_url is the base URL of your worker.

## How It Works
1. Message Reception:
Telex POSTs every channel message to your worker’s target URL. The payload includes the actual message (which may contain HTML) and an array of settings.

2. Settings Extraction:
The worker extracts integration settings from the settings array using a helper function. These settings include:
telegram_bot_token: Your Telegram Bot API token.
telegram_chat_id: The chat ID where alerts should be sent.
keywords: A comma-separated list of alert keywords.

3. Message Processing:
The worker strips HTML tags from the message to convert it to plain text. It then converts the text to lowercase and checks whether any of the specified keywords are present.

4. Telegram Alert:
If a keyword is detected, the worker sends an alert to Telegram using the Bot API.

5. Acknowledgment:
The worker immediately responds with HTTP 200 OK to acknowledge receipt of the payload. Telex does not expect additional feedback.

## Setup Instructions
### Prerequisites
#### Cloudflare Account & Workers: 
Sign up for a Cloudflare account and set up Cloudflare Workers.
#### Wrangler CLI:
Install the Wrangler CLI for local development and deployment.
#### Telegram Bot:
Create a Telegram bot using BotFather and obtain your bot token. Ensure the target chat is active and the bot has permissions to send messages.Obtain your chat ID from the chatbot history using a curl request to the link below
```
       https://api.telegram.org/bot<your-bot-token>/getUpdates
```
## Local Development

1. clone the repository:
Install dependencies and use Wrangler to run your worker locally:

```sh
       npm install
       npx wrangler dev
```
This will typically run on http://localhost:8787.

2. Optional – Expose via ngrok:
To test externally:

Install ngrok.
Run:
```sh
       ngrok http 8787
```
Update your integration JSON (returned by GET) to use the ngrok URL as your app_url and target_url.

## Testing
1. local testing 

```sh
curl -X POST -H "Content-Type: application/json" -d '{
  "message": "<p>testing help</p>",
  "settings": [
    {
      "default": "your-bot-token",
      "label": "telegram_bot_token",
      "required": true,
      "type": "text"
    },
    {
      "default": "your-telegram-chat or group-id",
      "label": "telegram_chat_id",
      "required": true,
      "type": "text"
    },
    {
      "default": "alert, warning, stop, help",
      "label": "keywords",
      "required": true,
      "type": "text"
    },
  ]
}' "https://your-worker-domain.workers.dev/"

```
2. Integration Testing
sign into telex.im
Go to app section
Add an application and enter your worker url
Go to setting and enter you bot token, alert words and chat ID
activate the application
you recieve a  message anytime the words are used in the channel


## Screenshots





## Troubleshooting

1. Telegram API 401 Unauthorized:
Verify the bot token and chat ID. Ensure the target chat has activated the bot.
2. No Alert Triggered:
Confirm that the message (after stripping HTML) contains one of the alert keywords in a case-insensitive manner.
3. Integration Not Installed Correctly:
Ensure the integration JSON is hosted at a publicly accessible URL and that Telex is configured to use your worker’s URL.
4. Ensure the right bot token and chat id are set

## Deployment
Deploy the Worker: Publish your worker using Wrangler:

```sh
      npx wrangler publish
```

This assigns your worker a public URL (e.g., https://your-worker-domain.workers.dev).

## Contributing

Pull requests are welcome. For major changes, please open an issue first
to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License

[MIT](https://choosealicense.com/licenses/mit/)

## Author
Abiodun Jegede
