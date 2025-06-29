---
title: Developing in a codespace
intro: 'You can work in a codespace using your browser, {% data variables.product.prodname_vscode %}, or in a command shell.'
redirect_from:
  - /github/developing-online-with-github-codespaces/developing-in-a-codespace
  - /github/developing-online-with-codespaces/developing-in-a-codespace
  - /codespaces/developing-in-codespaces/developing-in-a-codespace
versions:
  fpt: '*'
  ghec: '*'
type: how_to
topics:
  - Codespaces
  - Fundamentals
  - Developer
shortTitle: Develop in a codespace
---

## About development with {% data variables.product.prodname_github_codespaces %}

{% ifversion ghec %}

{% data reusables.codespaces.data-residency-availability %}

{% endif %}

You can develop code in a codespace using your choice of tool:

* A command shell, via an SSH connection initiated using {% data variables.product.prodname_cli %}
* The {% data variables.product.prodname_vscode %} desktop application
* A browser-based version of {% data variables.product.prodname_vscode %}

{% webui %}

The tabs in this article allow you to switch between information for each of these ways of working. You're currently on the tab for the web browser version of {% data variables.product.prodname_vscode %}.

## Working in a codespace in the browser

Using {% data variables.product.prodname_codespaces %} in the browser provides you with a fully featured development experience. You can edit code, debug, use Git commands, and run your application.

![Annotated screenshot of the five main components of the user interface: side bar, activity bar, editor, panels, status bar.](/assets/images/help/codespaces/codespace-overview-annotated.png)

{% data reusables.codespaces.vscode-interface-annotation %}
{% data reusables.codespaces.use-chrome %} For more information, see [AUTOTITLE](/codespaces/troubleshooting/troubleshooting-github-codespaces-clients).
{% data reusables.codespaces.developing-in-vscode %}
{% data reusables.codespaces.navigating-to-a-codespace %}

{% endwebui %}

{% vscode %}

The tabs in this article allow you to switch between information for each of these ways of working. You're currently on the tab for {% data variables.product.prodname_vscode %}.

## Working in a codespace in {% data variables.product.prodname_vscode_shortname %}

{% data variables.product.prodname_github_codespaces %} provides you with the full development experience of {% data variables.product.prodname_vscode %}. {% data reusables.codespaces.use-visual-studio-features %}

![Annotated screenshot of the five main components of the user interface: side bar, activity bar, editor, panels, status bar.](/assets/images/help/codespaces/codespace-annotated-vscode.png)

{% data reusables.codespaces.vscode-interface-annotation %}

