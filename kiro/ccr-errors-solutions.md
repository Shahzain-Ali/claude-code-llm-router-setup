# CCR Service Start Problem - Solution Guide
**Roman Urdu Mein**

## Error-1: Port Already in Use

### Problem

Agar CCR Ubuntu terminal mein chal raha ho aur aap usko stop nahi karte, to Windows terminal `CMD` mein error aata hai.

### Ubuntu Terminal Output

**Pehli baar start karne par:**
```bash
agentive-solution@DESKTOP-PNAKLV9:/mnt/d/agentic-todo-evolution$ ccr start 
Loaded JSON config from: /home/agentive-solution/.claude-code-router/config.json
```

**Multiple times par:**
```bash
claude-code-router server is running
```

**Restart karne par:**
```bash
agentive-solution@DESKTOP-PNAKLV9:/mnt/d/agentic-todo-evolution$ ccr restart 
claude code router service has been stopped. 
Starting claude code router service... ✅ 
Service started successfully in the background.
```

**Status check:**
```bash
agentive-solution@DESKTOP-PNAKLV9:/mnt/d/agentic-todo-evolution$ ccr status 
📊 Claude Code Router Status 
════════════════════════════════════════ 
✅ Status: Running 
🆔 Process ID: 3650 
🌐 Port: 3456 
📡 API Endpoint: http://127.0.0.1:3456 
📄 PID File: /home/agentive-solution/.claude-code-router/.claude-code-router.pid 
🚀 Ready to use! Run the following commands:    
   ccr code    # Start coding with Claude    
   ccr stop    # Stop the service
```

### Windows Terminal Error

Jab aap Windows terminal `CMD` mein `ccr start` aur `ccr status` run karte ho:

![Status-Not-Running](/images/Status-Not-Running.png "The title for the image")

### Solution

Matlab port busy hai. Pehle WSL2 (Ubuntu terminal) mein Claude Code Router stop karo:

```bash
ccr stop
```

Phir `ccr start` aur `ccr status` run karo. Agar Claude Code Router running ho to fine hai, nahi to restart karo:

```bash
ccr restart
```

Phir status check karo:

```bash
ccr status
```

### Problem Kya Thi?

Jab `ccr start` command chalaya to service start nahi hui. Error message mein sirf "Loaded JSON config" dikhaya aur phir exit code 1 ke saath fail ho gaya.

`ccr status` check kiya to "Not Running" dikh raha tha, lekin `ccr start` karne pe bhi service shuru nahi ho rahi thi.

### Problem Ki Wajah (Reason)

Port 3456 pe ek purana process "wslrelay.exe" chal raha tha.

Yeh isliye hua kyunki:
- Pehle ccr ko WSL Ubuntu terminal se start kiya gaya tha
- WSLRelay.exe ek helper process hai jo Windows aur WSL ke beech network traffic forward karta hai
- Jab ccr band hua ya crash hua, wslrelay ne port 3456 release nahi kiya
- Is wajah se naya ccr instance us port ko use nahi kar saka

**Simple words mein:** Darwaza (port 3456) pehle se kisi aur ne rok rakha tha, isliye ccr andar nahi ja saka.

---

## Error-2: Config File Exist Hi Nahi Karta Tha!

### Check Karne Ka Command

```bash
cat ~/.claude-code-router/config.json
```

### Case 1: File Delete Ho Gayi

```bash
agentive-solution@DESKTOP-PNAKLV9:/mnt/d/agentic-todo-evolution$ cat ~/.claude-code-router/config.json
cat: /home/agentive-solution/.claude-code-router/config.json: No such file or directory
```

Ye tab hota hai jab manually file delete kar di ho:
```bash
rm ~/.claude-code-router/config.json
```

### Case 2: File Khali/Incomplete Hai

```bash
agentive-solution@DESKTOP-PNAKLV9:~$ cat ~/.claude-code-router/config.json
{
  "PORT": 3456,
  "Providers": [],
  "Router": {}
}
```

Ye tab hota hai jab ek baar ccr chalaya ho jaise `ccr status`.

### Is Output Se Ye Error Milta Hai

![400-Error-Missing-Model-In-Request-Body](/images/400-missing-model-error.png "The title for the image")

### Solution

Ubuntu terminal mein ye configuration paste karo:

