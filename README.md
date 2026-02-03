# AI Voice Bot API Documentation v1.0
## Global Standard Response Schema

All endpoints return the same JSON structure.

*Success Schema:*
```
{
  "success": true,
  "status": 200,
  "message": "SUCCESS",
  "data": [
    {
      "id": 1,
      "item_name": "Example Data Object",
      "details": "Additional information goes here"
    }
  ],
  "errors": null
}
```
*Error Schema:*
```
{
  "success": false,
  "status": 422,
  "message": "VALIDATION_FAILED",
  "data": null,
  "errors": [
    {
      "loc": [
        "body",
        "exophone"
      ],
      "msg": "invalid phone number format",
      "type": "value_error.invalid"
    }
  ]
}
```
### Authorization Header
### All requests to protected routes must include the following header:
*Authorization: Bearer YOUR_ACCESS_TOKEN*

Token Lifecycle
    - Access Token: Short-lived (e.g., 15–60 minutes). Used for every API request.
    - Refresh Token: Long-lived (e.g., 7–30 days). Used only to obtain a new Access Token once the current one expires.

---

### Common Audit & Status Fields
Every database instance created (e.g., Prompts, Sessions, Bots, SMS, Campaigns) will include these standard fields to track changes and handle soft deletes.

| Field Name   | Data Type | Description |
|-------------|----------|-------------|
| created_by  | Int      | The ID of the user who created the record |
| modified_by | Int      | The ID of the user who last updated the record |
| created_at  | DateTime | Timestamp when the record was first created |
| modified_at | DateTime | Timestamp of the most recent update |
| is_active   | Boolean  | Flag to indicate if the record is currently usable in live campaigns |
| is_deleted  | Boolean  | Flag used for soft deletes to keep data for historical audit purposes |

---

### Login (Obtain Tokens)
*POST | api/v1/auth/login*

**Payload:**
```
{
  "username": "user",
  "password": "secure_password_123"
}
```

**Success Response:**
```
{
  "success": true,
  "status": 200,
  "message": "LOGIN_SUCCESSFUL",
  "data": [
    {
      "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "refresh_token": "def8806ad2432_refresh_v1...",
      "token_type": "Bearer",
      "expires_in": 3600
    }
  ],
  "errors": null
}
```
---

### Refresh Token
*POST | api/v1/auth/refresh*

Generates a new Access Token without requiring the user to log in again.

**Payload:**
```
{
  "refresh_token": "def8806ad2432_refresh_v1..."
}
```

**Success Response:**
```
{
  "success": true,
  "status": 200,
  "message": "TOKEN_REFRESHED",
  "data": [
    {
      "access_token": "new_eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "token_type": "Bearer",
      "expires_in": 3600
    }
  ],
  "errors": null
}
```

### Logout / Revoke
*POST | api/v1/auth/logout*

Blacklist the current Refresh Token to end the session.

**Payload:**
```
{
  "refresh_token": "def8806ad2432_refresh_v1..."
}

```

**Success Response:**
```
{
  "success": true,
  "status": 200,
  "message": "LOGOUT_SUCCESSFUL",
  "data": [],
  "errors": null
}
```
---

## Module 1: Prompt & Voice Management
### Manage the "scripts" and configuration the bot will use.

| sr no | Endpoint Name | Method | URL | Description |
| :--- | :--- | :--- | :--- | :--- |
| 1 | List Prompts | GET | `api/v1/prompts` | Retrieves all available system prompts. |
| 2 | Get Prompt | GET | `api/v1/prompts/{id}` | Fetches details of a specific prompt version. |
| 3 | Generate Prompt | POST | `api/v1/generatePrompt` | Generate a prompt for review. |
| 4 | Create/Update Prompt | POST | `api/v1/prompts` | Creates a new bot script/instruction. |
| 5 | List Sessions | GET | `api/v1/sessions` | Lists all available and default Session configuration. |
| 6 | Get Session | GET | `api/v1/sessions/{id}` | Fetches a specific Session by ID. |
| 7 | Create/Update Session | POST | `api/v1/sessions` | Saves a new Session. |

