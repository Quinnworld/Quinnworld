<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>心理测评系统示范</title>
<style>
  body {
    max-width: 600px; margin: 20px auto; font-family: "微软雅黑", sans-serif;
    background: #fff; padding: 20px; color:#000;
  }
  h2, h3 { text-align: center; }
  label, select, input[type=number] {
    width: 100%; margin: 10px 0 15px; font-size: 16px;
  }
  select, input[type=number] {
    padding: 8px; border-radius: 6px; border: 1px solid #ccc;
  }
  button {
    width: 48%; padding: 12px; font-size: 16px; border: none;
    border-radius: 6px; cursor: pointer;
    color: white;
    background: #4338ca;
    margin: 5px 1%;
    box-sizing: border-box;
  }
  button:disabled {
    background: #a5b4fc; cursor: not-allowed;
  }
  .option {
    background: white; border: 1px solid #bbb; border-radius: 6px;
    padding: 10px; margin: 6px 0; cursor: pointer; user-select:none;
    color: black;
    transition: background-color 0.3s, color 0.3s;
  }
  .option.selected {
    background: #4f46e5; color: white; border-color: #4338ca;
  }
  pre {
    white-space: pre-wrap; word-break: break-word;
    background: #eee; padding: 10px; border-radius: 6px;
    font-family: monospace;
    user-select: all;
    cursor: default;
  }
  #premiumAnalysis {
    background: #fff0f0; color: #888; cursor: pointer;
  }
  #totalScoreText {
    text-align: center; font-weight: bold; margin: 10px 0;
    font-size: 20px;
  }
  #progressText {
    font-size: 14px; color: #555; text-align: center; margin-top: 5px;
  }
</style>
</head>
<body>

<h2>心理测评系统示范</h2>

<div id="sectionUserInfo">
  <label for="inputAge">年龄</label>
  <input type="number" id="inputAge" value="25" min="0" />
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
  <button id="btnPrev" disabled>上一题</button>
  <button id="btnNext" disabled>下一题</button>
  <div id="progressText"></div>
</div>

