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
    let results = [];
    geneEffects.forEach(effect => {
      if (effect.condition(normalizedScores)) {
        results.push({ type: effect.type, description: effect.description });
      }
    });
    return results;
  }

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

    for (let i = 0; i < q.options.length; i++) {
      const opt = document.createElement("div");
      opt.className = "option";
      opt.textContent = q.options[i].text;
      opt.onclick = () => selectOption(i);
      optionsEl.appendChild(opt);
    }
    progressText.textContent = `第 ${questionCount + 1} / ${questionPool.length} 题`;
  }

  function showResult() {
    sectionQuiz.style.display = "none";
    sectionResult.style.display = "block";

    const normalizedScores = geneDimensions.map(dim => {
      const raw = tagScores[dim.key] || 0;
      let norm = (raw + 8) / 16 * 100; // 0~100
      norm = Math.min(100, Math.max(0, norm));
      return +norm.toFixed(1);
    });
    const scoreObj = {};
    geneDimensions.forEach((dim, idx) => {
      scoreObj[dim.key] = normalizedScores[idx];
    });

    const totalScoreRaw = normalizedScores.reduce((a, b) => a + b, 0) / normalizedScores.length;
    const totalScore = totalScoreRaw.toFixed(1);
    totalScoreText.textContent = `总分: ${totalScore}`;

    drawRadarChart(tagScores, age, gender);

    let prompt = `Age: ${age}, Gender: ${gender === "male" ? "Male" : "Female"}\n`;
    geneDimensions.forEach((dim, idx) => {
      prompt += `${dim.label}: ${normalizedScores[idx]} / 100\n`;
    });
    promptOutput.textContent = prompt.trim();

    let prevGeneTypeDivs = sectionResult.querySelectorAll('.gene-type-result');
    prevGeneTypeDivs.forEach(div => div.remove());

    const geneTypes = getGeneType(scoreObj);
    if (geneTypes.length) {
      geneTypes.forEach(t => {
        const div = document.createElement("div");
        div.className = "gene-type-result";
        div.innerHTML = `<b>${t.type}</b>：${t.description}`;
        sectionResult.appendChild(div);
      });
    } else {
      const div = document.createElement("div");
      div.className = "gene-type-result";
      div.innerHTML = `<b>普通型</b>：你的人格较为均衡。`;
      sectionResult.appendChild(div);
    }
  }

  function drawRadarChart(scores, age, gender) {
    // 这里可以补充你的雷达图实现
  }

  // 题库：可自由增加问题！
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
  { id: "Q3", text: "你喜欢尝试新鲜的体验或爱好？", options: [
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
  ]}
];  // 继续添加任意多的问题...
  ];

  btnNext.onclick = () => {
    if (selectedOptionIndex === null) return;
    const selTags = currentQuestion.options[selectedOptionIndex].tags;
    for (let t in selTags) {
      tagScores[t] = (tagScores[t] || 0) + selTags[t];
    }
    askedIds.add(currentQuestion.id);
    questionCount++;
    const nextQ = questionPool[questionCount] || null;
    if (!nextQ) {
      showResult();
    } else {
      renderQuestion(nextQ);
    }
  };

  btnPrev.onclick = () => {
    questionCount--;
    const prevQ = questionPool[questionCount];
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
    renderQuestion(questionPool[questionCount]);
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