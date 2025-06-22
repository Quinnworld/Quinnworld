<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>基因科学动态问卷 + 动漫Prompt生成</title>
<style>
  body { max-width: 600px; margin: 20px auto; font-family: "微软雅黑", sans-serif; background: #f0f4f8; padding: 20px; }
  h2 { text-align: center; }
  label, select, input, button { width: 100%; margin: 6px 0; font-size: 16px; }
  .option { background: white; border: 1px solid #bbb; border-radius: 6px; padding: 10px; margin: 6px 0; cursor: pointer; user-select:none; }
  .option.selected { background: #4f46e5; color: white; border-color: #4338ca; }
  button { padding: 12px; border:none; border-radius:6px; background:#4338ca; color:#fff; cursor:pointer; }
  button:disabled { background:#a5b4fc; cursor:not-allowed; }
  #radarContainer { width: 100%; height: 350px; margin-top: 20px; }
  pre { background: #eee; padding: 10px; border-radius: 6px; white-space: pre-wrap; max-height: 140px; overflow-y: auto; }
</style>
</head>
<body>

<h2>基因科学动态问卷 + 动漫Prompt生成</h2>

<div id="sectionUserInfo">
  <label for="inputAge">年龄（18-80）</label>
  <input type="number" id="inputAge" min="18" max="80" value="25" />
  <label for="selectGender">性别</label>
  <select id="selectGender">
    <option value="male">男性</option>
    <option value="female">女性</option>
  </select>
  <button id="btnStart">开始答题</button>
</div>

<div id="sectionQuiz" style="display:none;">
  <div id="questionText" style="font-weight: bold; margin-bottom: 8px;"></div>
  <div id="optionsContainer"></div>
  <button id="btnNext" disabled>下一题</button>
  <div id="progressText" style="margin-top: 8px; color: #555;"></div>
</div>

<div id="sectionResult" style="display:none;">
  <div id="totalScore" style="font-weight: bold; font-size: 18px; text-align: center;"></div>
  <div id="radarContainer">
    <canvas id="radarChart"></canvas>
  </div>
  <h3>动漫风格Prompt：</h3>
  <pre id="promptOutput"></pre>
  <button id="btnRestart">重新开始</button>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
  // 题库（示例12题）
  const questions = [
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
        { text: "非常紧张和焦虑", tags: { emotion_stability: -2, anxiety: 2 } },
        { text: "有些不安", tags: { emotion_stability: -1, anxiety: 1 } },
        { text: "情绪稳定", tags: { emotion_stability: 1, anxiety: -1 } },
        { text: "非常冷静", tags: { emotion_stability: 2, anxiety: -2 } }
      ]
    },
    //... 省略其余题，和你之前题库一致，直接复制过去即可
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
        { text: "经常拖延", tags: { responsibility: -2, conscientiousness: -2 } },
        { text: "偶尔拖延", tags: { responsibility: -1, conscientiousness: -1 } },
        { text: "大部分时间按时", tags: { responsibility: 1, conscientiousness: 1 } },
        { text: "总是按时完成", tags: { responsibility: 2, conscientiousness: 2 } }
      ]
    },
    {
      id: "Q5", text: "你做决定时是否容易冲动？",
      options: [
        { text: "完全不会冲动", tags: { impulsivity: -2, self_control: 2 } },
        { text: "不太冲动", tags: { impulsivity: -1, self_control: 1 } },
        { text: "有时冲动", tags: { impulsivity: 1, self_control: -1 } },
        { text: "非常冲动", tags: { impulsivity: 2, self_control: -2 } }
      ]
    },
    {
      id: "Q6", text: "你是否容易感到焦虑？",
      options: [
        { text: "几乎不焦虑", tags: { anxiety: -2, emotion_stability: 2 } },
        { text: "偶尔焦虑", tags: { anxiety: -1, emotion_stability: 1 } },
        { text: "经常焦虑", tags: { anxiety: 1, emotion_stability: -1 } },
        { text: "非常焦虑", tags: { anxiety: 2, emotion_stability: -2 } }
      ]
    },
    {
      id: "Q7", text: "你喜欢艺术和创造性活动吗？",
      options: [
        { text: "完全不喜欢", tags: { openness: -2 } },
        { text: "不太喜欢", tags: { openness: -1 } },
        { text: "比较喜欢", tags: { openness: 1 } },
        { text: "非常喜欢", tags: { openness: 2 } }
      ]
    },
    {
      id: "Q8", text: "你待人是否友好与合作？",
      options: [
        { text: "不太友好", tags: { agreeableness: -2 } },
        { text: "有时冷淡", tags: { agreeableness: -1 } },
        { text: "比较友好", tags: { agreeableness: 1 } },
        { text: "非常友好", tags: { agreeableness: 2 } }
      ]
    },
    {
      id: "Q9", text: "你平时做事是否细致且有条理？",
      options: [
        { text: "非常粗心", tags: { conscientiousness: -2 } },
        { text: "有些粗心", tags: { conscientiousness: -1 } },
        { text: "比较细心", tags: { conscientiousness: 1 } },
        { text: "非常细心", tags: { conscientiousness: 2 } }
      ]
    },
    {
      id: "Q10", text: "你控制情绪的能力如何？",
      options: [
        { text: "很差", tags: { self_control: -2 } },
        { text: "一般", tags: { self_control: -1 } },
        { text: "较好", tags: { self_control: 1 } },
        { text: "非常好", tags: { self_control: 2 } }
      ]
    },
    {
      id: "Q11", text: "你是否经常拖延？",
      options: [
        { text: "经常拖延", tags: { responsibility: -2 } },
        { text: "偶尔拖延", tags: { responsibility: -1 } },
        { text: "较少拖延", tags: { responsibility: 1 } },
        { text: "几乎不拖延", tags: { responsibility: 2 } }
      ]
    },
    {
      id: "Q12", text: "你喜欢接受新观点和变化吗？",
      options: [
        { text: "完全不喜欢", tags: { openness: -2 } },
        { text: "不太喜欢", tags: { openness: -1 } },
        { text: "比较喜欢", tags: { openness: 1 } },
        { text: "非常喜欢", tags: { openness: 2 } }
      ]
    }
  ];

  const geneTags = [
    "extraversion", "emotion_stability", "novelty_seek", "responsibility",
    "impulsivity", "anxiety", "self_control", "openness", "agreeableness", "conscientiousness"
  ];

  const geneTagsCN = {
    extraversion: "外向性",
    emotion_stability: "情绪稳定性",
    novelty_seek: "新奇寻求",
    responsibility: "责任感",
    impulsivity: "冲动性",
    anxiety: "焦虑",
    self_control: "自我控制",
    openness: "开放性",
    agreeableness: "宜人性",
    conscientiousness: "尽责性"
  };

  let currentQuestionIndex = 0;
  let scores = {}; // 基因标签分数累计
  let age = 25;
  let gender = "male";

  const sectionUserInfo = document.getElementById("sectionUserInfo");
  const sectionQuiz = document.getElementById("sectionQuiz");
  const sectionResult = document.getElementById("sectionResult");
  const questionText = document.getElementById("questionText");
  const optionsContainer = document.getElementById("optionsContainer");
  const btnNext = document.getElementById("btnNext");
  const progressText = document.getElementById("progressText");
  const promptOutput = document.getElementById("promptOutput");
  const btnStart = document.getElementById("btnStart");
  const btnRestart = document.getElementById("btnRestart");
  const inputAge = document.getElementById("inputAge");
  const selectGender = document.getElementById("selectGender");
  const totalScoreEl = document.getElementById("totalScore");

  let selectedOption = null;
  let radarChart = null;

  function resetScores() {
    scores = {};
    geneTags.forEach(tag => scores[tag] = 0);
  }

  function renderQuestion() {
    selectedOption = null;
    btnNext.disabled = true;
    const q = questions[currentQuestionIndex];
    questionText.textContent = q.text;
    optionsContainer.innerHTML = "";
    q.options.forEach((opt, idx) => {
      const div = document.createElement("div");
      div.className = "option";
      div.textContent = opt.text;
      div.onclick = () => {
        selectedOption = idx;
        Array.from(optionsContainer.children).forEach((c, i) => {
          c.classList.toggle("selected", i === idx);
        });
        btnNext.disabled = false;
      };
      optionsContainer.appendChild(div);
    });
    progressText.textContent = `第 ${currentQuestionIndex + 1} / ${questions.length} 题`;
  }

  // 年龄性别调整(简单版)
  function adjustScoresByAgeGender(sc) {
    let adjusted = {...sc};
    if(age < 30){
      adjusted.novelty_seek = (adjusted.novelty_seek || 0) * 1.2;
      adjusted.openness = (adjusted.openness || 0) * 1.2;
    }
    if(gender === "female"){
      adjusted.anxiety = (adjusted.anxiety || 0) * 1.15;
    }
    return adjusted;
  }

  function generatePrompt(sc) {
    const s = adjustScoresByAgeGender(sc);
    let descs = [];

    if(s.extraversion > 1) descs.push("energetic and outgoing personality");
    else if(s.extraversion < -1) descs.push("calm and introverted demeanor");
    else descs.push("balanced social behavior");

    if(s.emotion_stability > 1) descs.push("emotionally stable and confident");
    else if(s.emotion_stability < -1) descs.push("sensitive and prone to stress");

    if(s.novelty_seek > 1) descs.push("open to new experiences and curious");
    else if(s.novelty_seek < -1) descs.push("prefers familiarity and routine");

    if(s.responsibility > 1) descs.push("responsible and reliable");
    else if(s.responsibility < -1) descs.push("sometimes careless or procrastinating");

    if(s.impulsivity > 1) descs.push("impulsive and spontaneous");
    else if(s.impulsivity < -1) descs.push("cautious and self-controlled");

    if(s.anxiety > 1) descs.push("often anxious or worried");
    else if(s.anxiety < -1) descs.push("calm and relaxed");

    if(s.self_control > 1) descs.push("excellent emotional regulation");
    else if(s.self_control < -1) descs.push("difficulty controlling emotions");

    if(s.openness > 1) descs.push("creative and imaginative");
    else if(s.openness < -1) descs.push("practical and down-to-earth");

    if(s.agreeableness > 1) descs.push("kind and cooperative");
    else if(s.agreeableness < -1) descs.push("competitive or skeptical");

    if(s.conscientiousness > 1) descs.push("organized and diligent");
    else if(s.conscientiousness < -1) descs.push("disorganized or negligent");

    return `Anime style portrait of a young adult with ${descs.join(", ")}, detailed shading, vibrant colors, sharp eyes, stylish modern clothes`;
  }

  function updateRadarChart(sc) {
    const s = adjustScoresByAgeGender(sc);
    // 线性映射，-24~24 → 0~100
    const maxAbs = 24;
    const data = geneTags.map(tag => {
      let val = s[tag] || 0;
      let score = ((val + maxAbs) / (2 * maxAbs)) * 100;
      if(score > 100) score = 100;
      if(score < 0) score = 0;
      return Math.round(score);
    });

    const labelsCN = geneTags.map(t => geneTagsCN[t]);

    if(radarChart) {
      radarChart.data.labels = labelsCN;
      radarChart.data.datasets[0].data = data;
      radarChart.update();
    } else {
      const ctx = document.getElementById("radarChart").getContext("2d");
      radarChart = new Chart(ctx, {
        type: 'radar',
        data: {
          labels: labelsCN,
          datasets: [{
            label: '基因科学得分',
            data: data,
            backgroundColor: 'rgba(67,56,202,0.2)',
            borderColor: 'rgba(67,56,202,1)',
            borderWidth: 2,
            pointBackgroundColor: 'rgba(67,56,202,1)'
          }]
        },
        options: {
          scales: {
            r: {
              min: 0,
              max: 100,
              ticks: { stepSize: 20 },
              pointLabels: { font: { size: 14 } }
            }
          },
          plugins: {
            legend: { display: true, position: 'top' }
          },
          maintainAspectRatio: false,
          responsive: true
        }
      });
    }
  }

  btnStart.onclick = () => {
    const a = parseInt(inputAge.value);
    if(isNaN(a) || a < 18 || a > 80){
      alert("请输入有效年龄（18-80）");
      return;
    }
    age = a;
    gender = selectGender.value;
    resetScores();
    currentQuestionIndex = 0;
    sectionUserInfo.style.display = "none";
    sectionResult.style.display = "none";
    sectionQuiz.style.display = "block";
    renderQuestion();
  };

  btnNext.onclick = () => {
    if(selectedOption === null) return;

    // 累加当前题选项的标签分数
    const q = questions[currentQuestionIndex];
    const optTags = q.options[selectedOption].tags;
    for(const key in optTags){
      if(scores.hasOwnProperty(key)){
        scores[key] += optTags[key];
      } else {
        scores[key] = optTags[key];
      }
    }

    currentQuestionIndex++;
    if(currentQuestionIndex >= questions.length){
      // 答题结束
      sectionQuiz.style.display = "none";
      sectionResult.style.display = "block";

      // 计算总分平均（0-100）
      updateRadarChart(scores);
      const avg = Math.round(geneTags.reduce((sum, t) => sum + (scores[t]||0), 0) / geneTags.length);
      totalScoreEl.textContent = `综合得分（平均）：${avg}`;

      promptOutput.textContent = generatePrompt(scores);
    } else {
      renderQuestion();
    }
  };

  btnRestart.onclick = () => {
    sectionResult.style.display = "none";
    sectionUserInfo.style.display = "block";
    resetScores();
  };
</script>

</body>
</html>