<div id="sectionResult" style="display:none;">
  <div id="totalScoreText"></div>
  <h3>基础解析（免费）</h3>
  <pre id="basicAnalysis"></pre>
  <h3>深度解析（付费）</h3>
  <pre id="premiumAnalysis" title="点击生成深度解析">点击生成深度解析</pre>
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
    { id: "Q3", text: "你喜欢尝试新鲜事物吗？", options: [
      { text: "完全不喜欢", tags: { novelty_seek: -2, openness: -1 } },
      { text: "不太喜欢", tags: { novelty_seek: -1, openness: 0 } },
      { text: "比较喜欢", tags: { novelty_seek: 1, openness: 1 } },
      { text: "非常喜欢", tags: { novelty_seek: 2, openness: 2 } }
    ]},
    { id: "Q4", text: "你通常是否按时完成任务？", options: [
      { text: "经常拖延", tags: { responsibility: -2 } },
      { text: "偶尔拖延", tags: { responsibility: -1 } },
      { text: "大部分时间按时", tags: { responsibility: 1 } },
      { text: "总是按时完成", tags: { responsibility: 2 } }
    ]},
    { id: "Q5", text: "你做决定时是否容易冲动？", options: [
      { text: "完全不会冲动", tags: { self_control: 2 } },
      { text: "不太冲动", tags: { self_control: 1 } },
      { text: "有时冲动", tags: { self_control: -1 } },
      { text: "非常冲动", tags: { self_control: -2 } }
    ]},
    { id: "Q6", text: "你是否喜欢探索未知？", options: [
      { text: "完全不喜欢", tags: { novelty_seek: -2 } },
      { text: "不太喜欢", tags: { novelty_seek: -1 } },
      { text: "比较喜欢", tags: { novelty_seek: 1 } },
      { text: "非常喜欢", tags: { novelty_seek: 2 } }
    ]},
    { id: "Q7", text: "你是否喜欢有条理地生活和工作？", options: [
      { text: "不喜欢", tags: { responsibility: -2, openness: -1 } },
      { text: "偶尔", tags: { responsibility: -1, openness: 0 } },
      { text: "经常", tags: { responsibility: 1, openness: 1 } },
      { text: "总是", tags: { responsibility: 2, openness: 2 } }
    ]},
    { id: "Q8", text: "你控制情绪的能力如何？", options: [
      { text: "很差", tags: { emotion_stability: -2, self_control: -2 } },
      { text: "一般", tags: { emotion_stability: -1, self_control: -1 } },
      { text: "较好", tags: { emotion_stability: 1, self_control: 1 } },
      { text: "非常好", tags: { emotion_stability: 2, self_control: 2 } }
    ]},
    { id: "Q9", text: "你是否愿意接受新观点和改变？", options: [
      { text: "完全不愿意", tags: { openness: -2 } },
      { text: "不太愿意", tags: { openness: -1 } },
      { text: "比较愿意", tags: { openness: 1 } },
      { text: "非常愿意", tags: { openness: 2 } }
    ]},
    { id: "Q10", text: "你是否喜欢团队合作？", options: [
      { text: "完全不喜欢", tags: { extraversion: -2, responsibility: -1 } },
      { text: "不太喜欢", tags: { extraversion: -1, responsibility: 0 } },
      { text: "比较喜欢", tags: { extraversion: 1, responsibility: 1 } },
      { text: "非常喜欢", tags: { extraversion: 2, responsibility: 2 } }
    ]},
    { id: "Q11", text: "你面对失败时的态度是？", options: [
      { text: "很沮丧，难以振作", tags: { emotion_stability: -2 } },
      { text: "有些消极", tags: { emotion_stability: -1 } },
      { text: "能较快调整", tags: { emotion_stability: 1 } },
      { text: "积极乐观", tags: { emotion_stability: 2 } }
    ]},
    { id: "Q12", text: "你是否喜欢规划未来？", options: [
      { text: "完全不喜欢", tags: { responsibility: -2, self_control: -1 } },
      { text: "不太喜欢", tags: { responsibility: -1, self_control: 0 } },
      { text: "比较喜欢", tags: { responsibility: 1, self_control: 1 } },
      { text: "非常喜欢", tags: { responsibility: 2, self_control: 2 } }
    ]}
  ];

  let currentIndex = 0;
  let selectedOptions = new Array(questionPool.length).fill(null);
  let tagScores = {};
  let age = 25;
  let gender = "male";
  let userHasPaid = false; // 模拟付费状态

  const sectionUserInfo = document.getElementById("sectionUserInfo");
  const sectionQuiz = document.getElementById("sectionQuiz");
  const sectionResult = document.getElementById("sectionResult");
  const qTextEl = document.getElementById("questionText");
  const optionsEl = document.getElementById("optionsContainer");
  const btnNext = document.getElementById("btnNext");
  const btnPrev = document.getElementById("btnPrev");
  const progressText = document.getElementById("progressText");
  const basicAnalysisEl = document.getElementById("basicAnalysis");
  const premiumAnalysisEl = document.getElementById("premiumAnalysis");
  const btnStart = document.getElementById("btnStart");
  const btnRestart = document.getElementById("btnRestart");
  const inputAge = document.getElementById("inputAge");
  const selectGender = document.getElementById("selectGender");
  const totalScoreText = document.getElementById("totalScoreText");

  function updateBackgroundColor() {
    document.body.style.backgroundColor = "#fff";
  }

  function selectOption(idx) {
    selectedOptions[currentIndex] = idx;
    Array.from(optionsEl.children).forEach((el, i) => {
      el.classList.toggle("selected", i === idx);
    });
    btnNext.disabled = false;
  }

  function renderQuestion(index) {
    currentIndex = index;
    const q = questionPool[index];
    qTextEl.textContent = q.text;
    optionsEl.innerHTML = "";
    btnNext.disabled = selectedOptions[index] === null;
    btnPrev.disabled = index === 0;
    for (let i = 0; i < q.options.length; i++) {
      const opt = document.createElement("div");
      opt.className = "option";
      opt.textContent = q.options[i].text;
      opt.onclick = () => selectOption(i);
      if (selectedOptions[index] === i) opt.classList.add("selected");
      optionsEl.appendChild(opt);
    }
    progressText.textContent = `第 ${index + 1} / ${questionPool.length} 题`;
    updateBackgroundColor();
  }

  function calculateScores() {
    tagScores = {};
    selectedOptions.forEach((optIdx, qIdx) => {
      if (optIdx == null) return;
      const tags = questionPool[qIdx].options[optIdx].tags;
      for (const k in tags) tagScores[k] = (tagScores[k] || 0) + tags[k];
    });
  }

  function computeTotalScore(scores) {
    const vals = geneDimensions.map(d => scores[d.key] || 0);
    const avgRaw = vals.reduce((a,b)=>a+b,0) / vals.length;
    const norm = (avgRaw + 24) / 48;
    return (norm * 100).toFixed(1);
  }

  function generateBasicAnalysis(scores) {
    const parts = [];
    const { extraversion=0, emotion_stability=0, novelty_seek=0, responsibility=0, self_control=0, openness=0 } = scores;

    if (extraversion >= 4) parts.push("你性格外向，善于交流，喜欢社交。");
    else if (extraversion <= -4) parts.push("你较为内向，喜欢独处。");
    else parts.push("你的外向性适中，灵活适应社交环境。");

    if (emotion_stability >= 4) parts.push("情绪稳定，能有效应对压力。");
    else if (emotion_stability <= -4) parts.push("情绪波动较大，建议学习情绪调节。");
    else parts.push("情绪较为平稳，能较好管理压力。");

    if (novelty_seek >= 4) parts.push("喜欢探索新事物，富有好奇心。");
    else if (novelty_seek <= -4) parts.push("偏好稳定环境，较少寻求新体验。");
    else parts.push("对新事物持开放态度。");

    if (responsibility >= 4) parts.push("责任感强，做事有条理。");
    else if (responsibility <= -4) parts.push("有时拖延，建议加强时间管理。");
    else parts.push("责任感适中，能平衡自由与约束。");

    if (self_control >= 4) parts.push("自控力强，善于控制冲动。");
    else if (self_control <= -4) parts.push("较易冲动，建议培养耐心。");
    else parts.push("自控力适中，较理智。");

    if (openness >= 4) parts.push("思想开放，乐于接受新观点。");
    else if (openness <= -4) parts.push("较为保守，喜欢传统。");
    else parts.push("开放性适中，兼顾传统与创新。");

    return parts.join("\n");
  }

  async function requestPremiumAnalysis() {
    premiumAnalysisEl.textContent = "生成中，请稍候...";
    try {
      const resp = await fetch('/api/generateAnalysis', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ scores: tagScores, age, gender })
      });
      const data = await resp.json();
      premiumAnalysisEl.textContent = data.analysisText;
    } catch {
      premiumAnalysisEl.textContent = "生成失败，请稍后重试。";
    }
  }

  premiumAnalysisEl.onclick = () => {
    if (!userHasPaid) {
      alert("请先购买深度解析服务。");
      return;
    }
    requestPremiumAnalysis();
  };

  btnStart.onclick = () => {
    age = parseInt(inputAge.value);
    if (isNaN(age) || age < 0) {
      alert("请输入有效年龄");
      return;
    }
    gender = selectGender.value;
    selectedOptions = new Array(questionPool.length).fill(null);
    tagScores = {};
    currentIndex = 0;
    sectionUserInfo.style.display = "none";
    sectionQuiz.style.display = "block";
    sectionResult.style.display = "none";
    renderQuestion(currentIndex);
  };

  btnRestart.onclick = () => {
    sectionUserInfo.style.display = "block";
    sectionQuiz.style.display = "none";
    sectionResult.style.display = "none";
    userHasPaid = false; // 重置付费状态，实际可根据登录状态
  };

  btnNext.onclick = () => {
    if (selectedOptions[currentIndex] == null) {
      alert("请选择一个选项");
      return;
    }
    if (currentIndex < questionPool.length - 1) {
      renderQuestion(currentIndex + 1);
    } else {
      calculateScores();
      renderResult();
    }
  };

  btnPrev.onclick = () => {
    if (currentIndex > 0) renderQuestion(currentIndex - 1);
  };

  function renderResult() {
    sectionQuiz.style.display = "none";
    sectionResult.style.display = "block";

    const total = computeTotalScore(tagScores);
    totalScoreText.textContent = `综合因子得分：${total}分`;
    totalScoreText.style.color = `rgb(${240 - 180 * total / 100},${80 + 40 * total / 100},${80 + 160 * total / 100})`;

    basicAnalysisEl.textContent = generateBasicAnalysis(tagScores);
    premiumAnalysisEl.textContent = "点击生成深度解析";
  }

  function renderQuestion(index) {
    currentIndex = index;
    const q = questionPool[index];
    qTextEl.textContent = q.text;
    optionsEl.innerHTML = "";
    btnNext.disabled = selectedOptions[index] === null;
    btnPrev.disabled = index === 0;
    for (let i = 0; i < q.options.length; i++) {
      const opt = document.createElement("div");
      opt.className = "option";
      opt.textContent = q.options[i].text;
      opt.onclick = () => selectOption(i);
      if (selectedOptions[index] === i) opt.classList.add("selected");
      optionsEl.appendChild(opt);
    }
    progressText.textContent = `第 ${index + 1} / ${questionPool.length} 题`;
    updateBackgroundColor();
  }
</script>

</body>
</html>
