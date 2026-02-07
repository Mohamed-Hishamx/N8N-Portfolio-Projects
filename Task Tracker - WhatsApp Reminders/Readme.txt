WhatsApp Task Reminders (n8n + Twilio)

Automated WhatsApp task reminders built with n8n, Twilio WhatsApp API (Sandbox / Trial), and Google Sheets.

The workflow reads pending tasks from Google Sheets, groups them by staff phone number, and sends one consolidated WhatsApp message per user.

✨ Features

📄 Read tasks from Google Sheets

⏳ Filter only Pending tasks

👥 Group tasks by WhatsApp number

📲 Send WhatsApp messages via Twilio

🔁 Supports multiple recipients

🧪 Works with Twilio Trial (Sandbox mode)

🧰 Tech Stack

n8n – Workflow automation

Twilio WhatsApp API

Google Sheets

JavaScript (n8n Code Nodes)

📁 Project Files
.
├── Task Tracker - WhatsApp Reminders.json   # n8n workflow export
└── README.md                               # Project documentation

📊 Google Sheet Structure

Your Google Sheet must include the following columns:

Column	Description
task	Task description
staff_name	Staff member name
phone	WhatsApp number (E.164 format)
status	Task status (Pending, Done)
📞 Phone Number Format (Required)

All phone numbers must be in E.164 format:

+201XXXXXXXXX
+966XXXXXXXXX


❌ No spaces
❌ No 00 prefix
❌ No leading zeros

🧪 Twilio WhatsApp Sandbox Setup (Trial)
1️⃣ Activate Sandbox

Log in to Twilio Console

Navigate to:
Messaging → Try it out → WhatsApp Sandbox

Activate the sandbox

You’ll receive:

A sandbox WhatsApp number (usually +14155238886)

A join code (example: join alpha-bravo)

2️⃣ Join Sandbox (MANDATORY)

Every WhatsApp number that should receive messages must join the sandbox:

Open WhatsApp on the phone

Send the join message to the sandbox number

Wait for Twilio’s confirmation reply

⚠️ If a number does not join → messages will NOT be delivered.

🔐 Twilio Credentials (n8n)

In Twilio Console, copy:

Account SID

Auth Token

In n8n:

Go to Credentials

Create a Twilio credential

Paste SID & Auth Token

Save

⚙️ n8n Workflow Overview
Main Nodes

Trigger (Manual or Cron)

Google Sheets – Read task rows

Code Node

Filter Pending tasks

Group tasks by phone number

Build WhatsApp message text

Twilio – Send WhatsApp message

📲 Twilio Node Configuration
Setting	Value
Resource	Message
Operation	Send
To	={{ $json.phoneNumber }}
From	Twilio Sandbox Number (e.g. +14155238886)
To WhatsApp	✅ Enabled
Message	={{ $json.message }}

🔁 n8n sends one message per item, enabling multiple recipients automatically.

📤 How Multiple Messages Work

The workflow outputs data like:

[
  {
    "phoneNumber": "+201234567890",
    "message": "Hello Ahmed, you have 2 pending tasks..."
  },
  {
    "phoneNumber": "+966512345678",
    "message": "Hello Sara, you have 1 pending task..."
  }
]


Each item = one WhatsApp message sent by Twilio.

✅ Testing Checklist

Before running the workflow:

✅ Twilio Sandbox is active

✅ All phone numbers joined the sandbox

✅ Phone numbers are in E.164 format

✅ Google Sheet contains Pending tasks

✅ Twilio credentials are configured in n8n

Run the workflow manually and inspect the Twilio node output for delivery status.

⚠️ Trial Limitations

Limited number of messages per day

Sandbox sender only

WhatsApp 24-hour session rules apply

🚀 Production Notes

To move to production:

Upgrade Twilio account

Enable a real WhatsApp sender

Use approved WhatsApp templates when required

🛠️ Possible Enhancements

⏰ Scheduled daily reminders (Cron)

📎 Media attachments (PDF / Images)

🟢 Update task status after delivery

📊 Message delivery logs

🧠 Smarter message personalization

👤 Author

Built with ❤️ using n8n and Twilio WhatsApp