---

### 1. List Prompts
*GET | baseurl/api/v1/prompts*

**Response:**
```
{
  "success": true,
  "status": 200,
  "message": "SUCCESS",
  "data": [
    {
      "id": 23,
      "prompt_id": "fbe806ad2432",
      "previous_id": null,
      "prompt_name": "Collection_Reminder_V1",
      "text": "# Personality and Tone\n## Identity\nYou are Tanya, a female AI voicebot representing Mahindra Finance. You are professional, helpful, and empathetic. Your goal is to guide existing customers through loan qualification."
    }
  ],
  "errors": null
}
```


### 2. Get Prompt
*GET | baseurl/api/v1/prompt/fbe806ad2432*

**Response:**
```
{
  "success": true,
  "status": 200,
  "message": "SUCCESS",
  "data": [
    {
      "id": 23,
      "prompt_id": "fbe806ad2432",
      "previous_id": null,
      "prompt_title": "Collection_Reminder_V1",
      "content": "# Personality and Tone\n## Identity\nYou are Tanya, a female AI voicebot representing Mahindra Finance. You are professional, helpful, and empathetic. Your goal is to guide existing customers through loan qualification."
    }
  ],
  "errors": null
}
```

### 3. Generate Prompt
*POST | baseurl/api/v1/generatePrompt*

**Payload:**
```
{
  "prompt_title": "Collection_Reminder_V1",
  "text": “make a prompt reminding users of their upcoming EMI due date.”,
  “
...
}
```

**Response:**
```
{
  "success": true,
  "status": 200,
  "message": "SUCCESS",
  "data": [
    {
      "prompt_title": "Collection_Reminder_V1",
      "content": "# Personality and Tone\n## Identity\nYou are Tanya, a female AI voicebot representing Mahindra Finance. You are professional, helpful, and empathetic. Your goal is to remind customers of their upcoming EMI."
    }
  ],
  "errors": null
}
```

### 4. Confirm & Save Prompt
*POST | baseurl/api/v1/prompt*

***Note: All the update will be a new prompt instance for maintaining the history.***

**Payload:**
```
{
  "prompt_title": "Collection_Reminder_V1",
  "content": 
# Personality and Tone
## Identity
You are Tanya, a female AI voicebot representing Mahindra Finance. You are professional, helpful, and empathetic. Your goal is to remind customers of their upcoming EMI.
...
}
```

### 5. List Sessions (There will always be 2 default session)
*POST | baseurl/api/v1/session*

**Response:**
```
{
  "success": true,
  "status": 200,
  "message": "SUCCESS",
  "data": [
    {
      "id": 32,
      "session_id": "fbe806ad2432",
      "previous_id": null,
      "turn_detection": {
        "type": "azure_semantic_vad",
        "threshold": 0.5,
        "prefix_padding_ms": 50,
        "silence_duration_ms": 100,
        "remove_filler_words": true
      },
      "input_audio_noise_reduction": {
        "type": "azure_deep_noise_suppression"
      },
      "input_audio_echo_cancellation": {
        "type": "server_echo_cancellation"
      },
      "voice": {
        "name": "en-IN-Meera:DragonHDIndicLatestNeural",
        "type": "azure-standard",
        "temperature": 0.3,
        "prefer_locales": [
          "hi-IN"
        ]
      }
    }
  ],
  "errors": null
}
```

### 6. Get Session
*GET | baseurl/api/v1/session/fbe806ad2432*

