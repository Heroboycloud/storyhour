# 📝 Memos: The Ultimate Self-Hosted Guide
> A complete manual covering installation, tips, tricks, cheatsheets, tokens, and webhooks.

---

## 🚀 1. Installation & Persistent Deployments

### Binary / Termux (Android)
To run the native compiled binary manually in a terminal or Termux environment:
```bash
# Start Memos on port 8080 with a custom data directory
./memos --mode prod --port 8080 --data ./memos_data
```

### Docker Deployment
The fastest way to spin up Memos securely with persistent storage:
```bash
docker run -d \
  --name memos \
  --publish 5230:5230 \
  --volume ~/.memos/:/var/opt/memos \
  --restart always \
  neoshowcase/memos:latest
```

### 🏃‍♂️ Running 24/7 in Termux (Background Task)
Termux will kill processes if Android puts the app to sleep. Keep Memos running forever using `tmux`:
1. Install tmux: `pkg install tmux`
2. Start a new session: `tmux new -s memos-session`
3. Launch your binary inside tmux: `./memos --port 8080`
4. Detach from the session safely: Press `Ctrl + B`, then let go and press `D`.
5. *Tip:* Run `termux-wake-lock` in your terminal to stop Android from sleeping your CPU.

---

## 🔑 2. Token-Based Security (Bearer Tokens)

To connect third-party apps, scripts, or automation frameworks to Memos, you must generate an Access Token.

### How to Generate a Token
1. Open your Memos Web UI (`http://localhost:5230` or your custom URL).
2. Log in and click on **Settings** (Gear Icon) in the sidebar.
3. Navigate to **My Account**.
4. Scroll down to the **Access Tokens** section.
5. Click **Create Token**, give it a description (e.g., "TextBee Bridge"), and copy the generated string.

### How to Use the Token
Always pass this token in your HTTP request headers as a Bearer signature:
```http
Authorization: Bearer YOUR_GENERATED_ACCESS_TOKEN
```

---

## ⚡ 3. REST API & Webhook Cheatsheet

### 📡 Creating a Memo via REST API (cURL)
Replace `YOUR_ACCESS_TOKEN` and `http://localhost:5230` with your actual setup details:

```bash
curl -X POST http://localhost:5230/api/v1/memos \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "#Inbox From API\nThis is an automated note created via cURL request. #api",
    "visibility": "PRIVATE"
  }'
```
*Note: Valid visibilities are `PRIVATE`, `PROTECTED` (visible to logged-in users), or `PUBLIC`.*

### 🛠️ Connecting TextBee Webhook to Memos (Node.js Express)
This script sets up a webhook listener server. When TextBee sends an incoming text to this server, it reformats it and posts it straight into Memos:

```javascript
const express = require('express');
const axios = require('axios');
const app = express();

app.use(express.json());

const MEMOS_API_URL = 'http://localhost:5230/api/v1/memos';
const MEMOS_TOKEN = 'YOUR_MEMOS_ACCESS_TOKEN';

// Webhook listener endpoint
app.post('/webhooks/sms', async (req, res) => {
    try {
        const { sender, message, timestamp } = req.body;

        // 1. Format the data neatly into Markdown text
        const memoContent = `### 📱 New SMS Received\n**From:** \${sender}\n**Time:** \({new Date(timestamp).toLocaleString()}\n\n>\){message}\n\n#sms #incoming`;

        // 2. Forward the content payload straight to Memos
        const response = await axios.post(MEMOS_API_URL, {
            content: memoContent,
            visibility: 'PRIVATE'
        }, {
            headers: {
                'Authorization': `Bearer \${MEMOS_TOKEN}`,
                'Content-Type': 'application/json'
            }
        });

        console.log('Successfully saved text to Memos! ID:', response.data.id);
        res.status(200).send('Webhook Processed Successfully');
    } catch (error) {
        console.error('Error forwarding message to Memos:', error.message);
        res.status(500).send('Internal Server Error');
    }
});

app.listen(3000, () => console.log('TextBee-to-Memos bridge listening on port 3000'));
```

---

## 📝 4. Markdown & Formatting Cheatsheet

Memos utilizes a highly optimized Markdown parser. Use these layouts to maximize note readability:

| Element | Syntax Style | Visual Result |
| :--- | :--- | :--- |
| **Tags** | `#journal/daily` | Creates deep nested tag hierarchies for filtering |
| **Checklists** | `- [ ] Tasks` | Renders functional toggle boxes inside the timeline |
| **Code Block** | ` ```javascript ` | Wraps code scripts with language syntaxes |
| **Links** | `[Memos](https://usememos.com)` | Embedded inline click targets |
| **To-Do** | `TODO: buy milk` | Highlights specific internal target flags |

---

## 💡 5. Power-User Tips & Tricks

### 🎨 Custom UI Themes (CSS Injection)
You can fundamentally redesign the appearance of your instance without modifying source code. Go to **Settings > System** and paste your custom styles inside the **Custom CSS** field:

```css
/* Custom Matrix Dark Theme Element Tweaks */
body {
  background-color: #0d1117 !important;
  color: #c9d1d9 !important;
}
.memo-wrapper {
  border: 1px solid #30363d !important;
  border-radius: 8px !important;
  background-color: #161b22 !important;
}
span.tag-span {
  background-color: #238636 !important;
  color: #ffffff !important;
}
```

### 🔍 Advanced URL Query Filtering
Bookmark specific system layout links on your browser or phone homescreen to skip straight to filtered perspectives instantly:
* Filter to public micro-blogs: `http://localhost:5230/?visibility=PUBLIC`
* Filter down exclusively to tasks: `http://localhost:5230/?tag=tasks`

### ⌨️ Native Global Keyboard Shortcuts
Speed up your navigation when using the desktop layout:
* `N` — Instantly open a blank text input field composer box.
* `Ctrl + Enter` — Save and post the note you are currently drafting.
* `Esc` — Close the text layout interface box completely.
