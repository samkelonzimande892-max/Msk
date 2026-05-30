# 🦁 MRDIEHARD MD

A lightweight, high-performance WhatsApp bot built using Node.js and the **Baileys** socket library. This bot utilizes a convenient terminal pairing code authentication system, eliminating the need for a headless browser or QR scanning.

---

## 🚀 Features

* **Pairing Code Authentication:** Connect using your phone number directly from the terminal.
* **Lightweight Engine:** Built on `@whiskeysockets/baileys` for fast socket communication and minimal server resource usage.
* **Automatic Session Saving:** Saves your session locally so you only need to link your account once.
* **Built-in Commands:**
    * `.menu` / `.help` - Displays the command panel.
    * `.ping` - Tests socket connection response speed.
    * `.server` / `.runtime` - Displays system runtime and resource allocation.
    * `.echo [text]` - Mirrors text input.

---

## 🛠️ Prerequisites

Before setting up, make sure you have installed:
* [Node.js](https://nodejs.org/) (v16.x or higher recommended)
* An active WhatsApp account on your mobile device

---

## 📦 Setup & Installation

Follow these steps to deploy the bot on your local computer or VPS:

### 1. Clone or Create the Project Folder
Ensure your directory layout looks like this:
```text
mrdiehard-bot/
├── package.json
└── server.js