**Response:**
```
{
  "success": true,
  "status": 200,
  "message": "SUCCESS",
  "data": [
    {
      "id": 32,
      "session_id": "fbe806ad2432",
      "previous_id": null,
      "turn_detection": {
        "type": "azure_semantic_vad",
        "threshold": 0.5,
        "prefix_padding_ms": 50,
        "silence_duration_ms": 100,
        "remove_filler_words": true,
        "end_of_utterance_detection": {
          "model": "semantic_detection_v1",
          "threshold": 0.1,
          "timeout": 1.0
        }
      },
      "input_audio_noise_reduction": {
        "type": "azure_deep_noise_suppression"
      },
      "input_audio_echo_cancellation": {
        "type": "server_echo_cancellation"
      },
      "voice": {
        "name": "en-IN-Meera:DragonHDIndicLatestNeural",
        "type": "azure-standard",
        "temperature": 0.3,
        "prefer_locales": [
          "hi-IN"
        ]
      }
    }
  ],
  "errors": null
}
```
---
### 7. Add Update Session Config
*POST | baseurl/api/v1/prompt*

***Note-1: If id is provided in the payload then update else new.***

***Note-2: All the update will be a new session config for maintaining the history so that we can go back and debug on which session it worked best. There is a flag kept for the same “previous_id”.***

**Payload:**
```
{
  "id": 32,
  "session_id": "fbe806ad2432",
  "previous_id": 19,
  "turn_detection": {
       "type": "azure_semantic_vad",
       "threshold": 0.5,
       "prefix_padding_ms": 50, 
       "silence_duration_ms": 100, 
       "remove_filler_words": True,
       "end_of_utterance_detection": {
       "model": "semantic_detection_v1",
       "threshold": 0.1,
       "timeout": 1.0,
       },
},
  "input_audio_noise_reduction": {"type": "azure_deep_noise_suppression"},
  "input_audio_echo_cancellation": {"type": "server_echo_cancellation"},
  "voice": {
      # "name": "hi-IN-ArjunNeural",
      "name": "en-IN-Meera:DragonHDIndicLatestNeural",
      "type": "azure-standard",
      "temperature": 0.3,
      "prefer_locales": ["hi-IN"],
      },
}
```

### Module 2: Bot Identity Settings
***Configuring the different components into one bot.***
| Sr No | Endpoint Name | Method | URL | Description |
|------:|--------------|--------|-----|-------------|
| 8 | List Bots | GET | `/api/v1/bots` | Lists all defined Bot Profiles |
| 9 | Get Bot | GET | `/api/v1/bot/{id}` | Returns the full Bot definition |
| 10 | Create Bot | POST | `/api/v1/bot` | Combines: Prompt, Voice, and SMS into an ID |
---

### 8. List Bots
*GET | baseurl/api/v1/bots*

**Response:**
```
{
  "success": true,
  "status": 200,
  "message": "SUCCESS",
  "data": [
    {
      "agent_id": "fbe806ad2431",
      "agent_name": "Vaani",
      "prompt_id": "fbe806ad2423",
      "session_id": "fbe806ad6932",
      "sms_id": "fbe806ad9032"
      "previous_id": null,
    },..
  ],
  "errors": null
}
```

### 9. Get Bot
*GET | baseurl/api/v1/bots*

**Response:**
```
{
  "success": true,
  "status": 200,
  "message": "SUCCESS",
  "data": [
    {
      "agent_id": "fbe806ad2431",
      "agent_name": "Vaani",
      "prompt_id": "fbe806ad2423",
      "session_id": "fbe806ad6932",
      "sms_id": "fbe806ad9032"
      "previous_id": null,
    }
  ],
  "errors": null
}
```
---

### 10. Add Update Session Config (One file field to upload example csv for validation in campaign)
*POST | baseurl/api/v1/bot*

**Payload:**
```
{
  "id": 321,
  "agent_id": "fbe806ad2431",
  "agent_name": "Vaani",
  "agent_gender": "M",
  "prompt_id": "fbe806ad2423",
  "session_id": "fbe806ad6932",
  "sms_id": "fbe806ad9032"
}
```
--- 

### Module 3: SMS Management
***Manage the SMS templates that are triggered automatically after a call concludes.***

