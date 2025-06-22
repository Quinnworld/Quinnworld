<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>基因科学多标签问卷 + 多标签雷达图 + 动漫Prompt</title>
<style>
  body { max-width: 700px; margin: 20px auto; font-family: "微软雅黑", sans-serif; background: #f0f4f8; padding: 20px; }
  h2, h3 { text-align: center; }
  label { display: block; margin: 10px 0 5px; font-weight: 600; }
  select, input[type=number] { width: 100%; padding: 8px; font-size: 16px; margin-bottom: 15px; border-radius: 6px; border: 1px solid #ccc; }
  .section { margin-bottom: 20px; }
  .question { font-weight: 700; margin-bottom: 10px; }
  .option { background: white; border: 1px solid #bbb; border-radius: 6px; padding: 10px; margin: 6px 0; cursor: pointer; user-select:none; }
  .option.selected { background: #4f46e5; color: white; border-color: #4338ca; }
  button { width: 100%; padding: 12px; font-size: 16px; border: none; border-radius: 6px; background: #4338ca; color: white; cursor: pointer; }
  button:disabled { background: #a5b4fc; cursor: not-allowed; }
  #radarChart { max-width: 600px; margin: 0 auto; }
  pre { background: #eee; padding: 10px; border-radius: 6px; overflow-x: auto; white-space: pre-wrap; }
</style>
</head>
<body>

<h2>基因科学多标签问卷 + 动漫风格Prompt生成</h2>

<div class="section" id="sectionUserInfo">
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
  <h3>Gene Function Scores Radar Chart</h3>
  <canvas id="radarChart" width="600" height="600"></canvas>
  <h3>Generated Anime-style Prompt:</h3>
  <pre id="promptOutput"></pre>
  <button id="btnRestart">重新开始</button>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
  // 基因功能标签列表
  const geneTags = [
    "attention", "memory", "learning_speed", "creativity", "language_ability",
    "muscle_strength", "endurance", "recovery", "coordination", "flexibility",
    "appetite_control", "lipid_metabolism", "fat_distribution", "energy_expenditure",
    "anxiety", "depression", "impulsivity", "sociality", "empathy",
    "inflammation_resistance", "infection_susceptibility", "allergy_tendency", "autoimmune_risk",
    "cell_repair", "antioxidant_capacity", "mitochondrial_function",
    "sleep_quality", "stress_tolerance", "drug_metabolism", "pain_sensitivity", "mood_stability", "self_control"
  ];

  // 示例题库3题，实际请扩充到250题
  const questionPool = [
    {
      id: "Q001",
      text: "你觉得自己在注意力集中方面表现如何？",
      options: [
        { text: "非常差", tags: { attention: -3, stress_tolerance: -1 } },
        { text: "稍差", tags: { attention: -1, stress_tolerance: 0 } },
        { text: "一般", tags: { attention: 1, stress_tolerance: 1 } },
        { text: "非常好", tags: { attention: 3, stress_tolerance: 2 } }
      ]
    },
    {
      id: "Q002",
      text: "你觉得自己情绪稳定吗？",
      options: [
        { text: "很不稳定", tags: { mood_stability: -3, anxiety: 3 } },
        { text: "有些波动", tags: { mood_stability: -1, anxiety: 1 } },
        { text: "比较稳定", tags: { mood_stability: 1, anxiety: -1 } },
        { text: "非常稳定", tags: { mood_stability: 3, anxiety: -3 } }
      ]
    },
    {
      id: "Q003",
      text: "你做事冲动吗？",
      options: [
        { text: "非常冲动", tags: { impulsivity: 3, self_control: -3 } },
        { text: "有时冲动", tags: { impulsivity: 1, self_control: -1 } },
        { text: "比较克制", tags: { impulsivity: -1, self_control: 1 } },
        { text: "非常克制", tags: { impulsivity: -3, self_control: 3 } }
      ]
    }
  ];

  const maxQuestions = 15; // 每次答题15题
  let selectedQuestions = [];
  let currentIndex = 0;
  let tagScores = {};
  let selectedOptionIndex = null;
  let gender = "male";

  // 页面元素引用
  const sectionUserInfo = document.getElementById("sectionUserInfo");
  const sectionQuiz = document.getElementById("sectionQuiz");
  const sectionResult = document.getElementById("sectionResult");
  const qTextEl = document.getElementById("questionText");
  const optionsEl = document.getElementById("optionsContainer");
  const btnNext = document.getElementById("btnNext");
  const progressText = document.getElementById("progressText");
  const promptOutput = document.getElementById("promptOutput");
  const btnStart = document.getElementById("btnStart");
  const btnRestart = document.getElementById("btnRestart");
  const selectGender = document.getElementById("selectGender");

  // 选项点击事件
  function selectOption(idx){
    selectedOptionIndex = idx;
    Array.from(optionsEl.children).forEach((el,i)=>{
      el.classList.toggle("selected", i === idx);
    });
    btnNext.disabled = false;
  }

  // 数组打乱函数
  function shuffleArray(arr){
    for(let i = arr.length - 1; i > 0; i--){
      let j = Math.floor(Math.random() * (i+1));
      [arr[i], arr[j]] = [arr[j], arr[i]];
    }
    return arr;
  }

  // 开始答题按钮事件
  btnStart.onclick = ()=>{
    gender = selectGender.value;
    tagScores = {};
    selectedOptionIndex = null;
    currentIndex = 0;
    btnNext.disabled = true;
    // 打乱题库并取前maxQuestions题
    selectedQuestions = shuffleArray([...questionPool]).slice(0, maxQuestions);
    sectionUserInfo.style.display = "none";
    sectionResult.style.display = "none";
    sectionQuiz.style.display = "block";
    renderQuestion(selectedQuestions[currentIndex]);
  };

  // 渲染当前题目和选项
  function renderQuestion(q){
    qTextEl.textContent = q.text;
    optionsEl.innerHTML = "";
    selectedOptionIndex = null;
    btnNext.disabled = true;
    for(let i=0; i<q.options.length; i++){
      const opt = document.createElement("div");
      opt.className = "option";
      opt.textContent = q.options[i].text;
      opt.onclick = ()=>selectOption(i);
      optionsEl.appendChild(opt);
    }
    progressText.textContent = `第 ${currentIndex+1} / ${maxQuestions} 题`;
  }

  // 下一题按钮事件
  btnNext.onclick = ()=>{
    if(selectedOptionIndex === null) return;
    // 累加标签分数
    const selectedTags = selectedQuestions[currentIndex].options[selectedOptionIndex].tags;
    for(let tag in selectedTags){
      tagScores[tag] = (tagScores[tag] || 0) + selectedTags[tag];
    }
    currentIndex++;
    if(currentIndex >= maxQuestions){
      showResult();
    } else {
      renderQuestion(selectedQuestions[currentIndex]);
    }
  };

  // 归一化标签分数到0-100
  function normalizeScores(scores){
    let normalized = {};
    const maxScore = maxQuestions * 3; // 最大可能分数
    for(let tag of geneTags){
      let val = scores[tag] || 0;
      let score = ((val + maxScore) / (2 * maxScore)) * 100;
      if(score > 100) score = 100;
      if(score < 0) score = 0;
      normalized[tag] = Math.round(score);
    }
    return normalized;
  }

  // 生成动漫风格英文Prompt（简化示范）
  function generatePrompt(normalizedTags){
    let descs = [];
    if(normalizedTags.creativity > 70) descs.push("creative and imaginative");
    else if(normalizedTags.creativity < 30) descs.push("practical and down-to-earth");

    if(normalizedTags.anxiety > 70) descs.push("often anxious or worried");
    else if(normalizedTags.anxiety < 30) descs.push("calm and relaxed");

    if(normalizedTags.sociality > 70) descs.push("friendly and outgoing");
    else if(normalizedTags.sociality < 30) descs.push("introverted and reserved");

    if(normalizedTags.self_control > 70) descs.push("excellent self-control");
    else if(normalizedTags.self_control < 30) descs.push("impulsive and spontaneous");

    if(normalizedTags.endurance > 70) descs.push("physically strong and enduring");
    else if(normalizedTags.endurance < 30) descs.push("physically delicate and sensitive");

    if(normalizedTags.mood_stability > 70) descs.push("emotionally stable and balanced");
    else if(normalizedTags.mood_stability < 30) descs.push("emotionally volatile");

    return `Anime-style portrait of a ${gender === "male" ? "young man" : "young woman"} who is ${descs.join(", ")}, with an expressive face and dynamic posture.`;
  }

  // 展示结果页面
  function showResult(){
    sectionQuiz.style.display = "none";
    sectionResult.style.display = "block";

    const normalized = normalizeScores(tagScores);

    const ctx = document.getElementById("radarChart").getContext("2d");
    if(window.radarChartInstance) window.radarChartInstance.destroy();

    window.radarChartInstance = new Chart(ctx, {
      type: 'radar',
      data: {
        labels: geneTags.map(t=>t.replace(/_/g, ' ')),
        datasets: [{
          label: 'Gene Function Scores',
          data: geneTags.map(t => normalized[t]),
          fill: true,
          backgroundColor: 'rgba(67,56,202,0.2)',
          borderColor: 'rgba(67,56,202,1)',
          pointBackgroundColor: 'rgba(67,56,202,1)',
          pointBorderColor: '#fff',
          pointHoverBackgroundColor: '#fff',
          pointHoverBorderColor: 'rgba(67,56,202,1)'
        }]
      },
      options: {
        scales: {
          r: {
            min: 0,
            max: 100,
            ticks: { stepSize: 20 },
            pointLabels: { font: { size: 11 } }
          }
        },
        plugins: {
          legend: { display: false }
        }
      }
    });

    promptOutput.textContent = generatePrompt(normalized);
  }

  // 重新开始按钮
  btnRestart.onclick = ()=>{
    sectionResult.style.display = "none";
    sectionUserInfo.style.display = "block";
  }
</script>

</body>
</html>
