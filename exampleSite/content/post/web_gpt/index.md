+++
author = "coucou"
title = "网页端gpt_demo"
date = "2023-12-01"
description = "网页端gpt_demo"
categories = [
    ""
]
tags = [
    ""
]
+++

## 如何使用

> 你只需要更改 API_KEY 即可使用

## 代码如下

index.html

```c
<!DOCTYPE html>
<html>
<head>
  <title>ChatGPT 3.5-turbo</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f5f5f5;
    }

    .container {
      max-width: 1000px;
      height: 700px;
      margin: 0 auto;
      padding: 20px;
      background-color: #fff;
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
      border-radius: 5px;
    }

    h1 {
      text-align: center;
      margin-bottom: 20px;
    }

    .chat-log {
      margin-bottom: 20px;
      overflow-y: scroll;
      max-height: 300px;
      padding-right: 10px;
      border-right: 1px solid #ccc;
    }

    .user-message {
      background-color: #f5f5f5;
      color: #333;
      border-radius: 5px;
      padding: 10px;
      margin-bottom: 10px;
    }

    .ai-message {
      background-color: #007bff;
      color: #fff;
      border-radius: 5px;
      padding: 10px;
      margin-bottom: 10px;
    }

    .input-container {
      display: flex;
      align-items: center;
    }

    .input {
      flex: 1;
      padding: 10px;
      border: 1px solid #ccc;
      border-radius: 5px;
    }

    .send-button {
      margin-left: 10px;
      padding: 10px 20px;
      background-color: #007bff;
      color: #fff;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>ChatGPT 3.5-turbo</h1>
    <div class="chat-log" id="chatLog"></div>
    <div class="input-container">
      <input class="input" id="inputMessage" placeholder="请输入消息"></input>
      <button class="send-button" onclick="sendMessage()">发送</button>
    </div>
  </div>

  <script>
    const chatLog = document.getElementById('chatLog');
    const inputMessage = document.getElementById('inputMessage');

    function sendMessage() {
      const message = inputMessage.value.trim();

      if (message === '') {
        return;
      }

      addMessageToChatLog(message, 'user-message');
      inputMessage.value = '';

      // 发送消息到服务器（ChatGPT）进行处理
      sendRequestToChatGPT(message);
    }

    function sendRequestToChatGPT(message) {
      // 构建请求对象
      const request = {
        "model": "gpt-3.5-turbo",
        "messages": [{
          "role": "user",
          "content": message
        }]
      };

      // 发送请求到服务器（这里使用示例的URL，根据实际情况修改）
      fetch('https://api.chatanywhere.com.cn/v1/chat/completions', {
        method: 'POST',
        headers: {
        'Authorization': 'Bearer YOUR_API_KEY',
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(request)
      })
      .then(response => response.json())
      .then(data => {
        console.log(data)
        const reply = data.choices[0].message.content;

        // 将服务器返回的回复添加到聊天记录中
        addMessageToChatLog(reply, 'ai-message');
      })
      .catch(error => {
        console.error('请求ChatGPT时出错:', error);
      });
    }

    function addMessageToChatLog(message, className) {
      const messageElement = document.createElement('div');
      messageElement.textContent = message;
      messageElement.classList.add(className);
      chatLog.appendChild(messageElement);
      chatLog.scrollTop = chatLog.scrollHeight;
    }
  </script>
</body>
</html>

```