```bash
cat > ~/.claude-code-router/config.json << 'EOF'
{
  "LOG": true,
  "LOG_LEVEL": "debug",
  "Providers": [
    {
      "name": "kiro",
      "api_base_url": "http://localhost:8000/v1/chat/completions",
      "api_key": "my-super-secret-password-123",
      "models": [
        "claude-sonnet-4-5",
        "claude-haiku-4-5",
        "claude-opus-4-5"
      ],
      "transformer": {
        "use": ["openrouter"]
      }
    }
  ],
  "Router": {
    "default": "kiro,claude-sonnet-4-5",
    "think": "kiro,claude-sonnet-4-5",
    "background": "kiro,claude-sonnet-4-5",
    "longContext": "kiro,claude-sonnet-4-5",
    "webSearch": "kiro,claude-sonnet-4-5"
  }
}
EOF
```

### Verify Karo

Config.json file ke andar properties check karne ke liye:

```bash
nano ~/.claude-code-router/config.json
```

### CCR Start Karo

```bash
ccr start
```

### Status Check Karo

```bash
ccr status
```

### Claude Code Start Karo

```bash
ccr code
```

### Result

Message `Hi` bhejne ke baad aap notice karoge ki previous error `400-Error-Missing-Model-In-Request-Body` resolve ho gaya hai. Lekin aapko ek aur error milega.

---

## Error-3: Connection Failed (500 Error)

### Error Screenshot

![500-Error-Fetch-FailedTypeError](/images/500-Error-Fetch-FailedTypeError.png "The title for the image")

### Pehle Kya Kiya

Root directory `kiro-openai-gateway` (jahan actual kiro repository clone ki thi) mein terminal mein ye run kiya:

```bash
python main.py
```

Server start ho gaya. Phir claude code CLI stop kiya aur restart kiya:

```bash
ccr restart
```

Phir dobara claude code start kiya:

```bash
ccr code
```

### Phir Bhi Error

![500-Error-Fetch-FailedTypeError](/images/500-Error-Fetch-FailedTypeError.png "The title for the image")

Ye error isliye aaya kyunki WSL apna localhost use kar raha tha.

### Solution

Localhost ko Windows IP Address se replace karo config.json file mein Ubuntu terminal se.

### Steps To Resolve

**Ubuntu terminal mein ye commands run karo:**

#### 1. Connection Diagnose Karo

```bash
curl http://localhost:8000
```
Failed - confirmed localhost doesn't work

#### 2. Windows Host IP Find Karo WSL Se

```bash
ip route show | grep -i default | awk '{ print $3}'
```
Output kuch aisa milega: `172.x.x.x`

#### 3. Verify Karo Kiro Gateway Reachable Hai

```bash
curl http://172.x.x.x:8000
```
Success: `{"status":"ok","message":"Kiro Gateway is running"}`

#### 4. CCR Config File Locate Karo

```bash
find ~ -name ".claude-code-router" -type d
```
Found: `/home/agentive-solution/.claude-code-router/`

#### 5. Configuration Update Karo

```bash
nano ~/.claude-code-router/config.json
```

#### 6. Save Aur Exit Karo

```
Ctrl + o → Save
Enter → Confirm
Ctrl + x → Exit
```

#### 7. Configuration Change Karo

```json
{
  "apiKey": "sk-dummy-key-12345",
  "baseURL": "http://172.x.x.x:8000/v1/chat/completions"
}
```

#### 8. CCR Restart Karo

```bash
ccr restart
ccr code
```

---

**End of Guide**


📌 PROBLEM SOLVE KARNE KE STEPS
--------------------------------------------------------------------------------

First run this `ccr start` then check:

Step 1: Check karo port 3456 kaun use kar raha hai
        Command: netstat -ano | findstr :3456

Step 2: Process ID (PID) note karo jo port use kar raha hai
        Example output: TCP 127.0.0.1:3456 ... LISTENING 13348
        (Yahan 13348 PID hai)

Step 3: Check karo yeh kaun sa process hai
        Command: tasklist | findstr 13348

Step 4: Us process ko kill karo
        Command (CMD mein): taskkill /PID 13348 /F

Step 5: Ab ccr start karo
        Command: ccr start

Step 6: Verify karo service chal rahi hai
        Command: ccr status


