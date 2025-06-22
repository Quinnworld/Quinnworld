<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>因•趣问答</title>
  <style>
    body {
      font-family: "微软雅黑", sans-serif;
      background: white;
      color: black;
      margin: 0;
      padding: 20px;
      max-width: 600px;
      margin: auto;
      transition: background 0.5s;
    }
    h2, h3 { text-align: center; }
    label, select, input[type=number] {
      width: 100%; margin: 10px 0; font-size: 16px;
    }
    select, input[type=number] {
      padding: 8px; border-radius: 6px; border: 1px solid #ccc;
    }
    button {
      width: 100%; padding: 12px; font-size: 16px; border: none;
      border-radius: 6px; cursor: pointer; color: white;
      background: #4338ca; margin-top: 10px;
    }
    .option {
      background: white; border: 1px solid #bbb; border-radius: 6px;
      padding: 10px; margin: 6px 0; cursor: pointer; user-select:none;
    }
    .option.selected {
      background: #4f46e5; color: white; border-color: #4338ca;
    }
    #promptOutput {
      white-space: pre-wrap; background: #eee;
      padding: 10px; border-radius: 6px; font-family: monospace;
      user-select: all;
    }
    #totalScoreText {
      text-align: center; font-weight: bold; font-size: 18px;
    }
    canvas {
      display: block; margin: 20px auto; max-width: 100%;
    }
    .nav-buttons {
      display: flex; justify-content: space-between;
    }
  </style>
</head>
<body>
  <h2>因•趣问答</h2>

  <div id="sectionUserInfo">
    <label for="inputAge">年龄（不限）</label>
    <input type="number" id="inputAge" min="0" value="25" />
    <label for="selectGender">性别</label>
    <select id="selectGender">
      <option value="male">男性</option>
      <option value="female">女性</option>
      <option value="other">其他</option>
    </select>
    <button id="btnStart">开始答题</button>
  </div>

  <div id="sectionQuiz" style="display:none;">
    <div id="questionText" style="font-weight:700; margin-bottom:10px;"></div>
    <div id="optionsContainer"></div>
    <div class="nav-buttons">
      <button id="btnPrev">上一题</button>
      <button id="btnNext" disabled>下一题</button>
    </div>
    <div id="progressText"></div>
  </div>

  <div id="sectionResult" style="display:none;">
    <h3>雷达图基因维度评分</h3>
    <canvas id="radarChart" width="400" height="400"></canvas>
    <div id="totalScoreText"></div>
    <h3>因风格 Prompt（点击可复制）</h3>
    <pre id="promptOutput" title="点击复制"></pre>
    <button id="btnRestart">重新开始</button>
  </div>

  <script>
    // JavaScript 逻辑将继续补充完整...
  </script>
</body>
</html>
