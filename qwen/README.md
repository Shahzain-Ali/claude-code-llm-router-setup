# Complete Setup Guide: Claude Code with Third-Party LLMs (Qwen) on Windows

### Prerequisites
- Node.js v20+ installed (inside Ubuntu / WSL)
- WSL & Ubuntu terminal ready

---

**Step 0: Install Claude Code Router**
Install Claude Code & Router:
```bash
npm install -g @anthropic-ai/claude-code @musistudio/claude-code-router
```
**Verify Installation:**
```bash
ccr version
claude --version
```

**Step 1: Qwen CLI installed globally and authenticated**  
```bash
npm install -g qwen
```

Verify Qwen CLI version:

```bash
qwen --version
```
Run `qwen` in the terminal you'll see

│ Get started                                                                                                           │
│                                                                                                                       │
│ How would you like to authenticate for this project?                                                                  │
│                                                                                                                       │
│ ● 1. Qwen OAuth                                                                                                       │
│   2. OpenAI                                                                                                           │
│              

Authenticate yourself using your `Gmail account` or an `API Key`.

**Step 2: Extract Your Access Token**  
Replace  `PC_USER` with your Windows username.

Open `C:\Users\PC_USER\.qwen\oauth_creds.json:`

It should look something like this
```json
{
  "access_token": "YOUR_QWEN_ACCESS_TOKEN_HERE",
  "token_type": "Bearer",
  "refresh_token": "YOUR_QWEN_REFRESH_TOKEN_HERE",
  "resource_url": "portal.qwen.ai",
  "expiry_date": 1764876220290
}
```

**Step 3: Create the Folders**  
Paste this into WSL terminal:
```bash
mkdir -p ~/.claude-code-router ~/.claude
```

**Step 4: Create the Config File**  
Paste this command to create and populate the config file.  

`"api_key": "YOUR_QWEN_ACCESS_TOKEN_HERE"` add your access token here then run this in your terinal.

```bash
cat > ~/.claude-code-router/config.json << 'EOF'
{  
  "LOG": true,  
  "LOG_LEVEL": "info",  
  "HOST": "127.0.0.1",  
  "PORT": 3456,  
  "API_TIMEOUT_MS": 600000,  
  "Providers": [  
    {  
      "name": "qwen",  
      "api_base_url": "https://portal.qwen.ai/v1/chat/completions",  
      "api_key": "YOUR_QWEN_ACCESS_TOKEN_HERE",  
      "models": [  
        "qwen3-coder-plus",  
        "qwen3-coder-plus",  
        "qwen3-coder-plus"  
      ]  
    }  
  ],  
  "Router": {  
    "default": "qwen,qwen3-coder-plus",  
    "background": "qwen,qwen3-coder-plus",  
    "think": "qwen,qwen3-coder-plus",  
    "longContext": "qwen,qwen3-coder-plus",  
    "longContextThreshold": 60000,  
    "webSearch": "qwen,qwen3-coder-plus"  
  }  
}
EOF
```

**Step 5: Start Using**  
Restart the router server:

```bash
ccr restart
```

Run Claude Code with Qwen models:

```bash
ccr code
```

Use these commands to verify your setup:

`ccr status` — Check if the background service is running.

**Test:**

To check qwen model is working successfully or not send message:
> hi

You'll get reply by model successfully and when the session is expired after specific timeframe then you get an error `401` so you are required to refresh your access token.

**Step 6: Refresh token after session expired**  
- Refresh it by deleting the `.qwen` folder from your root directory.  
- Then run `qwen` command in terminal and to authenticate your self again from another google account and in your root directory the .qwen folder is automatically created and again you'll get `access token` from there. 
- Update the api_key in your `config.json` by using the same `Step 4` method.
- Restart: using this command `ccr restart`
- Run this command `ccr code` to run claude-code with Qwen models.
- To check which model of qwen is connected with claude-code run `ccr model`.   
--- 
### Maintained by Shahzain Ali | [LinkedIn Profile](https://www.linkedin.com/in/shahzain-ali-518b862ba/)