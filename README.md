<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>因•趣问答完整版（2025心理科学版）</title>
<style>
  body {
    max-width: 600px; margin: 20px auto; font-family: "微软雅黑", sans-serif;
    background: #fff; padding: 20px; color:#000;
    transition: background-color 0.8s ease;
  }
  h2, h3 {
    text-align: center;
  }
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
  #promptOutput, #basicAnalysis, #premiumAnalysis {
    white-space: pre-wrap; word-break: break-word;
    background: #eee; padding: 10px; border-radius: 6px;
    font-family: monospace;
    user-select: all; /* 方便复制 */
    cursor: default;
  }
  #premiumAnalysis {
    background: #fff0f0; color: #888; cursor: pointer;
  }
  #totalScoreText {
    text-align: center; font-weight: bold; margin: 10px 0;
    font-size: 20px;
    transition: color 0.5s ease;
  }
  canvas {
    display: block;
    margin: 0 auto 20px;
    max-width: 100%;
    background: white;
  }
  #progressText {
    font-size: 14px; color: #555; text-align: center; margin-top: 5px;
  }
  #btnsContainer {
    text-align: center;
    margin-top: 10px;
  }
</style>
</head>
<body>

<h2>趣问答 & 因得分完整版（2025心理科学版）</h2>

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
  <div id="btnsContainer">
    <button id="btnPrev" disabled>上一题</button>
    <button id="btnNext" disabled>下一题</button>
  </div>
  <div id="progressText"></div>
</div>