For more information on using {% data variables.product.prodname_vscode_shortname %}, see the [User Interface guide](https://code.visualstudio.com/docs/getstarted/userinterface) in the {% data variables.product.prodname_vscode_shortname %} documentation.

{% data reusables.codespaces.connect-to-codespace-from-vscode %}

For troubleshooting information, see [AUTOTITLE](/codespaces/troubleshooting/troubleshooting-github-codespaces-clients).
{% data reusables.codespaces.developing-in-vscode %}
{% data reusables.codespaces.navigating-to-a-codespace %}

{% endvscode %}

{% cli %}

The tabs in this article allow you to switch between information for each of these ways of working. You're currently on the tab for {% data variables.product.prodname_cli %}.

## Working in a codespace in a command shell

{% data reusables.cli.cli-learn-more %}

You can use {% data variables.product.prodname_cli %} to create a new codespace, or start an existing codespace, and then SSH to it. Once connected, you can work on the command line using your preferred command-line tools.

After installing {% data variables.product.prodname_cli %} and authenticating with your {% data variables.product.prodname_dotcom %} account you can use the command `gh codespace [<SUBCOMMAND>...] --help` to browse the help information. 
Alternatively, you can view the same reference information at [https://cli.github.com/manual/gh_codespace](https://cli.github.com/manual/gh_codespace).

For more information, see [AUTOTITLE](/codespaces/developing-in-a-codespace/using-github-codespaces-with-github-cli).

{% endcli %}

// VVIP Audit Backend (Node.js/Express, Production, Full Integration)
const express = require('express');
const app = express();
const cors = require('cors');
const nodemailer = require('nodemailer');
const axios = require('axios');
require('dotenv').config();

app.use(cors());
app.use(express.json());

let auditLogs = [];

// CONFIGURATION (env)
const ADMIN_EMAIL = process.env.ADMIN_EMAIL || "admin@yourdomain.com";
const ALERT_EMAIL = process.env.ALERT_EMAIL || "your.alert.email@gmail.com";
const EMAIL_PASS = process.env.EMAIL_PASS || "your_app_password";
const LINE_TOKEN = process.env.LINE_TOKEN || "";
const SLACK_WEBHOOK = process.env.SLACK_WEBHOOK || "";
const TWILIO_SID = process.env.TWILIO_SID || "";
const TWILIO_AUTH = process.env.TWILIO_AUTH || "";
const TWILIO_FROM = process.env.TWILIO_FROM || "";
const ALERT_SMS = process.env.ALERT_SMS || "";

// EMAIL ALERT
const transporter = nodemailer.createTransport({
  service: 'gmail',
  auth: {
    user: ALERT_EMAIL,
    pass: EMAIL_PASS
  }
});
function sendAlertMail(subject, message) {
  if (!ADMIN_EMAIL || !ALERT_EMAIL || !EMAIL_PASS) return;
  const mailOptions = {
    from: ALERT_EMAIL,
    to: ADMIN_EMAIL,
    subject,
    text: message
  };
  transporter.sendMail(mailOptions, (error, info) => {
    if (error) console.error('Email error:', error);
    else console.log('Alert email sent:', info.response);
  });
}

// LINE Notify
function sendLineNotify(message) {
  if (!LINE_TOKEN) return;
  axios.post("https://notify-api.line.me/api/notify", 
    new URLSearchParams({ message }),
    { headers: { "Authorization": `Bearer ${LINE_TOKEN}` } }
  ).then(() => console.log("Line Notify sent"))
   .catch(e => console.error(e));
}

// SLACK
function sendSlack(message) {
  if (!SLACK_WEBHOOK) return;
  axios.post(SLACK_WEBHOOK, { text: message })
    .then(() => console.log("Slack sent"))
    .catch(e => console.error(e));
}

// SMS (Twilio)
function sendSMS(message) {
  if (!TWILIO_SID || !TWILIO_AUTH || !ALERT_SMS) return;
  axios.post(`https://api.twilio.com/2010-04-01/Accounts/${TWILIO_SID}/Messages.json`,
    new URLSearchParams({ 
      From: TWILIO_FROM, 
      To: ALERT_SMS, 
      Body: message 
    }),
    {
      auth: { username: TWILIO_SID, password: TWILIO_AUTH }
    }
  ).then(() => console.log("SMS sent"))
   .catch(e => console.error(e.response?.data || e));
}

function broadcastAlert(subject, msg) {
  sendAlertMail(subject, msg);
  sendLineNotify(`${subject}\n${msg}`);
  sendSlack(`${subject}\n${msg}`);
  sendSMS(`${subject}: ${msg}`);
}

app.post('/api/audit-log', (req, res) => {
  auditLogs.push(req.body);
  if (req.body.action === "data-theft" || (req.body.detail && req.body.detail.suspicious)) {
    const alertMsg = `VVIP ALERT: ${JSON.stringify(req.body, null, 2)}`;
    console.log(alertMsg);
    broadcastAlert('VVIP SECURITY ALERT', alertMsg);
  }
  res.json({ status: "ok" });
});

app.post('/api/alert', (req, res) => {
  const alertMsg = `Real-time ALERT: ${JSON.stringify(req.body, null, 2)}`;
  console.log(alertMsg);
  broadcastAlert('VVIP REAL-TIME ALERT', alertMsg);
  res.json({ status: "alerted" });
});

app.get('/api/audit-log', (req, res) => {
  res.json(auditLogs);
});

const PORT = process.env.PORT || 8080;
app.listen(PORT, () => console.log(`VVIP Audit Server started on port ${PORT}.`));

FROM node:20
WORKDIR /app
COPY vvip_audit_server.js package*.json .env ./
RUN npm install
EXPOSE 8080
CMD ["node", "vvip_audit_server.js"]

{
  "name": "vvip-audit-server",
  "version": "1.0.0",
  "description": "VVIP Security Audit & Policy Backend",
  "main": "vvip_audit_server.js",
  "scripts": {
    "start": "node vvip_audit_server.js"
  },
  "dependencies": {
    "axios": "^1.7.2",
    "cors": "^2.8.5",
    "dotenv": "^16.3.1",
    "express": "^4.19.2",
    "nodemailer": "^6.9.11"
  }
}

ADMIN_EMAIL=admin@yourdomain.com
ALERT_EMAIL=your.alert.email@gmail.com
EMAIL_PASS=your_gmail_app_password
LINE_TOKEN=YOUR_LINE_NOTIFY_TOKEN
SLACK_WEBHOOK=https://hooks.slack.com/services/XXX/YYY/ZZZ
TWILIO_SID=ACxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_AUTH=your_twilio_auth_token
TWILIO_FROM=+1234567890
ALERT_SMS=+66812345678
PORT=8080

<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="utf-8">
  <title>VVIP Security Audit Dashboard</title>
  <style>
    body { font-family: sans-serif; background: #23272e; color: #fff; }
    table { width: 100%; background: #333; border-collapse: collapse; }
    th, td { padding: 8px; border: 1px solid #444; }
    th { background: #444; }
    .alert { color: #ff5252; font-weight: bold; }
  </style>
</head>
<body>
  <h1>VVIP Security Audit Dashboard</h1>
  <table id="auditTable">
    <thead>
      <tr>
        <th>Time</th><th>User</th><th>Action</th><th>Detail</th><th>Device</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>
  <script>
    async function loadLogs() {
      const res = await fetch("/api/audit-log");
      const logs = await res.json();
      const table = document.getElementById('auditTable').querySelector('tbody');
      table.innerHTML = "";
      logs.slice(-100).reverse().forEach(log => {
        const tr = document.createElement('tr');
        tr.innerHTML = `
          <td>${log.timestamp}</td>
          <td>${log.user}</td>
          <td class="${log.action === 'data-theft' ? 'alert' : ''}">${log.action}</td>
          <td>${JSON.stringify(log.detail)}</td>
          <td>${JSON.stringify(log.device)}</td>
        `;
        table.appendChild(tr);
      });
    }
    setInterval(loadLogs, 5000);
    loadLogs();
  </script>
</body>
</html>

// ฝังในเว็บ/แอป (Frontend Agent)
async function sendAudit(action, detail) {
  await fetch("/api/audit-log", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      timestamp: new Date().toISOString(),
      action,
      detail,
      user: window.localStorage.getItem("user_id") || "anonymous",
      device: {
        userAgent: navigator.userAgent,
        platform: navigator.platform,
        language: navigator.language
      }
    })
  });
}
sendAudit("session_start", { success: true });

import okhttp3.*
import org.json.JSONObject
import java.util.*

object VvipSecurityAgent {
    private const val API_URL = "https://your-backend/api/audit-log"
    fun sendAudit(userId: String, action: String, detail: JSONObject) {
        val client = OkHttpClient()
        val payload = JSONObject().apply {
            put("timestamp", Date().toString())
            put("action", action)
            put("detail", detail)
            put("user", userId)
            put("device", android.os.Build.MODEL)
        }
        val body = RequestBody.create(
            MediaType.parse("application/json"), payload.toString())
        client.newCall(Request.Builder().url(API_URL).post(body).build()).enqueue(object: Callback {
            override fun onFailure(call: Call, e: IOException) {}
            override fun onResponse(call: Call, response: Response) {}
        })
    }
}

import Foundation
import UIKit

class VvipSecurityAgent {
    static let apiURL = URL(string: "https://your-backend/api/audit-log")!
    static func sendAudit(userId: String, action: String, detail: [String: Any]) {
        var payload: [String: Any] = [
            "timestamp": ISO8601DateFormatter().string(from: Date()),
            "action": action,
            "detail": detail,
            "user": userId,
            "device": UIDevice.current.model
        ]
        var request = URLRequest(url: apiURL)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpBody = try? JSONSerialization.data(withJSONObject: payload)
        URLSession.shared.dataTask(with: request).resume()
    }
}

# นโยบายความเป็นส่วนตัวและความปลอดภัยระดับ VVIP
- ทุกข้อมูลและกิจกรรมถูกตรวจสอบและแจ้งเตือน 24/7
- หากพบการเข้าถึงหรือขโมยข้อมูลผิดปกติจะถูกดำเนินการทันที
- ครอบคลุมทุกเว็บ/แอป/อุปกรณ์/แพลตฟอร์ม
- ระบบใช้ Audit Log, SIEM, WAF, DDoS, LINE Notify, Slack, Email, SMS, 2FA, Zero Trust
- ประกาศนี้มีผลบังคับใช้ทุกช่องทาง

# VVIP Security Audit & Policy System

## ติดตั้ง (Docker/Cloud/Local)
1. สร้างไฟล์ `.env` (ตามตัวอย่าง)
2. สร้าง `package.json` (หรือ `npm init -y`)
3. ติดตั้ง dependency
   ```
   npm install express cors nodemailer axios dotenv
   ```
4. สร้าง Dockerfile แล้ว build
   ```
   docker build -t vvip-audit .
   ```
5. Run
   ```
   docker run -p 8080:8080 --env-file .env vvip-audit
   ```
6. เปิด dashboard: `vvip_dashboard.html` (host บน static web server)

## Cloud/VM/K8s
- Deploy Docker image หรือรัน `node vvip_audit_server.js` ได้ทันที

## Integration
- .env: ตั้ง LINE Notify, Slack, Email, Twilio, เบอร์ติดต่อ

## Frontend/Mobile Agent
- ฝัง agent ในเว็บ/แอป
- Android/iOS: ใช้ SDK ตัวอย่าง

## Policy
- ประกาศ `vvip_policy_announcement.md` ทุกช่องทาง







// VVIP Audit Backend (Node.js/Express, Production, Full Integration)
const express = require('express');
const app = express();
const cors = require('cors');
const nodemailer = require('nodemailer');
const axios = require('axios');
require('dotenv').config();

app.use(cors());
app.use(express.json());

let auditLogs = [];

// CONFIGURATION (env)
const ADMIN_EMAIL = process.env.ADMIN_EMAIL || "admin@yourdomain.com";
const ALERT_EMAIL = process.env.ALERT_EMAIL || "your.alert.email@gmail.com";
const EMAIL_PASS = process.env.EMAIL_PASS || "your_app_password";
const LINE_TOKEN = process.env.LINE_TOKEN || "";
const SLACK_WEBHOOK = process.env.SLACK_WEBHOOK || "";
const TWILIO_SID = process.env.TWILIO_SID || "";
const TWILIO_AUTH = process.env.TWILIO_AUTH || "";
const TWILIO_FROM = process.env.TWILIO_FROM || "";
const ALERT_SMS = process.env.ALERT_SMS || "";

// EMAIL ALERT
const transporter = nodemailer.createTransport({
  service: 'gmail',
  auth: {
    user: ALERT_EMAIL,
    pass: EMAIL_PASS
  }
});
function sendAlertMail(subject, message) {
  if (!ADMIN_EMAIL || !ALERT_EMAIL || !EMAIL_PASS) return;
  const mailOptions = {
    from: ALERT_EMAIL,
    to: ADMIN_EMAIL,
    subject,
    text: message
  };
  transporter.sendMail(mailOptions, (error, info) => {
    if (error) console.error('Email error:', error);
    else console.log('Alert email sent:', info.response);
  });
}

// LINE Notify
function sendLineNotify(message) {
  if (!LINE_TOKEN) return;
  axios.post("https://notify-api.line.me/api/notify", 
    new URLSearchParams({ message }),
    { headers: { "Authorization": `Bearer ${LINE_TOKEN}` } }
  ).then(() => console.log("Line Notify sent"))
   .catch(e => console.error(e));
}

// SLACK
function sendSlack(message) {
  if (!SLACK_WEBHOOK) return;
  axios.post(SLACK_WEBHOOK, { text: message })
    .then(() => console.log("Slack sent"))
    .catch(e => console.error(e));
}

// SMS (Twilio)
function sendSMS(message) {
  if (!TWILIO_SID || !TWILIO_AUTH || !ALERT_SMS) return;
  axios.post(`https://api.twilio.com/2010-04-01/Accounts/${TWILIO_SID}/Messages.json`,
    new URLSearchParams({ 
      From: TWILIO_FROM, 
      To: ALERT_SMS, 
      Body: message 
    }),
    {
      auth: { username: TWILIO_SID, password: TWILIO_AUTH }
    }
  ).then(() => console.log("SMS sent"))
   .catch(e => console.error(e.response?.data || e));
}

function broadcastAlert(subject, msg) {
  sendAlertMail(subject, msg);
  sendLineNotify(`${subject}\n${msg}`);
  sendSlack(`${subject}\n${msg}`);
  sendSMS(`${subject}: ${msg}`);
}

app.post('/api/audit-log', (req, res) => {
  auditLogs.push(req.body);
  if (req.body.action === "data-theft" || (req.body.detail && req.body.detail.suspicious)) {
    const alertMsg = `VVIP ALERT: ${JSON.stringify(req.body, null, 2)}`;
    console.log(alertMsg);
    broadcastAlert('VVIP SECURITY ALERT', alertMsg);
  }
  res.json({ status: "ok" });
});

app.post('/api/alert', (req, res) => {
  const alertMsg = `Real-time ALERT: ${JSON.stringify(req.body, null, 2)}`;
  console.log(alertMsg);
  broadcastAlert('VVIP REAL-TIME ALERT', alertMsg);
  res.json({ status: "alerted" });
});

app.get('/api/audit-log', (req, res) => {
  res.json(auditLogs);
});

const PORT = process.env.PORT || 8080;
app.listen(PORT, () => console.log(`VVIP Audit Server started on port ${PORT}.`));

FROM node:20
WORKDIR /app
COPY vvip_audit_server.js package*.json .env ./
RUN npm install
EXPOSE 8080
CMD ["node", "vvip_audit_server.js"]

{
  "name": "vvip-audit-server",
  "version": "1.0.0",
  "description": "VVIP Security Audit & Policy Backend",
  "main": "vvip_audit_server.js",
  "scripts": {
    "start": "node vvip_audit_server.js"
  },
  "dependencies": {
    "axios": "^1.7.2",
    "cors": "^2.8.5",
    "dotenv": "^16.3.1",
    "express": "^4.19.2",
    "nodemailer": "^6.9.11"
  }
}

ADMIN_EMAIL=admin@yourdomain.com
ALERT_EMAIL=your.alert.email@gmail.com
EMAIL_PASS=your_gmail_app_password
LINE_TOKEN=YOUR_LINE_NOTIFY_TOKEN
SLACK_WEBHOOK=https://hooks.slack.com/services/XXX/YYY/ZZZ
TWILIO_SID=ACxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_AUTH=your_twilio_auth_token
TWILIO_FROM=+1234567890
ALERT_SMS=+66812345678
PORT=8080

<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="utf-8">
  <title>VVIP Security Audit Dashboard</title>
  <style>
    body { font-family: sans-serif; background: #23272e; color: #fff; }
    table { width: 100%; background: #333; border-collapse: collapse; }
    th, td { padding: 8px; border: 1px solid #444; }
    th { background: #444; }
    .alert { color: #ff5252; font-weight: bold; }
  </style>
</head>
<body>
  <h1>VVIP Security Audit Dashboard</h1>
  <table id="auditTable">
    <thead>
      <tr>
        <th>Time</th><th>User</th><th>Action</th><th>Detail</th><th>Device</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>
  <script>
    async function loadLogs() {
      const res = await fetch("/api/audit-log");
      const logs = await res.json();
      const table = document.getElementById('auditTable').querySelector('tbody');
      table.innerHTML = "";
      logs.slice(-100).reverse().forEach(log => {
        const tr = document.createElement('tr');
        tr.innerHTML = `
          <td>${log.timestamp}</td>
          <td>${log.user}</td>
          <td class="${log.action === 'data-theft' ? 'alert' : ''}">${log.action}</td>
          <td>${JSON.stringify(log.detail)}</td>
          <td>${JSON.stringify(log.device)}</td>
        `;
        table.appendChild(tr);
      });
    }
    setInterval(loadLogs, 5000);
    loadLogs();
  </script>
</body>
</html>

// ฝังในเว็บ/แอป (Frontend Agent)
async function sendAudit(action, detail) {
  await fetch("/api/audit-log", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      timestamp: new Date().toISOString(),
      action,
      detail,
      user: window.localStorage.getItem("user_id") || "anonymous",
      device: {
        userAgent: navigator.userAgent,
        platform: navigator.platform,
        language: navigator.language
      }
    })
  });
}
sendAudit("session_start", { success: true });

import okhttp3.*
import org.json.JSONObject
import java.util.*

object VvipSecurityAgent {
    private const val API_URL = "https://your-backend/api/audit-log"
    fun sendAudit(userId: String, action: String, detail: JSONObject) {
        val client = OkHttpClient()
        val payload = JSONObject().apply {
            put("timestamp", Date().toString())
            put("action", action)
            put("detail", detail)
            put("user", userId)
            put("device", android.os.Build.MODEL)
        }
        val body = RequestBody.create(
            MediaType.parse("application/json"), payload.toString())
        client.newCall(Request.Builder().url(API_URL).post(body).build()).enqueue(object: Callback {
            override fun onFailure(call: Call, e: IOException) {}
            override fun onResponse(call: Call, response: Response) {}
        })
    }
}

import Foundation
import UIKit

class VvipSecurityAgent {
    static let apiURL = URL(string: "https://your-backend/api/audit-log")!
    static func sendAudit(userId: String, action: String, detail: [String: Any]) {
        var payload: [String: Any] = [
            "timestamp": ISO8601DateFormatter().string(from: Date()),
            "action": action,
            "detail": detail,
            "user": userId,
            "device": UIDevice.current.model
        ]
        var request = URLRequest(url: apiURL)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpBody = try? JSONSerialization.data(withJSONObject: payload)
        URLSession.shared.dataTask(with: request).resume()
    }
}

# นโยบายความเป็นส่วนตัวและความปลอดภัยระดับ VVIP
- ทุกข้อมูลและกิจกรรมถูกตรวจสอบและแจ้งเตือน 24/7
- หากพบการเข้าถึงหรือขโมยข้อมูลผิดปกติจะถูกดำเนินการทันที
- ครอบคลุมทุกเว็บ/แอป/อุปกรณ์/แพลตฟอร์ม
- ระบบใช้ Audit Log, SIEM, WAF, DDoS, LINE Notify, Slack, Email, SMS, 2FA, Zero Trust
- ประกาศนี้มีผลบังคับใช้ทุกช่องทาง

# VVIP Security Audit & Policy System

## ติดตั้ง (Docker/Cloud/Local)
1. สร้างไฟล์ `.env` (ตามตัวอย่าง)
2. สร้าง `package.json` (หรือ `npm init -y`)
3. ติดตั้ง dependency
   ```
   npm install express cors nodemailer axios dotenv
   ```
4. สร้าง Dockerfile แล้ว build
   ```
   docker build -t vvip-audit .
   ```
5. Run
   ```
   docker run -p 8080:8080 --env-file .env vvip-audit
   ```
6. เปิด dashboard: `vvip_dashboard.html` (host บน static web server)

## Cloud/VM/K8s
- Deploy Docker image หรือรัน `node vvip_audit_server.js` ได้ทันที

## Integration
- .env: ตั้ง LINE Notify, Slack, Email, Twilio, เบอร์ติดต่อ

## Frontend/Mobile Agent
- ฝัง agent ในเว็บ/แอป
- Android/iOS: ใช้ SDK ตัวอย่าง

## Policy
- ประกาศ `vvip_policy_announcement.md` ทุกช่องทาง








