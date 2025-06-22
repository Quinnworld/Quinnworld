<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>基因科学问卷 + 动漫Prompt生成（12题版）</title>
<style>
  body { max-width: 600px; margin: 20px auto; font-family: "微软雅黑", sans-serif; background: #f0f4f8; padding: 20px; }
  h2 { text-align: center; }
  label { display: block; margin: 10px 0 5px; font-weight: 600; }
  input[type=number], select { width: 100%; padding: 8px; font-size: 16px; margin-bottom: 15px; border-radius: 6px; border: 1px solid #ccc; }
  .section { margin-bottom: 20px; }
  .question { font-weight: 700; margin-bottom: 10px; }
  .option { background: white; border: 1px solid #bbb; border-radius: 6px; padding: 10px; margin: 6px 0; cursor: pointer; user-select:none; }
  .option.selected { background: #4f46e5; color: white; border-color: #4338ca; }
  button { width: 100%; padding: 12px; font-size: 16px; border: none; border-radius: 6px; background: #4338ca; color: white; cursor: pointer; }
  button:disabled { background: #a5b4fc; cursor: not-allowed; }
  pre { background: #eee; padding: 10px; border-radius: 6px; overflow-x: auto; white-space: pre-wrap; }
  canvas { background: white; border-radius: 6px; margin-top: 15px; }
</style>
</head>
<body>

<h2>基因科学问卷 + 动漫Prompt生成（12题）</h2>

<div class="section" id="sectionUserInfo">
  <label for="inputAge">年龄（不限）</label>
  <input type="number" id="inputAge" placeholder="请输入您的年龄" />
  <label for="selectGender">性别</label>
  <select id="selectGender">
    <option value="male">男性</option>
    <option value="female">女性</option>
  </select>
  <button id="btnStart">开始答题</button>
</div>

<div class="section" id="sectionQuiz" style="display:none;">
  <div id="questionText" class="question"></div>
  <div id="optionsContainer"></div>
  <button id="btnNext" disabled>下一题</button>
  <div style="margin-top:10px; font-size:14px; color:#555;" id="progressText"></div>
</div>

<div class="section" id="sectionResult" style="display:none;">
  <h3>Genetic Trait Scores Radar Chart</h3>
  <canvas id="radarChart" width="400" height="400"></canvas>
  <h3>Generated Anime-style Prompt (English)</h3>
  <pre id="promptOutput"></pre>
  <button id="btnRestart">重新开始</button>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
  // 6个核心基因功能维度，基于基因科学划分
  const geneDimensions = [
    "Extraversion",
    "Emotional Stability",
    "Cognitive Ability",
    "Physical Endurance",
    "Immune Response",
    "Metabolism Efficiency"
  ];

  // 12题题库，4选项，分值-2~2，严格关联6个基因维度
  const questions = [
    { text: "你喜欢参加社交活动吗？", options: [
      { text: "非常不喜欢", scores: {Extraversion:-2} },
      { text: "不太喜欢", scores: {Extraversion:-1} },
      { text: "比较喜欢", scores: {Extraversion:1} },
      { text: "非常喜欢", scores: {Extraversion:2} },
    ]},
    { text: "面对压力时你情绪稳定吗？", options: [
      { text: "非常不稳定", scores: {Emotional Stability:-2} },
      { text: "有些不稳定", scores: {Emotional Stability:-1} },
      { text: "较为稳定", scores: {Emotional Stability:1} },
      { text: "非常稳定", scores: {Emotional Stability:2} },
    ]},
    { text: "你觉得自己学习和思考能力如何？", options: [
      { text: "较差", scores: {Cognitive Ability:-2} },
      { text: "一般", scores: {Cognitive Ability:-1} },
      { text: "较好", scores: {Cognitive Ability:1} },
      { text: "非常好", scores: {Cognitive Ability:2} },
    ]},
    { text: "你平时体力和耐力如何？", options: [
      { text: "很差", scores: {Physical Endurance:-2} },
      { text: "一般", scores: {Physical Endurance:-1} },
      { text: "较好", scores: {Physical Endurance:1} },
      { text: "非常好", scores: {Physical Endurance:2} },
    ]},
    { text: "你容易生病或感冒吗？", options: [
      { text: "非常容易", scores: {Immune Response:-2} },
      { text: "偶尔生病", scores: {Immune Response:-1} },
      { text: "很少生病", scores: {Immune Response:1} },
      { text: "几乎不生病", scores: {Immune Response:2} },
    ]},
    { text: "你的新陈代谢情况如何？", options: [
      { text: "代谢差，容易发胖", scores: {Metabolism Efficiency:-2} },
      { text: "代谢一般", scores: {Metabolism Efficiency:-1} },
      { text: "代谢较好", scores: {Metabolism Efficiency:1} },
      { text: "代谢非常好", scores: {Metabolism Efficiency:2} },
    ]},
    { text: "你喜欢主动和别人交流吗？", options: [
      { text: "不喜欢", scores: {Extraversion:-2} },
      { text: "有时不喜欢", scores: {Extraversion:-1} },
      { text: "比较喜欢", scores: {Extraversion:1} },
      { text: "非常喜欢", scores: {Extraversion:2} },
    ]},
    { text: "遇到困难时你的情绪控制如何？", options: [
      { text: "很差，经常焦虑", scores: {Emotional Stability:-2} },
      { text: "有些波动", scores: {Emotional Stability:-1} },
      { text: "较为平稳", scores: {Emotional Stability:1} },
      { text: "非常平稳", scores: {Emotional Stability:2} },
    ]},
    { text: "你能快速理解复杂事物吗？", options: [
      { text: "较慢", scores: {Cognitive Ability:-2} },
      { text: "一般", scores: {Cognitive Ability:-1} },
      { text: "较快", scores: {Cognitive Ability:1} },
      { text: "非常快", scores: {Cognitive Ability:2} },
    ]},
    { text: "你平时运动耐力如何？", options: [
      { text: "差", scores: {Physical Endurance:-2} },
      { text: "一般", scores: {Physical Endurance:-1} },
      { text: "好", scores: {Physical Endurance:1} },
      { text: "非常好", scores: {Physical Endurance:2} },
    ]},
    { text: "你免疫系统反应强吗？", options: [
      { text: "很弱", scores: {Immune Response:-2} },
      { text: "一般", scores: {Immune Response:-1} },
      { text: "较强", scores: {Immune Response:1} },
      { text: "非常强", scores: {Immune Response:2} },
    ]},
    { text: "你的新陈代谢速度怎么样？", options: [
      { text: "慢", scores: {Metabolism Efficiency:-2} },
      { text: "一般", scores: {Metabolism Efficiency:-1} },
      { text: "快", scores: {Metabolism Efficiency:1} },
      { text: "非常快", scores: {Metabolism Efficiency:2} },
    ]},
  ];

  let scores = {};
  geneDimensions.forEach(d => scores[d] = 0);

  let currentIndex = 0;
  let selectedOption = -1;

  const sectionUserInfo = document.getElementById('sectionUserInfo');
  const sectionQuiz = document.getElementById('sectionQuiz');
  const sectionResult = document.getElementById('sectionResult');

  const inputAge = document.getElementById('inputAge');
  const selectGender = document.getElementById('selectGender');
  const btnStart = document.getElementById('btnStart');
  const questionText = document.getElementById('questionText');
  const optionsContainer = document.getElementById('optionsContainer');
  const btnNext = document.getElementById('btnNext');
  const progressText = document.getElementById('progressText');
  const promptOutput = document.getElementById('promptOutput');
  const btnRestart = document.getElementById('btnRestart');

  btnStart.onclick = () => {
    if (inputAge.value === "") {
      alert("请输入年龄");
      return;
    }
    sectionUserInfo.style.display = "none";
    sectionQuiz.style.display = "block";
    currentIndex = 0;
    resetScores();
    showQuestion(currentIndex);
  };

  btnRestart.onclick = () => {
    sectionResult.style.display = "none";
    sectionUserInfo.style.display = "block";
    btnNext.disabled = true;
    selectedOption = -1;
    promptOutput.textContent = "";
  };

  function resetScores() {
    geneDimensions.forEach(d => scores[d] = 0);
  }

  function showQuestion(idx) {
    selectedOption = -1;
    btnNext.disabled = true;
    let q = questions[idx];
    questionText.textContent = `${idx + 1}. ${q.text}`;
    optionsContainer.innerHTML = "";
    q.options.forEach((opt, i) => {
      const div = document.createElement("div");
      div.className = "option";
      div.textContent = opt.text;
      div.onclick = () => {
        selectedOption = i;
        btnNext.disabled = false;
        Array.from(optionsContainer.children).forEach((c, j) => {
          c.classList.toggle("selected", i === j);
        });
      };
      optionsContainer.appendChild(div);
    });
    progressText.textContent = `题目 ${idx + 1} / ${questions.length}`;
  }

  btnNext.onclick = () => {
    if (selectedOption === -1) return;
    const selectedScores = questions[currentIndex].options[selectedOption].scores;
    for (const dim in selectedScores) {
      if (scores.hasOwnProperty(dim)) {
        scores[dim] += selectedScores[dim];
      }
    }
    currentIndex++;
    if (currentIndex < questions.length) {
      showQuestion(currentIndex);
    } else {
      sectionQuiz.style.display = "none";
      showResult();
      sectionResult.style.display = "block";
    }
  };

  function normalize(val) {
    // 12题，每题范围[-2,2],最大24分，最小-24分，映射0~100
    let norm = (val + questions.length * 2) / (questions.length * 4) * 100;
    if (norm > 100) norm = 100;
    if (norm < 0) norm = 0;
    return Math.round(norm);
  }

  let radarChartInstance = null;
  function showResult() {
    const normalizedScores = geneDimensions.map(d => normalize(scores[d]));
    const age = inputAge.value;
    const gender = selectGender.value === "male" ? "male" : "female";

    renderRadarChart(geneDimensions, normalizedScores);

    let lines = [];
    lines.push(`Anime-style portrait of a ${gender} character, aged ${age},`);
    lines.push("with the following genetic trait scores (0-100 scale):");
    geneDimensions.forEach((d, i) => {
      lines.push(`- ${d}: ${normalizedScores[i]}`);
    });
    lines.push("The character's appearance and personality reflect these genetic influences.");

    promptOutput.textContent = lines.join("\n");
  }

  function renderRadarChart(labels, data) {
    const ctx = document.getElementById("radarChart").getContext("2d");
    if (radarChartInstance) radarChartInstance.destroy();
    radarChartInstance = new Chart(ctx, {
      type: "radar",
      data: {
        labels: labels,
        datasets: [{
          label: "Genetic Trait Scores",
          data: data,
          backgroundColor: "rgba(67,56,202,0.3)",
          borderColor: "rgba(67,56,202,1)",
          borderWidth: 2,
          pointBackgroundColor: "rgba(67,56,202,1)"
        }]
      },
      options: {
        scales: {
          r: {
            min: 0,
            max: 100,
            ticks: { stepSize: 20 },
            pointLabels: { font: { size: 12 } }
          }
        },
        plugins: {
          legend: { display: false }
        }
      }
    });
  }
</script>

</body>
</html>