| Sr No | Endpoint Name | Method | URL | Description |
|------:|--------------|--------|-----|-------------|
| 11 | List SMS | GET | `/api/v1/sms` | Retrieves all available SMS templates |
| 12 | Get SMS | GET | `/api/v1/sms/{id}` | Fetches details of a specific SMS template |
| 13 | Create/Update SMS | POST | `/api/v1/sms` | Creates a new SMS template or updates existing one |

### 11. List SMS
*GET | baseurl/api/v1/sms*

**Response:**
```
{
  "success": true,
  "status": 200,
  "message": "SUCCESS",
  "data": [
    {
      "id": 101,
      "sms_id": "sms_8823ad2432",
      "template_id": "DLT_MMFSL_01",
      "sms_title": "EMI_Reminder_SMS",
      "content": "Dear Customer, your EMI of INR {amount} is due for your loan {loan_no}. Click here to pay: {link} - Mahindra Finance",
      "previous_id": null
    }
  ],
  "errors": null
}
```
---
### 12. Get SMS
*GET | baseurl/api/v1/sms/8823ad2432*

**Response:**
```
{
  "success": true,
  "status": 200,
  "message": "SUCCESS",
  "data": [
    {
      "id": 101,
      "sms_id": "sms_8823ad2432",
      "template_id": "DLT_MMFSL_01",
      "sms_title": "EMI_Reminder_SMS",
      "content": "Dear Customer, your EMI of INR {amount} is due for your loan {loan_no}. Click here to pay: {link} - Mahindra Finance",
      "previous_id": null
    }
  ],
  "errors": null
}
```
---

### 13. Add / Update SMS Config
*POST | baseurl/api/v1/sms*

***Note-1: If id is provided in the payload, the system creates a new version linked via previous_id.***

***Note-2: This ensures a complete audit trail of what SMS text was sent during specific campaign runs.***

**Payload:**
```
{
  "id": 101,
  "sms_id": "sms_8823ad2432",
  "previous_id": 88,
  "template_id": "DLT_MMFSL_01",
  "sms_title": "EMI_Reminder_SMS",
  "content": "Dear Customer, your EMI of INR {amount} is due for your loan {loan_no}. Click here to pay: {link} - Mahindra Finance"
}
```

**Response:**
```
{
  "success": true,
  "status": 200,
  "message": "SMS template updated successfully",
  "data": [
    {
      "id": 102,
      "sms_id": "sms_8823ad2432",
      "previous_id": 101,
      "sms_title": "EMI_Reminder_SMS"
    }
  ],
  "errors": null
}
```
---

### Module 4: Campaign & Data Management
***Handling the customer base (CSV) and the execution triggers for automated dialing.***

| Sr No | Endpoint Name | Method | URL | Description |
|------:|--------------|--------|-----|-------------|
| 14 | List Campaigns | GET | `/api/v1/campaigns` | Lists all outbound campaigns |
| 15 | Get Campaign | GET | `/api/v1/campaign/{id}` | Returns metadata, status, and assigned Exophone |
| 16 | Create Campaign | POST | `/api/v1/campaign` | Combines: Bot ID + Exophone + Schedule |
| 17 | Upload Base Data | POST | `/api/v1/campaign/data` | Uploads CSV records for a campaign |
| 18 | Start Campaign | POST | `/api/v1/campaign/run` | Triggers the ThreadPool loop (eg: 100 calls/min) |

### 16. Create Campaign 
*POST | baseurl/api/v1/campaign*

***Note: This will bind technical "Caller ID" (Exophone) to the campaign.***

**Payload:**
```
{
  "campaign_title": "Feb_EMI_Reminders",
  "agent_id": "fbe806ad2431",
  "exophone": "8068874370",
  "exotel_app_id": "31646",
  "scheduled_start": "2026-02-05 09:00:00",
  "batch_size": 15
}
```
---

### 17. Upload Base Data
*POST | baseurl/api/v1/campaign/data*

**Payload (Multipart):**
    • campaign_id: 8823ad2432
    • file: base_data.csv (contains serial_no, customer_name, mobile_no)




