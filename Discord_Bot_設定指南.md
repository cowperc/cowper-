# Discord Bot + OpenCode AI 設定指南

## 目標
讓 Discord 機器人用 OpenCode AI 回覆訊息

---

## 第一步：建立 Discord Bot

1. 前往 [Discord Developer Portal](https://discord.com/developers/applications)
2. 點擊 **New Application**，輸入名稱（如：未來之書 AI）
3. 左側點 **Bot**
4. 點 **Reset Token** 並複製 Token（只會顯示一次！）
5. 往下滾動，開啟 **Privileged Gateway Intents**：
   - MESSAGE CONTENT INTENT

---

## 第二步：邀請 Bot 到伺服器

1. 左側點 **OAuth2 > URL Generator**
2. Scopes 勾選 `bot`
3. Bot Permissions 勾選：
   - Read Messages
   - Send Messages
   - Read Message History
4. 複製產生的 URL，用瀏覽器開啟，選擇伺服器

---

## 第三步：安裝軟體

### Python
下載安裝：https://python.org

### 安裝套件
```bash
pip install discord.py
```

---

## 第四步：建立 Bot 程式

在資料夾建立 `bot.py`，內容如下：

```python
# -*- coding: utf-8 -*-
import discord
import asyncio

intents = discord.Intents.all()
client = discord.Client(intents=intents)

async def get_ai_response(prompt):
    try:
        proc = await asyncio.create_subprocess_exec(
            'opencode', 'run', prompt,
            stdout=asyncio.subprocess.PIPE,
            stderr=asyncio.subprocess.PIPE
        )
        stdout, _ = await asyncio.wait_for(proc.communicate(), timeout=120)
        return stdout.decode('utf-8', errors='ignore').strip()
    except asyncio.TimeoutError:
        return "Timeout error"
    except Exception as e:
        return f"Error: {str(e)}"

@client.event
async def on_ready():
    print(f'Bot is ready!')

@client.event
async def on_message(message):
    if message.author == client.user:
        return
    
    await message.channel.send("Thinking...")
    response = await get_ai_response(message.content)
    await message.channel.send(response[:2000])

client.run('你的_Bot_Token')
```

---

## 第五步：設定 OpenCode

### 安裝 OpenCode
```bash
npm install -g opencode-ai
```

### 設定 API Key
```bash
opencode providers add
# 選擇你的 AI provider 並輸入 API Key
```

---

## 第六步：啟動 Bot

```bash
python bot.py
```

---

## 第七步：測試

在 Discord 頻道發送任何訊息，Bot 就會用 AI 回覆！

---

## 常見問題

### Bot 沒有回應？
確認 Discord Developer Portal 已開啟 MESSAGE CONTENT INTENT

### OpenCode 路徑錯誤？
Windows 可能需要用完整路徑：
```python
proc = await asyncio.create_subprocess_exec(
    'cmd', '/c', 'C:\\Users\\你的名稱\\AppData\\Roaming\\npm\\opencode.cmd', 'run', prompt,
    ...
)
```

### 想加語音功能？
安裝 edge-tts：
```bash
pip install edge-tts
```

在 bot.py 加入：
```python
import subprocess
import os

def speak(text):
    output_path = "output.mp3"
    subprocess.run([
        "edge-tts",
        "--voice", "zh-TW-HsiaoYuNeural",
        "--text", text[:500],
        "--write-media", output_path
    ])
    os.startfile(output_path)
```

---

## 可用的中文語音

| 語音 | 說明 |
|------|------|
| zh-TW-HsiaoYuNeural | 台灣女生 |
| zh-TW-YunJheNeural | 台灣男生 |
| zh-CN-XiaoxiaoNeural | 中國女生 |
| zh-CN-YunxiNeural | 中國男生 |

---

## 永續執行（讓 Bot 一直運行）

### 方法 1：pm2
```bash
npm install -g pm2
pm2 start python --name "discord-bot" -- bot.py
pm2 save
pm2 startup
```

### 方法 2：建立批次檔
建立 `run_bot.bat`：
```batch
@echo off
python bot.py
pause
```
雙擊就能啟動。

---

需要幫助請聯繫！
