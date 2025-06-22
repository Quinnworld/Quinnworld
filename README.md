<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>因•趣问答完整版</title>
  <style>
    body {
      background: white;
      color: black;
      font-family: "微软雅黑", sans-serif;
      max-width: 800px;
      margin: 0 auto;
      padding: 20px;
    }
    h2, h3 {
      text-align: center;
    }
    label, select, input[type=number] {
      width: 100%;
      margin: 10px 0;
      font-size: 16px;
    }
    select, input[type=number] {
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 6px;
    }
    button {
      padding: 10px 20px;
      font-size: 16px;
      margin-top: 10px;
      border: none;
      border-radius: 6px;
      cursor: pointer;
    }
    .option {
      background: #f5f5f5;
      border: 1px solid #ccc;
      border-radius: 6px;
      padding: 10px;
      margin: 6px 0;
      cursor: pointer;
      user-select: none;
    }
    .option.selected {
      background: #4f46e5;
      color: white;
    }
    #promptOutput {
      background: #eee;
      padding: 10px;
      font-family: monospace;
      user-select: all;
    }
    #totalScoreText {
      font-weight: bold;
      text-align: center;
      margin: 10px 0;
      font-size: 18px;
    }
    canvas {
      display: block;
      margin: 20px auto;
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
    <button onclick="startQuiz()">开始答题</button>
  </div>

  <div id="sectionQuiz" style="display:none;">
    <div id="questionText"></div>
    <div id="optionsContainer"></div>
    <button onclick="prevQuestion()">上一题</button>
    <button onclick="nextQuestion()" id="btnNext" disabled>下一题</button>
    <div id="progressText"></div>
  </div>

  <div id="sectionResult" style="display:none;">
    <h3>雷达图：因维度评分</h3>
    <canvas id="radarChart" width="400" height="400"></canvas>
    <div id="totalScoreText"></div>
    <h3>视觉Prompt</h3>
    <pre id="promptOutput" onclick="copyPrompt()"></pre>
    <button onclick="restartQuiz()">重新开始</button>
  </div>

  <script>
    const geneDimensions = [
      { key: "extraversion", label: "外向性" },
      { key: "emotion_stability", label: "情绪稳定性" },
      { key: "novelty_seek", label: "新奇寻求" },
      { key: "responsibility", label: "责任感" },
      { key: "self_control", label: "自控力" },
      { key: "openness", label: "开放性" }
    ];

    const questionPool = [
      {
        text: "你喜欢参加社交活动吗？",
        options: [
          { text: "非常不喜欢", tags: { extraversion: -2 } },
          { text: "不太喜欢", tags: { extraversion: -1 } },
          { text: "比较喜欢", tags: { extraversion: 1 } },
          { text: "非常喜欢", tags: { extraversion: 2 } }
        ]
      },
      {
        text: "面对压力时你的情绪表现？",
        options: [
          { text: "非常紧张和焦虑", tags: { emotion_stability: -2 } },
          { text: "有些不安", tags: { emotion_stability: -1 } },
          { text: "情绪稳定", tags: { emotion_stability: 1 } },
          { text: "非常冷静", tags: { emotion_stability: 2 } }
        ]
      }
      // 请继续添加题目，总计12题
    ];

    let age = 0, gender = "male", currentIndex = 0, answers = [], scores = {};

    function startQuiz() {
      age = parseInt(document.getElementById("inputAge").value);
      gender = document.getElementById("selectGender").value;
      document.getElementById("sectionUserInfo").style.display = "none";
      document.getElementById("sectionQuiz").style.display = "block";
      renderQuestion();
    }

    function renderQuestion() {
      const q = questionPool[currentIndex];
      document.getElementById("questionText").textContent = q.text;
      const container = document.getElementById("optionsContainer");
      container.innerHTML = "";
      q.options.forEach((opt, i) => {
        const div = document.createElement("div");
        div.className = "option";
        div.textContent = opt.text;
        if (answers[currentIndex] === i) div.classList.add("selected");
        div.onclick = () => selectOption(i);
        container.appendChild(div);
      });
      document.getElementById("btnNext").disabled = answers[currentIndex] == null;
      document.getElementById("progressText").textContent = `第 ${currentIndex+1} / ${questionPool.length} 题`;
      document.body.style.backgroundColor = `hsl(${(currentIndex * 40)%360}, 80%, 95%)`;
    }

    function selectOption(i) {
      answers[currentIndex] = i;
      renderQuestion();
    }

    function nextQuestion() {
      if (currentIndex < questionPool.length - 1) {
        currentIndex++;
        renderQuestion();
      } else {
        calculateResults();
      }
    }

    function prevQuestion() {
      if (currentIndex > 0) {
        currentIndex--;
        renderQuestion();
      }
    }

    function calculateResults() {
      scores = {};
      questionPool.forEach((q, idx) => {
        const selected = q.options[answers[idx]];
        for (let key in selected.tags) {
          scores[key] = (scores[key] || 0) + selected.tags[key];
        }
      });
      showResults();
    }

    function showResults() {
      document.getElementById("sectionQuiz").style.display = "none";
      document.getElementById("sectionResult").style.display = "block";

      const normalized = geneDimensions.map(dim => {
        let v = (scores[dim.key] || 0 + 8) / 16 * 100;
        return +Math.min(100, Math.max(0, v)).toFixed(1);
      });
      const total = (normalized.reduce((a,b)=>a+b,0) / normalized.length).toFixed(1);
      document.getElementById("totalScoreText").textContent = `总分: ${total}`;
      let prompt = `Age: ${age}, Gender: ${gender}\n`;
      geneDimensions.forEach((d, i) => {
        prompt += `${d.label}: ${normalized[i]} / 100\n`;
      });
      document.getElementById("promptOutput").textContent = prompt;
      drawRadar(normalized);
    }

    function drawRadar(values) {
      const ctx = document.getElementById("radarChart").getContext("2d");
      ctx.clearRect(0,0,400,400);
      const cx=200, cy=200, r=100, n=values.length, step=2*Math.PI/n;
      ctx.beginPath();
      values.forEach((v, i) => {
        const angle = step*i;
        const x = cx + r * (v/100) * Math.sin(angle);
        const y = cy - r * (v/100) * Math.cos(angle);
        i===0 ? ctx.moveTo(x,y) : ctx.lineTo(x,y);
      });
      ctx.closePath();
      ctx.strokeStyle = "#333";
      ctx.stroke();
    }

    function restartQuiz() {
      location.reload();
    }

    function copyPrompt() {
      navigator.clipboard.writeText(document.getElementById("promptOutput").textContent)
        .then(() => alert("已复制Prompt"));
    }
  </script>
</body>
</html>