📊 Quick Reference Table
TaskWindows (CMD)Ubuntu/WSL (Bash)Port checknetstat -ano | findstr :3456sudo lsof -i :3456Process listtasklist | findstr <PID>ps aux | grep <PID>Kill processtaskkill /PID <PID> /Fkill -9 <PID>CCR startccr startccr startCCR statusccr statusccr status


📌 FUTURE MEIN IS PROBLEM SE BACHNE KE LIYE (SUGGESTIONS)
--------------------------------------------------------------------------------
1. EK JAGAH USE KARO:
   - Ya sirf Windows CMD mein ccr chalao
   - Ya sirf WSL Ubuntu mein
   - Dono jagah mat chalao (port conflict hoga)

2. PROPERLY BAND KARO:
   - Hamesha "ccr stop" use karo band karne ke liye
   - Terminal seedha close mat karo jab ccr chal raha ho

3. WSL SE WINDOWS PE SWITCH KARTE WAQT:
   - Pehle WSL mein: ccr stop
   - Phir Windows mein: wsl --shutdown
   - Ab Windows mein: ccr start

4. MERI RECOMMENDATION:
   - Windows CMD use karo - cleaner hai
   - WSLRelay wali problems nahi aayengi


📌 AGAR DOBARA YAHI PROBLEM AAYE TO KYA KARO?
--------------------------------------------------------------------------------
Quick Fix Commands (Windows CMD/PowerShell mein):

1. Port check karo:
   netstat -ano | findstr :3456

2. Agar koi process mil jaye to uski PID note karo aur kill karo:
   taskkill /PID [PID_NUMBER] /F

3. Ya shortcut - poora WSL restart karo:
   wsl --shutdown

4. Phir ccr start karo:
   ccr start

==============================================================================

Localhost computer ka apna address hota hai, aur Windows aur WSL aksar isi localhost ko share karte hain.

Localhost zyada tar same hota hai, lekin jab service ghalat jagah bind ho ya VM-style networking ho, to error aa jata hai.
Means jab hamare configuration set na ho claude code router ki tho ye problem aata hai.

WSL mein localhost = WSL ka apna network (Windows nahi!)

✅ Ab Dono Jagah Kyun Chal Raha Hai?
WSL (Ubuntu) - Ab chal raha hai kyunki:
bash# Humne config fix kiya
~/.claude-code-router/config.json
# Ab ye Windows ka IP use kar raha hai: http://172.x.x.x:8000
Ab WSL → Windows ka IP → Kiro Gateway ✅
Windows (CMD) - Chal raha hai kyunki:
cmdC:\Users\TechLink\.claude-code-router\config.json
# Ye localhost use karta hai: http://localhost:8000
Windows → localhost → Kiro Gateway (same computer) ✅

(Error wala):

WSL se: "Hello Kiro, kahan ho?" → localhost (WSL ka) → Koi nahi mila ❌
Kiro Windows mein baitha tha, WSL ke localhost mein nahi tha.


(Working):

WSL se: "Hello Kiro, kahan ho?" → Windows ka IP → Kiro mil gaya ✅
Windows se: "Hello Kiro, kahan ho?" → localhost (Windows ka) → Kiro mil gaya ✅


Another same Error in Ubuntu:

Problem Summary & Resolution Guide
🔴 What Was The Problem?
Error: fetch failed - Claude Code Router (CCR) couldn't connect to the Kiro Gateway server.
Root Cause: Network connectivity issue between WSL (Windows Subsystem for Linux) and Windows. CCR was trying to connect to localhost:8000, but when running from WSL, localhost refers to the WSL network interface, NOT the Windows host where Kiro Gateway was running.

🤔 Why Did This Problem Occur?

WSL Network Isolation: WSL2 runs in a virtualized environment with its own network stack. localhost in WSL ≠ localhost in Windows.
Configuration Issue: CCR's config file (~/.claude-code-router/config.json) was either:

Not configured at all, OR
Pointing to http://localhost:8000 instead of the Windows host IP


Why It Worked Yesterday:

Network configuration might have changed (Windows IP can change on restart)
CCR config might have been accidentally reset
WSL might have been restarted, changing network routing


### Maintained by Shahzain Ali | [LinkedIn Profile](https://www.linkedin.com/in/shahzain-ali-518b862ba/)
