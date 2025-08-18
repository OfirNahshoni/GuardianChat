# Build AI WhatsApp Bots with Pure Python

This guide walks you through building a WhatsApp bot using the Meta (formerly Facebook) Cloud API with pure Python and Flask. We’ll configure webhooks to receive messages in real-time and integrate OpenAI for AI-powered responses.

---

## Prerequisites

1. A [Meta developer account](https://developers.facebook.com/).
2. A business app (create one [here](https://developers.facebook.com/docs/development/create-an-app/)).  
   > If you don’t see the business app option, select **Other > Next > Business**.
3. Python knowledge and a working environment.

---

## Table of Contents

- [Build AI WhatsApp Bots with Pure Python](#build-ai-whatsapp-bots-with-pure-python)
  - [Prerequisites](#prerequisites)
  - [Table of Contents](#table-of-contents)
  - [Get Started](#get-started)
  - [Step 1: Select Phone Numbers](#step-1-select-phone-numbers)
  - [Step 2: Send Messages with the API](#step-2-send-messages-with-the-api)
    - [Extend Token Lifetime](#extend-token-lifetime)
    - [Required Information](#required-information)
  - [Step 3: Configure Webhooks to Receive Messages](#step-3-configure-webhooks-to-receive-messages)
  - [Step 4: Understanding Webhook Security](#step-4-understanding-webhook-security)
      - [Verification Requests](#verification-requests)
      - [Validating Verification Requests](#validating-verification-requests)
      - [Validating Payloads](#validating-payloads)
  - [Step 5: Learn about the API and Build Your App](#step-5-learn-about-the-api-and-build-your-app)
  - [Step 6: Integrate AI into the Application](#step-6-integrate-ai-into-the-application)

---

## Get Started

1. Read the [WhatsApp Cloud API quickstart](https://developers.facebook.com/docs/whatsapp/cloud-api/get-started).
2. Find your bots [here](https://developers.facebook.com/apps/).
3. Read the [official documentation](https://developers.facebook.com/docs/whatsapp).
4. Reference: [Python guide](https://developers.facebook.com/blog/post/2022/10/24/sending-messages-with-whatsapp-in-your-python-applications/).
5. API Docs: [Send Messages](https://developers.facebook.com/docs/whatsapp/cloud-api/guides/send-messages).

---

## Step 1: Select Phone Numbers

- Add WhatsApp to your app.
- Use the provided test number to send messages to up to 5 numbers.
- Go to **API Setup** and select your test number.
- Add your own WhatsApp number — you’ll receive a verification code.

---

## Step 2: Send Messages with the API

1. Generate a 24-hour access token under **API Access**.
2. Use the given `curl` command (or Postman) to test sending a message.
3. Convert it to Python with the [requests library](start/whatsapp_quickstart.py).
4. Create a `.env` file (based on `example.env`) with required credentials.
5. You should receive a **Hello World** message (allow 1–2 minutes delay).

### Extend Token Lifetime
- Create a [System User](https://business.facebook.com/settings/system-users).
- Assign WhatsApp App with full control → Save changes.
- Generate a new token (60-day or never expire).
- Select **all permissions**.
- Copy token.

### Required Information
- **APP_ID** — from App Dashboard  
- **APP_SECRET** — from App Dashboard  
- **RECIPIENT_WAID** — your verified WhatsApp number  
- **VERSION** — e.g. `v19.0`  
- **ACCESS_TOKEN** — long-lived system user token  

⚠️ **Important**: The **first message must be a template type** ("hello_world"). Free-form messages only work after a user replies.

---

## Step 3: Configure Webhooks to Receive Messages  

1. Install requirements:  
   ```bash
   pip install -r requirements.txt
   ```
2. Run webhook server:  
   ```bash
   python run.py
   ```
3. Expose locally (e.g., with [ngrok](https://ngrok.com/)):  
   ```bash
   ngrok http 5000
   ```


## Step 4: Understanding Webhook Security

Below is some information from the Meta Webhooks API docs about verification and security. It is already implemented in the code, but you can reference it to get a better understanding of what's going on in [security.py](https://github.com/daveebbelaar/python-whatsapp-bot/blob/main/app/decorators/security.py)

#### Verification Requests

[Source](https://developers.facebook.com/docs/graph-api/webhooks/getting-started#:~:text=process%20these%20requests.-,Verification%20Requests,-Anytime%20you%20configure)

Anytime you configure the Webhooks product in your App Dashboard, we'll send a GET request to your endpoint URL. Verification requests include the following query string parameters, appended to the end of your endpoint URL. They will look something like this:

```
GET https://www.your-clever-domain-name.com/webhook?
  hub.mode=subscribe&
  hub.challenge=1158201444&
  hub.verify_token=meatyhamhock
```

The verify_token, `meatyhamhock` in the case of this example, is a string that you can pick. It doesn't matter what it is as long as you store in the `VERIFY_TOKEN` environment variable.

#### Validating Verification Requests

[Source](https://developers.facebook.com/docs/graph-api/webhooks/getting-started#:~:text=Validating%20Verification%20Requests)

Whenever your endpoint receives a verification request, it must:
- Verify that the hub.verify_token value matches the string you set in the Verify Token field when you configure the Webhooks product in your App Dashboard.
- Respond with the hub.challenge value.

#### Validating Payloads

[Source](https://developers.facebook.com/docs/graph-api/webhooks/getting-started#:~:text=int-,Validating%20Payloads,-We%20sign%20all)

WhatsApp signs all Event Notification payloads with a SHA256 signature and include the signature in the request's X-Hub-Signature-256 header, preceded with sha256=. You don't have to validate the payload, but you should.

To validate the payload:
- Generate a SHA256 signature using the payload and your app's App Secret.
- Compare your signature to the signature in the X-Hub-Signature-256 header (everything after sha256=). If the signatures match, the payload is genuine.

---

## Step 5: Learn about the API and Build Your App

Review the developer documentation to learn how to build your app and start sending messages.  
[See documentation here](https://developers.facebook.com/docs/whatsapp/cloud-api).

---

## Step 6: Integrate AI into the Application

Now that we have an end-to-end connection, we can make the bot a little more clever than just replying in uppercase. All you have to do is create your own `generate_response()` function in [whatsapp_utils.py](https://github.com/daveebbelaar/python-whatsapp-bot/blob/main/app/utils/whatsapp_utils.py).

If you want a ready-made example to integrate the OpenAI Assistants API with a retrieval tool, then follow these steps:

1. Watch this video: [OpenAI Assistants Tutorial](https://www.youtube.com/watch?v=0h1ry-SqINc)
2. Create your own assistant with OpenAI and update your `OPENAI_API_KEY` and `OPENAI_ASSISTANT_ID` in the environment variables.
3. Provide your assistant with data and instructions.
4. Update [openai_service.py](https://github.com/daveebbelaar/python-whatsapp-bot/blob/main/app/services/openai_service.py) to your use case.
5. Import `generate_response` into [whatsapp_utils.py](https://github.com/daveebbelaar/python-whatsapp-bot/blob/main/app/utils/).
6. Update `process_whatsapp_message()` with the new `generate_response()` function.
