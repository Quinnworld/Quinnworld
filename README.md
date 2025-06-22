<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>因•趣问答</title>
<style>
  body {
    max-width: 600px; margin: 20px auto; font-family: "微软雅黑", sans-serif;
    background: #f0f4f8; padding: 20px; color:#222;
    transition: background-color 0.3s ease, color 0.3s ease;
  }
  h2, h3 { text-align: center; }
  label, select, input[type=number] {
    width: 100%; margin: 10px 0 15px; font-size: 16px;
  }
  select, input[type=number] {
    padding: 8px; border-radius: 6px; border: 1px solid #ccc;
  }
  button {
    width: 100%; padding: 12px; font-size: 16px; border: none;
    border-radius: 6px; cursor: pointer;
    color: white;
    background: #4338ca;
    transition: background 0.3s ease;
  }
  button:disabled {
    background: #a5b4fc; cursor: not-allowed;
  }
  .option {
    background: white; border: 1px solid #bbb; border-radius: 6px;
    padding: 10px; margin: 6px 0; cursor: pointer; user-select:none;
  }
  .option.selected {
    background: #4f46e5; color: white; border-color: #4338ca;
  }
  #promptOutput {
    white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
    background: #eee; padding: 10px; border-radius: 6px; font-family: monospace;
    user-select: all; /* 方便复制 */
  }
  #totalScoreText {
    text-align: center; font-weight: bold; margin: 10px 0;
    font-size: 18px;
  }
  canvas {
    display: block;
    margin: 0 auto 20px;
    max-width: 100%;
  }
  #progressText {
    font-size: 14px; color: #555; text-align: center; margin-top: 5px;
  }
  .button-container {
    display: flex;
    justify-content: space-between;
  }
  .button-container button {
    width: 48%;
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
  <div id="questionText" class="question" style="font-weight:700; margin-bottom:10px;"></div>
  <div id="optionsContainer"></div>
  <div class="button-container">
    <button id="btnPrev" disabled>上一题</button>
    <button id="btnNext" disabled>下一题</button>
  </div>
  <div id="progressText"></div>
</div>

<div id="sectionResult" style="display:none;">
  <h3>雷达图因维度评分</h3>
  <canvas id="radarChart" width="400" height="400"></canvas>
  <div id="totalScoreText"></div>
  <h3>因风格Prompt（点击可复制）</h3>
  <pre id="promptOutput" title="点击复制"></pre>
  <button id="btnRestart">重新开始</button>
</div>

<script>
  // 六个因维度（中文）
  const geneDimensions = [
    { key: "extraversion", label: "外向性" },
    { key: "emotion_stability", label: "情绪稳定性" },
    { key: "novelty_seek", label: "新奇寻求" },
    { key: "responsibility", label: "责任感" },
    { key: "self_control", label: "自控力" },
    { key: "openness", label: "开放性" }
  ];

  // 12题题库，每题4选项，对应六个因维度评分-2~2
  const questionPool = [
    {
      id: "Q1", text: "你喜欢参加社交活动吗？",
      options: [
        { text: "非常不喜欢", tags: { extraversion: -2 } },
        { text: "不太喜欢", tags: { extraversion: -1 } },
        { text: "比较喜欢", tags: { extraversion: 1 } },
        { text: "非常喜欢", tags: { extraversion: 2 } }
      ]
    },
    {
      id: "Q2", text: "面对压力时你的情绪表现？",
      options: [
        { text: "非常紧张和焦虑", tags: { emotion_stability: -2, self_control: -1 } },
        { text: "有些不安", tags: { emotion_stability: -1, self_control: 0 } },
        { text: "情绪稳定", tags: { emotion_stability: 1, self_control: 1 } },
        { text: "非常冷静", tags: { emotion_stability: 2, self_control: 2 } }
      ]
    },
    {
      id: "Q3", text: "你喜欢尝试新鲜事物吗？",
      options: [
        { text: "完全不喜欢", tags: { novelty_seek: -2, openness: -1 } },
        { text: "不太喜欢", tags: { novelty_seek: -1, openness: 0 } },
        { text: "比较喜欢", tags: { novelty_seek: 1, openness: 1 } },
        { text: "非常喜欢", tags: { novelty_seek: 2, openness: 2 } }
      ]
    },
    {
      id: "Q4", text: "你通常是否按时完成任务？",
      options: [
        { text: "经常拖延", tags: { responsibility: -2 } },
        { text: "偶尔拖延", tags: { responsibility: -1 } },
        { text: "大部分时间按时", tags: { responsibility: 1 } },
        { text: "总是按时完成", tags: { responsibility: 2 } }
      ]
    },
    {
      id: "Q5", text: "你做决定时是否容易冲动？",
      options: [
        { text: "完全不会冲动", tags: { self_control: 2 } },
        { text: "不太冲动", tags: { self_control: 1 } },
        { text: "有时冲动", tags: { self_control: -1 } },
        { text: "非常冲动", tags: { self_control: -2 } }
      ]
    },
    {
      id: "Q6", text: "你是否喜欢探索未知？",
      options: [
        { text: "完全不喜欢", tags: { novelty_seek: -2 } },
        { text: "不太喜欢", tags: { novelty_seek: -1 } },
        { text: "比较喜欢", tags: { novelty_seek: 1 } },
        { text: "非常喜欢", tags: { novelty_seek: 2 } }
      ]
    },
    {
      id: "Q7", text: "你是否喜欢有条理地生活和工作？",
      options: [
        { text: "不喜欢", tags: { responsibility: -2, openness: -1 } },
        { text: "偶尔", tags: { responsibility: -1, openness: 0 } },
        { text: "经常", tags: { responsibility: 1, openness: 1 } },
        { text: "总是", tags: { responsibility: 2, openness: 2 } }
      ]
    },
    {
      id: "Q8", text: "你控制情绪的能力如何？",
      options: [
        { text: "很差", tags: { emotion_stability: -2, self_control: -2 } },
        { text: "一般", tags: { emotion_stability: -1, self_control: -1 } },
        { text: "较好", tags: { emotion_stability: 1, self_control: 1 } },
        { text: "非常好", tags: { emotion_stability: 2, self_control: 2 } }
      ]
    },
    {
      id: "Q9", text: "你喜欢挑战和冒险吗？",
      options: [
        { text: "完全不喜欢", tags: { novelty_seek: -2 } },
        { text: "不太喜欢", tags: { novelty_seek: -1 } },
        { text: "有点喜欢", tags: { novelty_seek: 1 } },
        { text: "非常喜欢", tags: { novelty_seek: 2 } }
      ]
    },
    {
      id: "Q10", text: "你是否有强烈的好奇心？",
      options: [
        { text: "没有", tags: { openness: -2 } },
        { text: "不太好奇", tags: { openness: -1 } },
        { text: "有点好奇", tags: { openness: 1 } },
        { text: "非常好奇", tags: { openness: 2 } }
      ]
    },
    {
      id: "Q11", text: "你是否倾向于保持乐观态度？",
      options: [
        { text: "非常悲观", tags: { extraversion: -2 } },
        { text: "比较悲观", tags: { extraversion: -1 } },
        { text: "一般", tags: { extraversion: 1 } },
        { text: "非常乐观", tags: { extraversion: 2 } }
      ]
    },
    {
      id: "Q12", text: "你是否喜欢有规律的生活？",
      options: [
        { text: "完全不喜欢", tags: { responsibility: -2 } },
        { text: "偶尔喜欢", tags: { responsibility: -1 } },
        { text: "比较喜欢", tags: { responsibility: 1 } },
        { text: "非常喜欢", tags: { responsibility: 2 } }
      ]
    }
  ];

  let tagScores = {};
  let askedIds = new Set();
  let questionCount = 0;
  const maxQuestions = questionPool.length;
  let age = 25;
  let gender = "male";
  let selectedOptionIndex = null;
  let currentQuestion = null;

  const sectionUserInfo = document.getElementById("sectionUserInfo");
  const sectionQuiz = document.getElementById("sectionQuiz");
  const sectionResult = document.getElementById("sectionResult");
  const qTextEl = document.getElementById("questionText");
  const optionsEl = document.getElementById("optionsContainer");
  const btnNext = document.getElementById("btnNext");
  const btnPrev = document.getElementById("btnPrev");
  const progressText = document.getElementById("progressText");
  const totalScoreText = document.getElementById("totalScoreText");
  const promptOutput = document.getElementById("promptOutput");
  const radarCanvas = document.getElementById("radarChart");
  const ctx = radarCanvas.getContext("2d");

  function selectOption(idx) {
    selectedOptionIndex = idx;
    Array.from(optionsEl.children).forEach((el, i) => {
      el.classList.toggle("selected", i === idx);
    });
    btnNext.disabled = false;
    updatePageStyle(idx);
  }

  function updatePageStyle(idx) {
    const selectedOption = currentQuestion.options[idx];
    const tags = selectedOption.tags;
    const extraversionScore = tags.extraversion || 0;
    const colorIntensity = (extraversionScore + 2) / 4; // Normalize color intensity based on score

    // Update the page background color and button colors based on score
    const backgroundColor = `rgba(${255 - colorIntensity * 255}, ${colorIntensity * 255}, 200, 0.1)`;
    document.body.style.backgroundColor = backgroundColor;
    btnNext.style.backgroundColor = `rgba(0, 0, 0, ${colorIntensity})`;
  }

  function renderQuestion(q) {
    currentQuestion = q;
    qTextEl.textContent = q.text;
    optionsEl.innerHTML = "";
    selectedOptionIndex = null;
    btnNext.disabled = true;
    for (let i = 0; i < q.options.length; i++) {
      const opt = document.createElement("div");
      opt.className = "option";
      opt.textContent = q.options[i].text;
      opt.onclick = () => selectOption(i);
      optionsEl.appendChild(opt);
    }
    progressText.textContent = `第 ${questionCount + 1} / ${maxQuestions} 题`;
    btnPrev.disabled = questionCount === 0;
  }

  function selectNextQuestion() {
    return questionPool[questionCount];
  }

  btnNext.onclick = () => {
    if (selectedOptionIndex === null) return;
    const selTags = currentQuestion.options[selectedOptionIndex].tags;
    for (let t in selTags) {
      tagScores[t] = (tagScores[t] || 0) + selTags[t];
    }
    askedIds.add(currentQuestion.id);
    questionCount++;
    if (questionCount >= maxQuestions) {
      showResult();
    } else {
      const nextQ = selectNextQuestion();
      renderQuestion(nextQ);
    }
  };

  btnPrev.onclick = () => {
    if (questionCount > 0) {
      questionCount--;
      const prevQ = selectNextQuestion();
      renderQuestion(prevQ);
    }
  };

  btnStart.onclick = () => {
    age = parseInt(document.getElementById("inputAge").value);
    if (isNaN(age) || age < 0) {
      alert("请输入有效年龄（非负整数）");
      return;
    }
    gender = document.getElementById("selectGender").value;
    tagScores = {};
    askedIds.clear();
    questionCount = 0;
    sectionUserInfo.style.display = "none";
    sectionResult.style.display = "none";
    sectionQuiz.style.display = "block";
    const firstQ = selectNextQuestion();
    renderQuestion(firstQ);
  };

  btnRestart.onclick = () => {
    tagScores = {};
    askedIds.clear();
    questionCount = 0;
    sectionResult.style.display = "none";
    sectionQuiz.style.display = "none";
    sectionUserInfo.style.display = "block";
  };

  function drawRadarChart(scores) {
    const w = radarCanvas.width;
    const h = radarCanvas.height;
    ctx.clearRect(0, 0, w, h);
    const cx = w / 2;
    const cy = h / 2;
    const radius = Math.min(cx, cy) * 0.75;
    const dimCount = geneDimensions.length;
    const angleStep = (2 * Math.PI) / dimCount;

    // Draw the radar chart (as earlier)
    ctx.beginPath();
    for (let i = 0; i < dimCount; i++) {
      const r = scores[i] * radius;
      const x = cx + r * Math.sin(i * angleStep);
      const y = cy - r * Math.cos(i * angleStep);
      if (i === 0) ctx.moveTo(x, y);
      else ctx.lineTo(x, y);
    }
    ctx.closePath();
    ctx.fillStyle = "rgba(75, 192, 192, 0.4)";
    ctx.strokeStyle = "rgba(75, 192, 192, 1)";
    ctx.fill();
    ctx.stroke();
  }

  function showResult() {
    sectionQuiz.style.display = "none";
    sectionResult.style.display = "block";

    // Calculate scores (normalize to 0-100)
    const normalizedScores = geneDimensions.map(dim => {
      const raw = tagScores[dim.key] || 0;
      let norm = (raw + 8) / 16 * 100; // 0~100
      norm = Math.min(100, Math.max(0, norm));
      return norm;
    });

    // Display total score
    const totalScore = normalizedScores.reduce((a, b) => a + b, 0) / normalizedScores.length;
    totalScoreText.textContent = `总分: ${totalScore.toFixed(1)}`;

    // Draw radar chart
    drawRadarChart(normalizedScores);

    // Create prompt
    let prompt = `年龄: ${age}, 性别: ${gender === "male" ? "男性" : gender === "female" ? "女性" : "其他"}\n`;
    normalizedScores.forEach((score, idx) => {
      prompt += `${geneDimensions[idx].label}: ${score.toFixed(1)} / 100\n`;
    });

    promptOutput.textContent = prompt.trim();
  }

  // Copy prompt to clipboard
  promptOutput.onclick = () => {
    navigator.clipboard.writeText(promptOutput.textContent).then(() => {
      alert("已复制Prompt文本");
    });
  };
</script>

</body>
</html>
