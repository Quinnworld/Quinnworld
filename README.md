<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>个性化动态心理测评系统</title>
<style>
  body { max-width: 600px; margin: 20px auto; font-family: "微软雅黑", sans-serif; background: #fff; padding: 20px; color:#000; }
  h2,h3 { text-align:center; }
  label, select, input[type=number] { width: 100%; margin:10px 0 15px; font-size:16px; }
  select, input[type=number] { padding:8px; border-radius:6px; border:1px solid #ccc; }
  button { width:48%; padding:12px; font-size:16px; border:none; border-radius:6px; cursor:pointer; color:#fff; background:#4338ca; margin:5px 1%; box-sizing:border-box; }
  button:disabled { background:#a5b4fc; cursor:not-allowed; }
  .option { background:#fff; border:1px solid #bbb; border-radius:6px; padding:10px; margin:6px 0; cursor:pointer; user-select:none; color:#000; transition: background-color 0.3s, color 0.3s; }
  .option.selected { background:#4f46e5; color:#fff; border-color:#4338ca; }
  #radarChart { width: 400px; height: 400px; margin: 20px auto; }
  #paySection { text-align:center; margin: 15px 0; }
  #paySection img { width: 200px; border:1px solid #ccc; border-radius:8px; }
  pre { white-space: pre-wrap; word-break: break-word; background:#eee; padding:10px; border-radius:6px; font-family: monospace; user-select: all; cursor: default; }
</style>
</head>
<body>

<h2>个性化动态心理测评系统</h2>

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
  <div id="progressText" style="text-align:center; margin-top:5px; color:#555;"></div>
</div>

<div id="sectionResult" style="display:none;">
  <div id="totalScoreText" style="text-align:center; font-weight:bold; font-size:20px; margin-bottom:10px;"></div>
  <div id="radarChart"></div>
  <h3>基础解析（免费）</h3>
  <pre id="basicAnalysis"></pre>
  <h3>深度解析（付费）</h3>
  <pre id="premiumAnalysis" title="点击生成深度解析" style="background:#fff0f0; color:#888; cursor:pointer;">点击生成深度解析</pre>

  <div id="paySection" style="display:none;">
    <p>请扫码付款解锁深度解析</p>
    <img src="https://pplx-res.cloudinary.com/image/upload/v1750588444/user_uploads/19241329/9b7936f2-1eeb-4500-99b9-9ddc57f69890/mm_facetoface_collect_qrcode_1750513571858.jpg" alt="收款码" />
    <br/>
    <button id="btnPaidConfirm" style="background:#10b981; margin-top:10px;">我已付款，解锁解析</button>
  </div>

  <button id="btnRestart" style="margin-top:20px;">重新开始</button>
</div>

<script src="https://cdn.jsdelivr.net/npm/echarts/dist/echarts.min.js"></script>
<script>
  // 洗牌算法
  function shuffle(arr) {
    for (let i = arr.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [arr[i], arr[j]] = [arr[j], arr[i]];
    }
    return arr;
  }

  // 题库
  const fullQuestionPool = [
    { id: "Q1", text: "你在社交场合的表现如何？", options: [
      { text: "非常害羞", tags: { extraversion: -2 } },
      { text: "有点紧张", tags: { extraversion: -1 } },
      { text: "比较自然", tags: { extraversion: 1 } },
      { text: "非常外向", tags: { extraversion: 2 } }
    ]},
    { id: "Q2", text: "你面对压力时的情绪反应？", options: [
      { text: "非常焦虑", tags: { emotion_stability: -2 } },
      { text: "有些紧张", tags: { emotion_stability: -1 } },
      { text: "情绪稳定", tags: { emotion_stability: 1 } },
      { text: "非常冷静", tags: { emotion_stability: 2 } }
    ]},
    { id: "Q3", text: "你喜欢尝试新鲜事物吗？", options: [
      { text: "不喜欢", tags: { novelty_seek: -2 } },
      { text: "偶尔尝试", tags: { novelty_seek: 0 } },
      { text: "喜欢", tags: { novelty_seek: 1 } },
      { text: "非常喜欢", tags: { novelty_seek: 2 } }
    ]},
    { id: "Q4", text: "你是否按时完成任务？", options: [
      { text: "经常拖延", tags: { responsibility: -2 } },
      { text: "偶尔拖延", tags: { responsibility: -1 } },
      { text: "大部分时间按时", tags: { responsibility: 1 } },
      { text: "总是按时", tags: { responsibility: 2 } }
    ]},
    { id: "Q5", text: "你控制冲动的能力如何？", options: [
      { text: "很差", tags: { self_control: -2 } },
      { text: "一般", tags: { self_control: -1 } },
      { text: "较好", tags: { self_control: 1 } },
      { text: "非常好", tags: { self_control: 2 } }
    ]}
  ];

  const geneDimensions = [
    { key: "extraversion", label: "外向性" },
    { key: "emotion_stability", label: "情绪稳定性" },
    { key: "novelty_seek", label: "新奇寻求" },
    { key: "responsibility", label: "责任感" },
    { key: "self_control", label: "自控力" }
  ];

  let currentIndex = 0;
  let selectedOptions = [];
  let tagScores = {};
  let userHasPaid = false;

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
  const paySection = document.getElementById("paySection");
  const btnPaidConfirm = document.getElementById("btnPaidConfirm");

  let questionPool = [];
  let shuffledOptionsList = [];

  function selectOption(idx) {
    selectedOptions[currentIndex] = idx;
    Array.from(optionsEl.children).forEach((el,i) => {
      el.classList.toggle("selected", i === idx);
    });
    btnNext.disabled = false;
  }

  function renderQuestion(index) {
    currentIndex = index;
    const q = questionPool[index];
    qTextEl.textContent = q.text;
    optionsEl.innerHTML = "";
    btnNext.disabled = selectedOptions[index] == null;
    btnPrev.disabled = index === 0;

    // 选项顺序动态乱序
    let options = shuffledOptionsList[index];
    if (!options) {
      options = shuffle(q.options.map((opt, i) => ({ ...opt, origIdx: i })));
      shuffledOptionsList[index] = options;
    }
    for(let i=0;i<options.length;i++) {
      const opt = document.createElement("div");
      opt.className = "option";
      opt.textContent = options[i].text;
      opt.onclick = () => selectOption(i);
      if(selectedOptions[index] === i) opt.classList.add("selected");
      optionsEl.appendChild(opt);
    }
    progressText.textContent = `第 ${index+1} / ${questionPool.length} 题`;
  }

  function calculateScores() {
    tagScores = {};
    selectedOptions.forEach((optIdx, qIdx) => {
      if(optIdx == null) return;
      const origIdx = shuffledOptionsList[qIdx][optIdx].origIdx;
      const tags = questionPool[qIdx].options[origIdx].tags;
      for(const k in tags) tagScores[k] = (tagScores[k]||0) + tags[k];
    });
  }

  function computeTotalScore(scores) {
    const vals = geneDimensions.map(d => scores[d.key] || 0);
    const avgRaw = vals.reduce((a,b)=>a+b,0) / vals.length;
    const norm = (avgRaw + 10) / 20;
    return (norm * 100).toFixed(1);
  }

  function generateBasicAnalysisWithAdvice(scores) {
    const advices = [];
    const strengths = [];
    const { extraversion=0, emotion_stability=0, novelty_seek=0, responsibility=0, self_control=0 } = scores;

    if(extraversion >= 3) strengths.push("你外向开朗，善于沟通和社交。");
    else if(extraversion <= -3) strengths.push("你内敛沉稳，善于倾听和深思。");

    if(emotion_stability >= 3) strengths.push("你情绪稳定，能有效管理压力。");
    else if(emotion_stability <= -3) strengths.push("你情感细腻，富有同理心。");

    if(novelty_seek >= 3) strengths.push("你富有好奇心和创新精神。");
    else if(novelty_seek <= -3) strengths.push("你喜欢稳定和安全，注重细节。");

    if(responsibility >= 3) strengths.push("你责任感强，做事认真负责。");
    else if(responsibility <= -3) strengths.push("你灵活适应，具备创造性思维。");

    if(self_control >= 3) strengths.push("你自控力强，善于时间和情绪管理。");
    else if(self_control <= -3) strengths.push("你反应迅速，适合快速决策。");

    if(strengths.length === 0) strengths.push("你的性格均衡，具备多方面潜力。");

    advices.push("【你的优势】");
    advices.push(strengths.join("\n"));
    advices.push("\n【发展建议】");
    advices.push("1. 发挥优势，选择适合的生活和工作方式。");
    advices.push("2. 针对弱项，尝试提升相关能力。");
    advices.push("3. 保持开放心态，积极接受挑战。");
    advices.push("4. 注重身心健康，合理安排生活。");

    return advices.join("\n");
  }

  function drawRadarChart(scores) {
    const chartDom = document.getElementById('radarChart');
    const myChart = echarts.init(chartDom);
    const indicators = geneDimensions.map(dim => ({ name: dim.label, max: 10 }));
    const dataValues = geneDimensions.map(dim => (scores[dim.key] || 0) + 5);
    const option = {
      tooltip: {},
      radar: {
        indicator: indicators,
        shape: 'circle',
        splitNumber: 5,
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
    const age = parseInt(document.getElementById("inputAge").value);
    if(isNaN(age) || age < 0) { alert("请输入有效年龄"); return; }
    selectedOptions = new Array();
    tagScores = {};
    questionPool = shuffle(fullQuestionPool.map(q => ({...q})));
    shuffledOptionsList = [];
    currentIndex = 0;
    sectionUserInfo.style.display = "none";
    sectionQuiz.style.display = "block";
    sectionResult.style.display = "none";
    renderQuestion(currentIndex);
  };

  btnNext.onclick = () => {
    if(selectedOptions[currentIndex] == null) {
      alert("请选择一个选项");
      return;
    }
    if(currentIndex < questionPool.length - 1) {
      renderQuestion(currentIndex + 1);
    } else {
      calculateScores();
      renderResult();
    }
  };

  btnPrev.onclick = () => {
    if(currentIndex > 0) renderQuestion(currentIndex - 1);
  };

  function renderResult() {
    sectionQuiz.style.display = "none";
    sectionResult.style.display = "block";
    userHasPaid = false;
    paySection.style.display = "none";

    const total = computeTotalScore(tagScores);
    totalScoreText.textContent = `综合因子得分：${total}分`;
    totalScoreText.style.color = `rgb(${240 - 180 * total / 100},${80 + 40 * total / 100},${80 + 160 * total / 100})`;

    drawRadarChart(tagScores);
    basicAnalysisEl.textContent = generateBasicAnalysisWithAdvice(tagScores);
    premiumAnalysisEl.textContent = "点击生成深度解析";
  }

  premiumAnalysisEl.onclick = () => {
    if(userHasPaid) {
      premiumAnalysisEl.textContent = "这是基于心理学的深度个性解析报告，内容更丰富、更专业。";
      paySection.style.display = "none";
    } else {
      paySection.style.display = "block";
      premiumAnalysisEl.textContent = "请扫码付款解锁深度解析";
    }
  };

  btnPaidConfirm.onclick = () => {
    userHasPaid = true;
    alert("付款确认成功，深度解析已解锁！");
    premiumAnalysisEl.textContent = "这是基于心理学的深度个性解析报告，内容更丰富、更专业。";
    paySection.style.display = "none";
  };

  btnRestart.onclick = () => {
    sectionUserInfo.style.display = "block";
    sectionQuiz.style.display = "none";
    sectionResult.style.display = "none";
    userHasPaid = false;
    paySection.style.display = "none";
  };
</script>

</body>
</html>
