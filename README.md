# 🌉 Tele-Sync — WhatsApp ↔ Telegram Bridge

A self-hosted bridge that **forwards your WhatsApp messages to Telegram** and lets you reply back — all from Telegram. Built on top of [watgbridge](https://github.com/akshettrj/watgbridge) with extra patches for flat-chat routing and group-to-group forwarding.

> ⚠️ **DISCLAIMER:** This project is not affiliated with WhatsApp or Telegram. Using unofficial WhatsApp clients may result in your account being banned. **Use at your own risk.**

---

## ✨ What it does

- 📨 Forwards incoming WhatsApp messages (DMs + groups) → Telegram
- 💬 Lets you reply from Telegram → sends message back to WhatsApp
- 🗂️ **Flat routing mode** — each WhatsApp group maps to its own Telegram chat (no Forum Topics needed)
- 📲 Also syncs messages you send from your other WhatsApp devices
- 🔁 React to messages, tag @everyone, send stickers and files — all supported

---

## 🖥️ Requirements

| Tool | Version | Notes |
|------|---------|-------|
| Go | ≥ 1.21 | [install guide](https://go.dev/dl/) |
| git | any | standard |
| gcc | any | needed by cgo |
| ffmpeg | any | for video/audio conversion |
| imagemagick | optional | for sticker conversion |

---

## 🚀 Step-by-Step Setup

### 1. Create a Telegram Bot

1. Open Telegram, find **@BotFather**
2. Send /newbot, give it a name and username
3. Copy the **bot token** — you will need it in config.yaml

### 2. Get your Telegram user/group ID

- Forward any message to **@userinfobot** to get your personal Telegram ID
- To get a group/channel ID: add **@userinfobot** to the group, or use [@getidsbot](https://t.me/getidsbot)

### 3. Clone and build

`ash
git clone https://github.com/opiumniy/Tele-Sync.git
cd Tele-Sync
go build
`

If the build succeeds, you will have a ./watgbridge binary in the folder.

### 4. Configure

`ash
cp sample_config.yaml config.yaml
`

Open config.yaml in any text editor and fill in the required fields:

`yaml
telegram:
  bot_token: 123456:ABC-your-bot-token-here
  owner_id: 123456789          # your personal Telegram user ID
  target_chat_id: -1001234567  # Telegram group where WA messages land

whatsapp:
  # Optional: map a specific WhatsApp group to a specific Telegram chat
  # group_jid: 120363XXXXX@g.us
  # group_telegram_id: -1009876543

send_my_messages_from_other_devices: true   # sync your own WA messages
`

> 💡 All config options are documented with comments inside sample_config.yaml — read it!

### 5. First run (WhatsApp login)

`ash
./watgbridge
`

On first launch the bot will display a **QR code** in the terminal AND save it as qrcode.png in the project folder.

Scan the QR code with your WhatsApp app:
> **WhatsApp → Settings → Linked Devices → Link a Device**

Once scanned, the bot logs in and starts forwarding messages.

### 6. Run in background (optional but recommended)

A sample systemd service file is included: watgbridge.service.sample

`ash
sudo cp watgbridge.service.sample /etc/systemd/system/watgbridge.service
# Edit the file: set User= and ExecStart= to match your paths
sudo systemctl daemon-reload
sudo systemctl enable --now watgbridge
`

Or just use 
ohup for a quick start:

`ash
nohup ./watgbridge > watgbridge.log 2>&1 &
tail -f watgbridge.log   # watch logs
`

---

## 🗂️ Flat Routing Mode (Group → Chat)

This fork adds **flat routing**: instead of using Telegram Forum Topics, each WhatsApp group is forwarded to a **dedicated Telegram chat**.

To enable group routing, set in config.yaml:

`yaml
whatsapp:
  group_jid: 120363419133044124@g.us   # WhatsApp group JID
  group_telegram_id: -1003979059641       # Telegram chat to forward to
`

**How to find a WhatsApp group JID:**
- Check the bot logs on first run — it prints JIDs for all groups you are in
- Or run: grep @g.us watgbridge.log | head -20

---

## 🔍 Finding WhatsApp Group JIDs

Run the bot once and send a message in the group. The JID will appear in the logs:

`
INFO  received message from 120363419133044124@g.us
`

---

## 🐛 Troubleshooting

| Problem | Fix |
|---------|-----|
| dial tcp: lookup api.telegram.org | Your server DNS is broken. Add 149.154.167.220 api.telegram.org to /etc/hosts |
| Bad Request: the chat is not a forum | Flat routing is active — make sure 	arget_chat_id is a plain group, not a Forum |
| Bot does not respond | Check watgbridge.log for errors; restart the bot |
| QR code expired | Restart the bot — a new QR will be generated |
| WhatsApp disconnects frequently | Normal behavior — the systemd service auto-restarts every 24 h |

---

## 📁 Project Structure

`
.
├── main.go                  # entry point
├── config.yaml              # your config (not committed)
├── sample_config.yaml       # config template with comments
├── whatsapp/
│   ├── client.go            # WA login, QR generation
│   └── handlers.go          # WA → Telegram forwarding logic
├── telegram/
│   └── handlers.go          # Telegram → WA forwarding logic
├── utils/
│   └── telegram.go          # helper: flat routing (no forum topics)
└── watgbridge.service.sample # systemd service template
`

---

## 🤝 Contributing

PRs are welcome! This fork focuses on flat-routing and group-to-chat mapping. If you have improvements, open a PR against this repo or the [upstream project](https://github.com/akshettrj/watgbridge).

---

## 📜 License

MIT — see [LICENSE](./LICENSE)
