# Complete Setup Guide: WSL, Ubuntu & Claude Code with Third-Party LLMs (Qwen / Gemini) on Windows

## Prerequisites

### Before you start, make sure you have:

> - **Windows Version:** Windows 10 (Build 2004+) or Windows 11  
> - **Note:** Node.js is not needed on Windows — we will install Node 20+ inside WSL

---

## Understanding the Basics (Important)

- **Linux** is the core technology (kernel) used by many operating systems.
- **Ubuntu** is a full operating system built on Linux.
- **WSL (Windows Subsystem for Linux)** allows Windows to run Linux-based operating systems like Ubuntu.
- When you open Ubuntu on Windows, you interact with it through a **terminal**, similar to CMD or PowerShell.


## Step 0: Install Windows Subsystem for Linux (WSL)

## 1️⃣ Enable WSL (Windows Subsystem for Linux):

### Open PowerShell as Administrator:   

#### Option A: Automatic (Recommended)  

**Run the following command:** 
```powershell
wsl --install
```

This will:
- Enable WSL
- Download Ubuntu 22.04 LTS by default
- Launch Ubuntu terminal for initial setup (username/password)

#### Option B: Manual

1. Open **Microsoft Store**
2. Search for **Ubuntu 22.04 LTS**
3. Click **Install**
4. Click **Launch**

---

### Note:
```text  
If Windows prompts for a restart, you must restart your PC
If there’s no prompt, you can continue without restarting.
```

---


## 2️⃣ 🔧 Launching Ubuntu Terminal:  
    
After `wsl --install`, the Ubuntu terminal usually opens automatically.

**If it does not open, you can manually launch it:**

### Option 1: From PowerShell
```powershell
wsl
```

### Option 2: From Start Menu
> Search "Ubuntu" → Click it

This opens your default Linux distribution (Ubuntu in this case)
### First-Time Ubuntu Setup:

When the Ubuntu terminal opens for the first time:

1. **Create a UNIX username** (when prompted)
2. **Set a password** (when prompted)

After this, you are inside the Ubuntu terminal, ready to install Node and other tools.

✅ **Ubuntu is now ready inside WSL**

✅ **After that, you are inside the Ubuntu terminal and ready to continue with Node, NVM, and Claude Code installation**

--- 

## Step 1: Install Node 20+ Using NVM (Inside Ubuntu)

### 1️⃣ **Why Node on Windows doesn’t matter**
```text
If Node.js is installed on Windows, it does not affect Ubuntu in WSL.
WSL is a separate Linux environment. Node must be installed inside Ubuntu.
So even if Node 18 is installed on Windows, WSL/Ubuntu won’t use it. That’s why you need Node 20+ inside Ubuntu.
```

### 2️⃣ How Node 20+ is installed inside Ubuntu (WSL)

We use NVM (Node Version Manager) to install Node inside Ubuntu.
Here are the commands from your setup:

- **Step 0 - Check Your Shell (Important)**

Run in terminal:
```bash
echo $SHELL
```

- **Step 1 — Install NVM**
```bash
curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash
```
- **Step 2 — Load NVM**
```bash
export NVM_DIR="$HOME/.nvm"
source "$NVM_DIR/nvm.sh"
```

- **Step 3 - Install Node 20 (LTS) using NVM:**
```bash
nvm install 20
```
- **Step 4 - Switch to use Node 20:**
```bash
nvm use 20
```

- **Step 5 — Verify Node**
```bash
node -v
npm -v
```

✅ Ensure Node version is 20+ before running Claude Code and CCR.

### Prerequisites  
- Qwen CLI installed and authenticated
- Node.js v18+ installed

**Step 1: Install Claude Code Router**
Install Claude Code & Router:
```bash
npm install -g @anthropic-ai/claude-code @musistudio/claude-code-router
```
**Verify Installation:**
```bash
ccr version
claude --version
```

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
First time you'll authenticate by using you `gmail account` or an `API Key`.

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
