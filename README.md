<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>因•趣问答</title>
  <style>
    body {
      max-width: 600px;
      margin: 20px auto;
      font-family: "微软雅黑", sans-serif;
      background: #ffffff; /* 默认背景为白色 */
      padding: 20px;
      color: #222;
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
      transition: background-color 0.3s ease;
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
  </style>
</head>
<body>

<h2> 趣问答& 因得分</h2>

<div id="sectionUserInfo">
  <label for="inputAge">年龄（不限）</label>
  <input type="number" id="inputAge" min="0" value="25" />
  <label for="selectGender">性别</label>
  <select id="selectGender">
    <option value="male">男性</option>
    <option value="female">女性</option>
  </select>
  <button id="btnStart">开始答题</button>
</div>

<div id="sectionQuiz" style="display:none;">
  <div id="questionText" class="question" style="font-weight:700; margin-bottom:10px;"></div>
  <div id="optionsContainer"></div>
  <button id="btnPrev" disabled>上一题</button>
  <button id="btnNext" disabled>下一题</button>
  <div id="progressText"></div>
</div>

<div id="sectionResult" style="display:none;">
  <h3>雷达图基因维度评分</h3>
  <canvas id="radarChart" width="400" height="400"></canvas>
  <div id="totalScoreText"></div>
  <h3>基因风格Prompt（点击可复制）</h3>
  <pre id="promptOutput" title="点击复制"></pre>
  <button id="btnRestart">重新开始</button>
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
    { id: "Q1", text: "你喜欢参加社交活动吗？", options: [
      { text: "非常不喜欢", tags: { extraversion: -2 } },
      { text: "不太喜欢", tags: { extraversion: -1 } },
      { text: "比较喜欢", tags: { extraversion: 1 } },
      { text: "非常喜欢", tags: { extraversion: 2 } }
    ]},
    { id: "Q2", text: "面对压力时你的情绪表现？", options: [
      { text: "非常紧张和焦虑", tags: { emotion_stability: -2, self_control: -1 } },
      { text: "有些不安", tags: { emotion_stability: -1, self_control: 0 } },
      { text: "情绪稳定", tags: { emotion_stability: 1, self_control: 1 } },
      { text: "非常冷静", tags: { emotion_stability: 2, self_control: 2 } }
    ]},
    // ... 其他问题
  ];

  let tagScores = {};
  let askedIds = new Set();
  let currentQuestion = null;
  let selectedOptionIndex = null;
  let questionCount = 0;
  const maxQuestions = 12;
  let age = 25;
  let gender = "male";
  const sectionUserInfo = document.getElementById("sectionUserInfo");
  const sectionQuiz = document.getElementById("sectionQuiz");
  const sectionResult = document.getElementById("sectionResult");
  const qTextEl = document.getElementById("questionText");
  const optionsEl = document.getElementById("optionsContainer");
  const btnNext = document.getElementById("btnNext");
  const btnPrev = document.getElementById("btnPrev");
  const progressText = document.getElementById("progressText");
  const promptOutput = document.getElementById("promptOutput");
  const btnStart = document.getElementById("btnStart");
  const btnRestart = document.getElementById("btnRestart");
  const inputAge = document.getElementById("inputAge");
  const selectGender = document.getElementById("selectGender");
  const totalScoreText = document.getElementById("totalScoreText");
  const radarCanvas = document.getElementById("radarChart");
  const ctx = radarCanvas.getContext("2d");

  function selectOption(idx) {
    selectedOptionIndex = idx;
    Array.from(optionsEl.children).forEach((el, i) => {
      el.classList.toggle("selected", i === idx);
    });
    btnNext.disabled = false;
  }

  function renderQuestion(q) {
    currentQuestion = q;
    qTextEl.textContent = q.text;
    optionsEl.innerHTML = "";
    selectedOptionIndex = null;
    btnNext.disabled = true;
    btnPrev.disabled = questionCount <= 0;

    // 页面颜色变化逻辑
    updatePageColor();

    for (let i = 0; i < q.options.length; i++) {
      const opt = document.createElement("div");
      opt.className = "option";
      opt.textContent = q.options[i].text;
      opt.onclick = () => selectOption(i);
      optionsEl.appendChild(opt);
    }
    progressText.textContent = `第 ${questionCount + 1} / ${maxQuestions} 题`;
  }

  function selectNextQuestion() {
    for (let q of questionPool) {
      if (!askedIds.has(q.id)) return q;
    }
    return null;
  }

  function selectPrevQuestion() {
    for (let q of questionPool) {
      if (askedIds.has(q.id)) return q;
    }
    return null;
  }

  btnNext.onclick = () => {
    if (selectedOptionIndex === null) return;
    const selTags = currentQuestion.options[selectedOptionIndex].tags;
    for (let t in selTags) {
      tagScores[t] = (tagScores[t] || 0) + selTags[t];
    }
    askedIds.add(currentQuestion.id);
    questionCount++;
    const nextQ = selectNextQuestion();
    if (!nextQ) {
      showResult();
    } else {
      renderQuestion(nextQ);
    }
  };

  btnPrev.onclick = () => {
    questionCount--;
    const prevQ = selectPrevQuestion();
    if (!prevQ) return;
    renderQuestion(prevQ);
  };

  btnStart.onclick = () => {
    age = parseInt(inputAge.value);
    if (isNaN(age) || age < 0) {
      alert("请输入有效年龄（非负整数）");
      return;
    }
    gender = selectGender.value;
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

  // 更新页面背景色（颜色根据基因评分变化）
  function updatePageColor() {
    const score = tagScores.extraversion || 0;  // 以外向性为例
    const intensity = Math.min(Math.abs(score) * 30, 255); // 根据得分计算强度
    document.body.style.backgroundColor = `rgb(${255 - intensity}, ${255 - intensity}, ${255})`;
    document.body.style.color = score < 0 ? '#333' : '#fff';
  }

  function showResult() {
    sectionQuiz.style.display = "none";
    sectionResult.style.display = "block";

    // 计算归一化分数0~100，保留一位小数
    const normalizedScores = geneDimensions.map(dim => {
      const raw = tagScores[dim.key] || 0;
      let norm = (raw + 8) / 16 * 100; // 0~100
      norm = Math.min(100, Math.max(0, norm));
      return +norm.toFixed(1);
    });

    // 总分 = 六维度平均，0~100，1位小数
    const totalScoreRaw = normalizedScores.reduce((a, b) => a + b, 0) / normalizedScores.length;
    const totalScore = totalScoreRaw.toFixed(1);
    totalScoreText.textContent = `总分: ${totalScore}`;

    // 画雷达图
    drawRadarChart(tagScores, age, gender);

    let prompt = `Age: ${age}, Gender: ${gender === "male" ? "Male" : "Female"}\n`;
    geneDimensions.forEach((dim, idx) => {
      prompt += `${dim.label}: ${normalizedScores[idx]} / 100\n`;
    });

    promptOutput.textContent = prompt.trim();
  }

  // 绘制雷达图
  function drawRadarChart(scores, age, gender) {
    // ... 绘制雷达图的代码
  }

  promptOutput.onclick = () => {
    navigator.clipboard.writeText(promptOutput.textContent).then(() => {
      alert("已复制Prompt文本");
    });
  };
</script>

</body>
</html>
