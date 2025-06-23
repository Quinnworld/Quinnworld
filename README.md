<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>基因外貌画像AI生成</title>
  <style>
    body {
      font-family: '微软雅黑', sans-serif;
      background: #f7f9fc;
      margin: 0;
      padding: 0;
      display: flex;
      flex-direction: column;
      align-items: center;
    }
    #quiz-container {
      width: 95vw;
      max-width: 700px;
      margin: 30px 0;
      padding: 24px;
      background: #ffffff;
      border-radius: 12px;
      box-shadow: 0 0 12px rgba(0, 0, 0, 0.08);
    }
    h1 {
      font-size: 24px;
      text-align: center;
      color: #2b2f4a;
      margin-bottom: 20px;
    }
    .question {
      margin-bottom: 20px;
    }
    .question h3 {
      font-size: 18px;
      margin-bottom: 10px;
    }
    .option {
      background: #f0f4ff;
      border-radius: 8px;
      padding: 10px;
      margin-bottom: 8px;
      cursor: pointer;
      border: 1px solid #d0d7ef;
    }
    .option.selected {
      background: #4f83ff;
      color: white;
      border-color: #4f83ff;
      font-weight: bold;
    }
    #resultBox {
      display: none;
      background: #eef3fc;
      border-radius: 12px;
      padding: 16px;
      margin-top: 30px;
    }
    #promptOutput {
      white-space: pre-wrap;
      background: #f7f9fc;
      padding: 12px;
      border: 1px solid #cfd8e9;
      border-radius: 8px;
      margin-top: 16px;
    }
    button {
      margin-top: 16px;
      padding: 10px 20px;
      background: #4f83ff;
      border: none;
      border-radius: 8px;
      color: white;
      font-weight: bold;
      cursor: pointer;
    }
    #paywall {
      display: none;
      background: #fff4f4;
      border: 1px solid #ffd6d6;
      padding: 20px;
      margin-top: 20px;
      border-radius: 8px;
      text-align: center;
    }
    #paywall img {
      width: 220px;
      margin-top: 10px;
    }
  </style>
</head>
<body>
<div id="quiz-container">
  <h1>AI虚拟形象基因问卷</h1>
  <div id="quiz"></div>
  <button id="submitBtn">提交并生成AI形象描述</button>
  <div id="resultBox">
    <h3>AI Prompt（可用于图像生成）：</h3>
    <div id="promptOutput"></div>
    <div id="paywall">
      <h4>深度人格报告（付费解锁）</h4>
      <p>扫描二维码支付 ¥9.9 元解锁完整版报告</p>
      <img src="https://api.qrserver.com/v1/create-qr-code/?size=200x200&data=YOUR_PAYMENT_LINK" alt="二维码" />
    </div>
  </div>
</div>
<script>
  const submitBtn = document.getElementById("submitBtn");
  const resultBox = document.getElementById("resultBox");
  const promptOutput = document.getElementById("promptOutput");
  const paywall = document.getElementById("paywall");

  submitBtn.onclick = function () {
    promptOutput.textContent = "Anime-style portrait of a confident young woman with fair skin, 3D facial structure, glossy dark hair, and radiant presence.";
    resultBox.style.display = "block";
    paywall.style.display = "block";
  };
</script>
</body>
</html>