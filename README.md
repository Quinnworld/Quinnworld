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
      background: #ffffff; 
      padding: 20px;
      color: #222;
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
      white-space: pre-wrap; overflow-x: auto; background: #eee; padding: 10px; border-radius: 6px; font-family: monospace;
      user-select: all;
    }
    #totalScoreText {
      text-align: center; font-weight: bold; margin: 10px 0;
      font-size: 32px;
      transition: color 0.5s;
    }
    canvas {
      display: block;
      margin: 0 auto 20px;
      max-width: 100%;
      background: #fff;
    }
    #progressText {
      font-size: 14px; color: #555; text-align: center; margin-top: 5px;
    }
    .gene-type-result {
      font-size: 17px;
      margin: 15px 0;
      padding: 12px;
      background: #f2f7ff;
      border-left: 5px solid #4338ca;
      border-radius: 6px;
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
    <option value="other">其他</option>
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

  const geneTypeColors = {
    "冒险家型": "#FF5722",     // Orange-Red
    "创新型": "#8E24AA",       // Purple
    "自由浪漫型": "#1DE9B6",   // Cyan-Green
    "可靠型": "#1976D2",       // Blue
    "内向敏感型": "#388E3C",   // Deep Green
    "普通型": "#888888"        // Gray
  };

  const geneEffects = [
    {
      condition: (scores) => scores.extraversion > 70 && scores.novelty_seek > 70,
      type: "冒险家型",
      description: "你具有极强的冒险精神和探索欲望，喜欢尝试新鲜事物。"
    },
    {
      condition: (scores) => scores.responsibility > 80 && scores.emotion_stability > 80,
      type: "可靠型",
      description: "你非常可靠且情绪稳定，是团队的中坚力量。"
    },
    {
      condition: (scores) => scores.openness > 75 && scores.novelty_seek > 75,
      type: "创新型",
      description: "你有很强的创新意识和接受新事物的能力。"
    },
    {
      condition: (scores) => scores.self_control < 30 && scores.novelty_seek > 80,
      type: "自由浪漫型",
      description: "你崇尚自由，不喜欢被约束，追求独特体验。"
    },
    {
      condition: (scores) => scores.extraversion < 30 && scores.emotion_stability < 30,
      type: "内向敏感型",
      description: "你性格内向，情绪较敏感，喜欢安静的环境。"
    }
  ];

  function getGeneType(normalizedScores) {
    for (const effect of geneEffects) {
      if (effect.condition(normalizedScores)) {
        return { type: effect.type, description: effect.description };
      }
    }
    return { type: "普通型", description: "你的人格较为均衡。" };
  }

  // 题库15题
  const questionPool = [
    { id: "Q1", text: "你喜欢参加热闹的聚会吗？", options: [
      { text: "非常不喜欢", tags: { extraversion: -2 } },
      { text: "不太喜欢", tags: { extraversion: -1 } },
      { text: "比较喜欢", tags: { extraversion: 1 } },
      { text: "非常喜欢", tags: { extraversion: 2 } }
    ]},
    { id: "Q2", text: "遇到突发问题时，你能迅速冷静下来吗？", options: [
      { text: "很难冷静", tags: { emotion_stability: -2, self_control: -1 } },
      { text: "偶尔能冷静", tags: { emotion_stability: -1 } },
      { text: "大多能冷静", tags: { emotion_stability: 1, self_control: 1 } },
      { text: "总能冷静", tags: { emotion_stability: 2, self_control: 2 } }
    ]},
    { id: "Q3", text: "你喜欢尝试新鲜的体验或爱好吗？", options: [
      { text: "完全不喜欢", tags: { novelty_seek: -2 } },
      { text: "不太喜欢", tags: { novelty_seek: -1 } },
      { text: "比较喜欢", tags: { novelty_seek: 1 } },
      { text: "非常喜欢", tags: { novelty_seek: 2 } }
    ]},
    { id: "Q4", text: "你会主动规划和安排自己的任务吗？", options: [
      { text: "几乎不会", tags: { responsibility: -2 } },
      { text: "偶尔会", tags: { responsibility: -1 } },
      { text: "经常会", tags: { responsibility: 1 } },
      { text: "总是会", tags: { responsibility: 2 } }
    ]},
    { id: "Q5", text: "你能自觉克制住即刻的欲望吗？", options: [
      { text: "很难控制", tags: { self_control: -2 } },
      { text: "偶尔能控制", tags: { self_control: -1 } },
      { text: "大多能控制", tags: { self_control: 1 } },
      { text: "非常容易控制", tags: { self_control: 2 } }
    ]},
    { id: "Q6", text: "你喜欢结识不同背景的人吗？", options: [
      { text: "完全不喜欢", tags: { openness: -2 } },
      { text: "不太喜欢", tags: { openness: -1 } },
      { text: "比较喜欢", tags: { openness: 1 } },
      { text: "非常喜欢", tags: { openness: 2 } }
    ]},
    { id: "Q7", text: "遇到压力时，你会主动寻求社交支持吗？", options: [
      { text: "很少会", tags: { extraversion: -1, emotion_stability: -1 } },
      { text: "偶尔会", tags: { extraversion: 0 } },
      { text: "经常会", tags: { extraversion: 1, emotion_stability: 1 } },
      { text: "总是会", tags: { extraversion: 2, emotion_stability: 2 } }
    ]},
    { id: "Q8", text: "你对新环境适应能力如何？", options: [
      { text: "很难适应", tags: { novelty_seek: -1, openness: -1 } },
      { text: "适应较慢", tags: { openness: 0 } },
      { text: "适应较快", tags: { novelty_seek: 1, openness: 1 } },
      { text: "非常容易适应", tags: { novelty_seek: 2, openness: 2 } }
    ]},
    { id: "Q9", text: "你习惯按计划一步步完成目标吗？", options: [
      { text: "很少按计划", tags: { responsibility: -2, self_control: -1 } },
      { text: "偶尔按计划", tags: { responsibility: -1 } },
      { text: "经常按计划", tags: { responsibility: 1, self_control: 1 } },
      { text: "总是按计划", tags: { responsibility: 2, self_control: 2 } }
    ]},
    { id: "Q10", text: "你容易被情绪影响做出冲动决定吗？", options: [
      { text: "经常如此", tags: { self_control: -2, emotion_stability: -1 } },
      { text: "偶尔如此", tags: { self_control: -1 } },
      { text: "较少如此", tags: { self_control: 1 } },
      { text: "几乎不会", tags: { self_control: 2, emotion_stability: 1 } }
    ]},
    { id: "Q11", text: "你乐于接受与自己观点不同的事物吗？", options: [
      { text: "很难接受", tags: { openness: -2 } },
      { text: "偶尔接受", tags: { openness: -1 } },
      { text: "较易接受", tags: { openness: 1 } },
      { text: "非常乐于接受", tags: { openness: 2 } }
    ]},
    { id: "Q12", text: "面对长期目标，你能持续保持动力吗？", options: [
      { text: "很难保持", tags: { responsibility: -2, self_control: -1 } },
      { text: "偶尔能保持", tags: { responsibility: -1 } },
      { text: "大多能保持", tags: { responsibility: 1, self_control: 1 } },
      { text: "始终充满动力", tags: { responsibility: 2, self_control: 2 } }
    ]},
    { id: "Q13", text: "你喜欢和朋友讨论新鲜话题吗？", options: [
      { text: "很少", tags: { extraversion: -1, openness: -1 } },
      { text: "偶尔", tags: { extraversion: 0 } },
      { text: "经常", tags: { extraversion: 1, openness: 1 } },
      { text: "总是", tags: { extraversion: 2, openness: 2 } }
    ]},
    { id: "Q14", text: "你做事容易虎头蛇尾吗？", options: [
      { text: "经常如此", tags: { responsibility: -2, self_control: -2 } },
      { text: "偶尔如此", tags: { responsibility: -1 } },
      { text: "较少如此", tags: { responsibility: 1 } },
      { text: "几乎不会", tags: { responsibility: 2, self_control: 2 } }
    ]},
    { id: "Q15", text: "你喜欢独自一人享受安静时光吗？", options: [
      { text: "非常喜欢", tags: { extraversion: -2, emotion_stability: 1 } },
      { text: "比较喜欢", tags: { extraversion: -1 } },
      { text: "偶尔喜欢", tags: { extraversion: 1 } },
      { text: "很少喜欢", tags: { extraversion: 2 } }
    ]}
  ];

  // 用于本次答题的抽取题目
  let currentQuestions = [];

  let tagScores = {};
  let askedIds = new Set();
  let currentQuestion = null;
  let selectedOptionIndex = null;
  let questionCount = 0;
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

  function drawQuestions(pool, count) {
    const copy = pool.slice();
    const result = [];
    for (let i = 0; i < count; i++) {
      const idx = Math.floor(Math.random() * copy.length);
      result.push(copy.splice(idx, 1)[0]);
    }
    return result;
  }

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

    for (let i = 0; i < q.options.length; i++) {
      const opt = document.createElement("div");
      opt.className = "option";
      opt.textContent = q.options[i].text;
      opt.onclick = () => selectOption(i);
      optionsEl.appendChild(opt);
    }
    progressText.textContent = `第 ${questionCount + 1} / ${currentQuestions.length} 题`;
  }

  function showResult() {
    sectionQuiz.style.display = "none";
    sectionResult.style.display = "block";

    // 归一化分数 0~1，精度4位
    const normalizedScores = geneDimensions.map(dim => {
      const raw = tagScores[dim.key] || 0;
      let norm = (raw + 8) / 16;
      norm = Math.min(1, Math.max(0, norm));
      norm = Number.isFinite(norm) ? norm : 0;
      return +norm.toFixed(4);
    });
    // 0~100分
    const scoreObj = {};
    geneDimensions.forEach((dim, idx) => {
      scoreObj[dim.key] = +(normalizedScores[idx]*100).toFixed(1);
    });

    // 总分
    const totalScoreRaw = normalizedScores.reduce((a, b) => a + b, 0) / normalizedScores.length;
    const totalScore = (totalScoreRaw*100).toFixed(1);

    // 基因型与色调
    const geneTypeObj = getGeneType(scoreObj);
    const geneType = geneTypeObj.type;
    const geneColor = geneTypeColors[geneType] || "#888888";

    // 总分颜色
    totalScoreText.textContent = `总分: ${totalScore}`;
    totalScoreText.style.color = geneColor;

    drawRadarChart(normalizedScores);

    // prompt输出为纯分数，便于绘图
    let prompt =
      geneDimensions.map((dim, idx) => `${dim.label}: ${scoreObj[dim.key]}`).join('\n');
    promptOutput.textContent = prompt;

    // 展示复合基因型标签
    let prevGeneTypeDivs = sectionResult.querySelectorAll('.gene-type-result');
    prevGeneTypeDivs.forEach(div => div.remove());
    const div = document.createElement("div");
    div.className = "gene-type-result";
    div.innerHTML = `<b>${geneType}</b>：${geneTypeObj.description}`;
    sectionResult.appendChild(div);
  }

  // 画雷达图，归一化分数数组传入
  function drawRadarChart(normArr) {
    ctx.clearRect(0, 0, radarCanvas.width, radarCanvas.height);
    const labels = geneDimensions.map(d => d.label);
    const center = 200, radius = 140;
    const count = labels.length;
    const angleStep = 2 * Math.PI / count;
    // 网格
    ctx.strokeStyle = "#ccc";
    ctx.lineWidth = 1;
    for (let r = 0.2; r <= 1; r += 0.2) {
      ctx.beginPath();
      for (let i = 0; i < count; i++) {
        const angle = angleStep * i - Math.PI/2;
        const x = center + Math.cos(angle) * radius * r;
        const y = center + Math.sin(angle) * radius * r;
        if (i === 0) ctx.moveTo(x, y);
        else ctx.lineTo(x, y);
      }
      ctx.closePath();
      ctx.stroke();
    }
    // 维度连线
    ctx.strokeStyle = "#bbb";
    for (let i = 0; i < count; i++) {
      const angle = angleStep * i - Math.PI/2;
      ctx.beginPath();
      ctx.moveTo(center, center);
      ctx.lineTo(center + Math.cos(angle)*radius, center + Math.sin(angle)*radius);
      ctx.stroke();
    }
    // 画得分区
    ctx.beginPath();
    for (let i = 0; i < count; i++) {
      const angle = angleStep*i - Math.PI/2;
      const r = normArr[i];
      const x = center + Math.cos(angle) * radius * r;
      const y = center + Math.sin(angle) * radius * r;
      if (i === 0) ctx.moveTo(x, y);
      else ctx.lineTo(x, y);
    }
    ctx.closePath();
    ctx.fillStyle = "rgba(67,56,202,0.18)";
    ctx.fill();
    ctx.strokeStyle = "#4338ca";
    ctx.lineWidth = 2;
    ctx.stroke();
    // 顶点圆点
    ctx.fillStyle = "#4338ca";
    for (let i = 0; i < count; i++) {
      const angle = angleStep*i - Math.PI/2;
      const r = normArr[i];
      const x = center + Math.cos(angle) * radius * r;
      const y = center + Math.sin(angle) * radius * r;
      ctx.beginPath();
      ctx.arc(x, y, 4, 0, 2*Math.PI);
      ctx.fill();
    }
    // 标签
    ctx.font = "16px 微软雅黑";
    ctx.fillStyle = "#222";
    for (let i = 0; i < count; i++) {
      const angle = angleStep*i - Math.PI/2;
      const x = center + Math.cos(angle) * (radius + 20);
      const y = center + Math.sin(angle) * (radius + 20);
      ctx.textAlign = "center";
      ctx.textBaseline = "middle";
      ctx.fillText(labels[i], x, y);
    }
  }

  // 答题逻辑，动态抽题，每次12题
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
    currentQuestions = drawQuestions(questionPool, 12);
    sectionUserInfo.style.display = "none";
    sectionResult.style.display = "none";
    sectionQuiz.style.display = "block";
    renderQuestion(currentQuestions[questionCount]);
  };

  btnNext.onclick = () => {
    if (selectedOptionIndex === null) return;
    const selTags = currentQuestion.options[selectedOptionIndex].tags;
    for (let t in selTags) {
      tagScores[t] = (tagScores[t] || 0) + selTags[t];
    }
    askedIds.add(currentQuestion.id);
    questionCount++;
    const nextQ = currentQuestions[questionCount] || null;
    if (!nextQ) {
      showResult();
    } else {
      renderQuestion(nextQ);
    }
  };

  btnPrev.onclick = () => {
    questionCount--;
    const prevQ = currentQuestions[questionCount];
    if (!prevQ) return;
    renderQuestion(prevQ);
  };

  btnRestart.onclick = () => {
    tagScores = {};
    askedIds.clear();
    questionCount = 0;
    sectionResult.style.display = "none";
    sectionQuiz.style.display = "none";
    sectionUserInfo.style.display = "block";
  };
</script>

</body>
</html>