<div id="sectionResult" style="display:none;">
  <h3>雷达图因子维度评分</h3>
  <canvas id="radarChart" width="400" height="400"></canvas>
  <div id="totalScoreText"></div>

  <h3>初步版个性解析（免费）</h3>
  <pre id="basicAnalysis"></pre>

  <h3>深度版个性解析（付费解锁）</h3>
  <pre id="premiumAnalysis" title="点击购买深度解析服务">点击购买深度解析服务</pre>

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

  let tagScores = {};
  let currentIndex = 0;
  let selectedOptions = [];
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
  const basicAnalysisEl = document.getElementById("basicAnalysis");
  const premiumAnalysisEl = document.getElementById("premiumAnalysis");
  const btnStart = document.getElementById("btnStart");
  const btnRestart = document.getElementById("btnRestart");
  const inputAge = document.getElementById("inputAge");
  const selectGender = document.getElementById("selectGender");
  const totalScoreText = document.getElementById("totalScoreText");
  const radarCanvas = document.getElementById("radarChart");
  const ctx = radarCanvas.getContext("2d");

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
    selectedOptions[index] = selectedOptions[index] ?? null;
    btnNext.disabled = selectedOptions[index] === null;
    btnPrev.disabled = index === 0;
    for (let i = 0; i < q.options.length; i++) {
      const opt = document.createElement("div");
      opt.className = "option";
      opt.textContent = q.options[i].text;
      opt.onclick = () => selectOption(i);
      if (selectedOptions[index] === i) {
        opt.classList.add("selected");
      }
      optionsEl.appendChild(opt);
    }
    progressText.textContent = `第 ${index + 1} / ${questionPool.length} 题`;
    updateBackgroundColor();
  }

  function calculateScores() {
    tagScores = {};
    selectedOptions.forEach((optIdx, qIdx) => {
      if (optIdx === null || optIdx === undefined) return;
      const tags = questionPool[qIdx].options[optIdx].tags;
      for (const k in tags) {
        tagScores[k] = (tagScores[k] || 0) + tags[k];
      }
    });
  }

  function drawRadarChart(scores) {
    const w = radarCanvas.width;
    const h = radarCanvas.height;
    ctx.clearRect(0, 0, w, h);
    const cx = w / 2;
    const cy = h / 2;
    const radius = Math.min(cx, cy) * 0.75;
    const dimCount = geneDimensions.length;
    const angleStep = (2 * Math.PI) / dimCount;

    ctx.lineWidth = 1;
    ctx.strokeStyle = "rgba(0,0,0,0.2)";
    for (let level = 1; level <= 5; level++) {
      ctx.beginPath();
      for (let i = 0; i < dimCount; i++) {
        const r = (radius * level) / 5;
        const x = cx + r * Math.sin(i * angleStep);
        const y = cy - r * Math.cos(i * angleStep);
        if (i === 0) ctx.moveTo(x, y);
        else ctx.lineTo(x, y);
      }
      ctx.closePath();
      ctx.stroke();
    }

    ctx.fillStyle = "#000";
    ctx.font = "14px 微软雅黑";
    ctx.textAlign = "center";
    ctx.textBaseline = "middle";
    for (let i = 0; i < dimCount; i++) {
      const x = cx + radius * Math.sin(i * angleStep);
      const y = cy - radius * Math.cos(i * angleStep);
      ctx.beginPath();
      ctx.moveTo(cx, cy);
      ctx.lineTo(x, y);
      ctx.stroke();

      const label = geneDimensions[i].label;
      const labelX = cx + (radius + 25) * Math.sin(i * angleStep);
      const labelY = cy - (radius + 25) * Math.cos(i * angleStep);
      ctx.fillText(label, labelX, labelY);
    }

    // 12题最大绝对值24
    const normalizedScores = geneDimensions.map(dim => {
      const raw = scores[dim.key] || 0;
      let norm = (raw + 24) / 48;
      norm = Math.min(1, Math.max(0, norm));
      return norm;
    });

    ctx.beginPath();
    for (let i = 0; i < dimCount; i++) {
      const r = normalizedScores[i] * radius;
      const x = cx + r * Math.sin(i * angleStep);
      const y = cy - r * Math.cos(i * angleStep);
      if (i === 0) ctx.moveTo(x, y);
      else ctx.lineTo(x, y);
    }
    ctx.closePath();
    ctx.fillStyle = "rgba(100, 80, 200, 0.4)";
    ctx.fill();
    ctx.strokeStyle = "rgba(80, 60, 180, 0.8)";
    ctx.lineWidth = 2;
    ctx.stroke();

    for (let i = 0; i < dimCount; i++) {
      const r = normalizedScores[i] * radius;
      const x = cx + r * Math.sin(i * angleStep);
      const y = cy - r * Math.cos(i * angleStep);
      ctx.beginPath();
      ctx.arc(x, y, 6, 0, Math.PI * 2);
      ctx.fillStyle = "rgb(100, 60, 220)";
      ctx.fill();
      ctx.strokeStyle = "#444";
      ctx.stroke();
    }
  }

  function computeTotalScore(scores) {
    const vals = geneDimensions.map(d => scores[d.key] || 0);
    const avgRaw = vals.reduce((a,b)=>a+b,0) / vals.length;
    const norm = (avgRaw + 24) / 48;
    return (norm * 100).toFixed(1);
  }

  // 基于2025心理科学进展的初步版个性解析
  function generateBasicAnalysis(scores) {
    let parts = [];
    for (const dim of geneDimensions) {
      const v = scores[dim.key] || 0;
      if (v >= 4) parts.push(`${dim.label}较高`);
      else if (v <= -4) parts.push(`${dim.label}较低`);
    }
    if (parts.length === 0) return "你的个性较为均衡，性格稳定，适应力强。";
    return parts.join("，") + "。";
  }

  // 结合最新心理科学研究成果的深度版个性解析示范
  function generatePremiumAnalysis(scores) {
    return `深度解析报告（基于2025心理科学进展）：

- 外向性：${scores.extraversion || 0}。研究表明，外向性高的人更善于建立人际关系，能有效利用社会支持缓解压力。
- 情绪稳定性：${scores.emotion_stability || 0}。情绪稳定性关联认知偏差矫正能力，影响个体面对逆境时的心理弹性。
- 新奇寻求：${scores.novelty_seek || 0}。新奇寻求高者倾向于探索创新，适应快速变化的环境，但也需注意冲动控制。
- 责任感：${scores.responsibility || 0}。责任感强的人在任务完成和自我管理上表现突出，有利于职业发展和生活规划。
- 自控力：${scores.self_control || 0}。自控力是心理健康的重要指标，关联成瘾行为防治和情绪调节能力。
- 开放性：${scores.openness || 0}。开放性高的人更容易接受新观点，促进学习和创造力的发展。

本报告基于最新心理学研究成果，如认知偏差矫正训练、情绪调节机制和行为成瘾防治策略，帮助你更深入理解自身个性特点。欲获取完整专业报告，请购买深度解析服务。`;
  }

  function setScoreTextColor(total) {
    const t = total / 100;
    const r = 240 + (60 - 240) * t;
    const g = 80 + (120 - 80) * t;
    const b = 80 + (240 - 80) * t;
    return `rgb(${r.toFixed(0)},${g.toFixed(0)},${b.toFixed(0)})`;
  }

  btnNext.onclick = () => {
    if (selectedOptions[currentIndex] == null) {
      alert("请先选择一个选项！");
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
    if (currentIndex > 0) {
      renderQuestion(currentIndex - 1);
    }
  };

  btnStart.onclick = () => {
    age = parseInt(inputAge.value);
    if (isNaN(age) || age < 0) {
      alert("请输入有效的年龄！");
      return;
    }
    gender = selectGender.value;
    selectedOptions = new Array(questionPool.length).fill(null);
    tagScores = {};
    currentIndex = 0;
    sectionUserInfo.style.display = "none";
    sectionQuiz.style.display = "block";
    sectionResult.style.display = "none";
    document.body.style.backgroundColor = "#fff";
    renderQuestion(currentIndex);
  };

  btnRestart.onclick = () => {
    sectionUserInfo.style.display = "block";
    sectionQuiz.style.display = "none";
    sectionResult.style.display = "none";
    document.body.style.backgroundColor = "#fff";
  };

  function renderResult() {
    sectionQuiz.style.display = "none";
    sectionResult.style.display = "block";

    drawRadarChart(tagScores);

    const total = computeTotalScore(tagScores);
    totalScoreText.textContent = `综合因子得分：${total}分`;
    totalScoreText.style.color = setScoreTextColor(total);

    basicAnalysisEl.textContent = generateBasicAnalysis(tagScores);

    premiumAnalysisEl.textContent = "点击购买深度解析服务";
    premiumAnalysisEl.style.color = "#888";
    premiumAnalysisEl.style.cursor = "pointer";
    premiumAnalysisEl.onclick = () => {
      alert("付费功能暂未开通，敬请期待！");
      // 这里可接入支付或跳转购买流程
      // 例如：window.location.href = "支付页面URL";
    };
  }

</script>

</body>
</html>
