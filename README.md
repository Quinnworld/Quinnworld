<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>心理测评系统示范（优点发掘版）</title>
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
  #radarChart {
    width: 400px; height: 400px; margin: 20px auto;
  }
</style>
</head>
<body>

<h2>心理测评系统示范（优点发掘版）</h2>

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
  <div id="radarChart"></div>
  <h3>基础解析（免费）</h3>
  <pre id="basicAnalysis"></pre>
  <h3>深度解析（付费）</h3>
  <pre id="premiumAnalysis" title="点击生成深度解析">点击生成深度解析</pre>

  <!-- 分享和邀请按钮 -->
  <div style="text-align:center; margin-top:15px;">
    <button id="btnShare" style="background:#10b981; margin-right:10px;">生成趣味分享卡</button>
    <button id="btnInvite" style="background:#3b82f6;">邀请好友解锁深度解析</button>
  </div>
  <div id="shareText" style="margin-top:10px; padding:10px; background:#f0fdf4; border:1px solid #34d399; border-radius:6px; display:none; user-select: all; cursor: pointer;"></div>

  <button id="btnRestart">重新开始</button>
</div>

<!-- 引入 ECharts -->
<script src="https://cdn.jsdelivr.net/npm/echarts/dist/echarts.min.js"></script>

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
  const totalScoreText = document.getElementById("totalScoreText");
  const btnShare = document.getElementById("btnShare");
  const btnInvite = document.getElementById("btnInvite");
  const shareTextDiv = document.getElementById("shareText");

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

  // 新版：发掘优点并给出综合发展建议
  function generateBasicAnalysisWithAdvice(scores) {
    const advices = [];
    const strengths = [];

    const { extraversion=0, emotion_stability=0, novelty_seek=0, responsibility=0, self_control=0, openness=0 } = scores;

    // 发掘优点
    if (extraversion >= 6) strengths.push("你外向开朗，善于沟通和社交，能够积极影响周围人。");
    else if (extraversion <= -6) strengths.push("你内敛沉稳，善于倾听和深度思考，具有独特的观察力。");

    if (emotion_stability >= 6) strengths.push("你情绪稳定，具备良好的压力管理能力，能够冷静应对挑战。");
    else if (emotion_stability <= -6) strengths.push("你情感细腻，富有同理心，能够敏锐感知他人情绪。");

    if (novelty_seek >= 6) strengths.push("你富有好奇心和创新精神，乐于尝试新事物和突破自我。");
    else if (novelty_seek <= -6) strengths.push("你喜欢稳定和安全，注重细节和规划，适合踏实稳健的工作。");

    if (responsibility >= 6) strengths.push("你责任感强，做事认真负责，是团队的中坚力量。");
    else if (responsibility <= -6) strengths.push("你灵活适应，能够在变化中寻找机会，具备创造性思维。");

    if (self_control >= 6) strengths.push("你自控力强，能够坚持长期目标，善于管理时间和情绪。");
    else if (self_control <= -6) strengths.push("你直觉敏锐，反应迅速，适合快速决策和应对突发状况。");

    if (openness >= 6) strengths.push("你思想开放，乐于学习和接受新观点，具备跨界整合能力。");
    else if (openness <= -6) strengths.push("你专注深耕，注重专业知识积累，能够成为某一领域的专家。");

    if (strengths.length === 0) {
      strengths.push("你的性格特点均衡，具备多方面的潜力和适应力。");
    }

    advices.push("【你的优势】");
    advices.push(strengths.join("\n"));

    // 综合发展建议
    advices.push("\n【发展建议】");
    advices.push("1. 充分发挥你的优势，选择适合自己性格特点的工作和生活方式。");
    advices.push("2. 针对较弱的维度，尝试适度锻炼和提升，例如学习情绪管理或增强自控力。");
    advices.push("3. 保持开放心态，积极接受反馈和新挑战，促进个人成长。");
    advices.push("4. 注重身心健康，合理安排工作与休息，保持良好状态。");

    return advices.join("\n");
  }

  // 绘制雷达图
  function drawRadarChart(scores) {
    const chartDom = document.getElementById('radarChart');
    const myChart = echarts.init(chartDom);

    const indicators = geneDimensions.map(dim => ({ name: dim.label, max: 48 }));

    const dataValues = geneDimensions.map(dim => {
      let val = scores[dim.key] || 0;
      return val + 24; // 转换到0~48区间
    });

    const option = {
      tooltip: {},
      radar: {
        indicator: indicators,
        shape: 'circle',
        splitNumber: 6,
        axisName: { color: '#000', fontSize: 14 }
      },
      series: [{
        name: '性格因子得分',
        type: 'radar',
        data: [{
          value: dataValues,
          name: '得分',
          areaStyle: { color: 'rgba(79, 70, 229, 0.4)' },
          lineStyle: { color: '#4f46e5' },
          symbolSize: 8,
          itemStyle: { color: '#4f46e5' }
        }]
      }]
    };

    myChart.setOption(option);
  }

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
    shareTextDiv.style.display = "none";
    renderQuestion(currentIndex);
  };

  btnRestart.onclick = () => {
    sectionUserInfo.style.display = "block";
    sectionQuiz.style.display = "none";
    sectionResult.style.display = "none";
    userHasPaid = false; // 重置付费状态
    shareTextDiv.style.display = "none";
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

    drawRadarChart(tagScores);

    basicAnalysisEl.textContent = generateBasicAnalysisWithAdvice(tagScores);

    premiumAnalysisEl.textContent = "点击生成深度解析";

    shareTextDiv.style.display = "none";
  }

  premiumAnalysisEl.onclick = () => {
    if (!userHasPaid) {
      alert("请先购买深度解析服务");
      return;
    }
    premiumAnalysisEl.textContent = "深度解析生成中，请稍候...";
    setTimeout(() => {
      premiumAnalysisEl.textContent = "这是基于AI生成的深度个性解析报告，内容更丰富、更专业。";
    }, 1500);
  };

  // 分享按钮事件
  btnShare.onclick = () => {
    const totalScore = computeTotalScore(tagScores);
    let funDesc = "";
    if (totalScore >= 80) funDesc = "你是个充满活力的阳光达人！🌟";
    else if (totalScore >= 60) funDesc = "你有着稳健的内心和积极的生活态度。😊";
    else if (totalScore >= 40) funDesc = "你是个思考细腻，值得信赖的朋友。🤔";
    else funDesc = "你有独特的个性魅力，值得更多了解！✨";

    const shareContent = `【心理测评结果】综合得分：${totalScore}分\n${funDesc}\n快来测测你的性格吧！👉 https://yourdomain.com/psych-test`;

    shareTextDiv.style.display = "block";
    shareTextDiv.textContent = shareContent;
    alert("分享卡已生成，长按复制内容分享到朋友圈或好友！");
  };

  // 邀请按钮事件
  btnInvite.onclick = () => {
    if (userHasPaid) {
      alert("您已解锁深度解析，无需邀请好友。");
      return;
    }
    const confirmInvite = confirm("邀请3位好友完成测评即可免费解锁深度解析，是否立即邀请？");
    if (confirmInvite) {
      alert("邀请链接已复制，请发送给好友！");
      userHasPaid = true; // 模拟邀请成功解锁
      premiumAnalysisEl.textContent = "点击生成深度解析";
    }
  };
</script>

</body>
</html>
