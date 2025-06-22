<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>基因科学问卷 + 雷达图 + 动漫Prompt生成</title>
<style>
  body {
    max-width: 600px;
    margin: 20px auto;
    font-family: "微软雅黑", "Microsoft YaHei", sans-serif;
    background: #f0f4f8;
    padding: 20px;
    color: #222;
  }
  h2 {
    text-align: center;
    margin-bottom: 24px;
  }
  label {
    display: block;
    margin: 12px 0 6px;
    font-weight: 600;
  }
  input[type=number], select {
    width: 100%;
    padding: 8px;
    font-size: 16px;
    border-radius: 6px;
    border: 1px solid #ccc;
  }
  .section {
    margin-bottom: 24px;
  }
  .question-text {
    font-weight: 700;
    margin-bottom: 10px;
  }
  .option {
    background: white;
    border: 1px solid #bbb;
    border-radius: 6px;
    padding: 10px;
    margin: 6px 0;
    cursor: pointer;
    user-select: none;
    transition: background-color 0.25s, color 0.25s;
  }
  .option.selected {
    background: #4338ca;
    color: white;
    border-color: #2c24a8;
  }
  button {
    width: 100%;
    padding: 14px;
    font-size: 16px;
    border: none;
    border-radius: 6px;
    background: #4338ca;
    color: white;
    cursor: pointer;
    transition: background-color 0.3s;
  }
  button:disabled {
    background: #a5b4fc;
    cursor: not-allowed;
  }
  #promptOutput {
    background: #fff;
    border: 1px solid #ccc;
    padding: 12px;
    border-radius: 6px;
    white-space: pre-wrap;
    user-select: text;
    font-family: monospace;
    margin-top: 12px;
    max-height: 140px;
    overflow-y: hidden; /* 禁止滚动，prompt不可翻动 */
  }
  #radarChart {
    max-width: 100%;
    margin-top: 20px;
  }
  #totalScore {
    font-weight: 700;
    font-size: 18px;
    margin-top: 10px;
    text-align: center;
  }
  #progressText {
    margin-top: 12px;
    font-size: 14px;
    color: #555;
    text-align: right;
  }
</style>
</head>
<body>

<h2>基因科学问卷 &amp; 雷达图 &amp; 动漫Prompt生成</h2>

<div id="sectionUserInfo" class="section">
  <label for="inputAge">年龄（任意数字）</label>
  <input type="number" id="inputAge" placeholder="请输入年龄" />
  <label for="selectGender">性别</label>
  <select id="selectGender">
    <option value="male">男性</option>
    <option value="female">女性</option>
  </select>
  <button id="btnStart">开始答题</button>
</div>

<div id="sectionQuiz" class="section" style="display:none;">
  <div id="questionText" class="question-text"></div>
  <div id="optionsContainer"></div>
  <button id="btnNext" disabled>下一题</button>
  <div id="progressText"></div>
</div>

<div id="sectionResult" class="section" style="display:none;">
  <canvas id="radarChart" width="400" height="400"></canvas>
  <div id="totalScore">总分：<span id="scoreValue"></span></div>
  <h3>Generated Anime-style Prompt:</h3>
  <pre id="promptOutput" readonly></pre>
  <button id="btnRestart">重新开始</button>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
  // 中文维度
  const geneDimensions = [
    "认知能力",
    "体力",
    "情绪稳定",
    "新奇寻求",
    "尽责性",
    "社交亲和"
  ];

  // 题库和之前一致
  const questions = [
    {
      text: "你觉得自己学习和理解新事物的能力如何？",
      options: [
        { text: "很差", scores: { "认知能力": -2 } },
        { text: "一般", scores: { "认知能力": -1 } },
        { text: "较好", scores: { "认知能力": 1 } },
        { text: "非常好", scores: { "认知能力": 2 } }
      ]
    },
    {
      text: "你在体育运动或体力活动中表现如何？",
      options: [
        { text: "很差", scores: { "体力": -2 } },
        { text: "一般", scores: { "体力": -1 } },
        { text: "较好", scores: { "体力": 1 } },
        { text: "非常好", scores: { "体力": 2 } }
      ]
    },
    {
      text: "面对压力和挑战时，你的情绪稳定性如何？",
      options: [
        { text: "很差，容易情绪波动", scores: { "情绪稳定": -2 } },
        { text: "一般", scores: { "情绪稳定": -1 } },
        { text: "较稳定", scores: { "情绪稳定": 1 } },
        { text: "非常稳定", scores: { "情绪稳定": 2 } }
      ]
    },
    {
      text: "你喜欢尝试新鲜和刺激的事情吗？",
      options: [
        { text: "不喜欢", scores: { "新奇寻求": -2 } },
        { text: "偶尔喜欢", scores: { "新奇寻求": -1 } },
        { text: "比较喜欢", scores: { "新奇寻求": 1 } },
        { text: "非常喜欢", scores: { "新奇寻求": 2 } }
      ]
    },
    {
      text: "你做事情是否有计划性和条理？",
      options: [
        { text: "很差", scores: { "尽责性": -2 } },
        { text: "一般", scores: { "尽责性": -1 } },
        { text: "较好", scores: { "尽责性": 1 } },
        { text: "非常好", scores: { "尽责性": 2 } }
      ]
    },
    {
      text: "你与他人相处时的友好和合作程度？",
      options: [
        { text: "不友好", scores: { "社交亲和": -2 } },
        { text: "一般", scores: { "社交亲和": -1 } },
        { text: "较友好", scores: { "社交亲和": 1 } },
        { text: "非常友好", scores: { "社交亲和": 2 } }
      ]
    },
    {
      text: "你学习新技能的速度如何？",
      options: [
        { text: "很慢", scores: { "认知能力": -2 } },
        { text: "一般", scores: { "认知能力": -1 } },
        { text: "较快", scores: { "认知能力": 1 } },
        { text: "非常快", scores: { "认知能力": 2 } }
      ]
    },
    {
      text: "你平时的身体耐力如何？",
      options: [
        { text: "很差", scores: { "体力": -2 } },
        { text: "一般", scores: { "体力": -1 } },
        { text: "较好", scores: { "体力": 1 } },
        { text: "非常好", scores: { "体力": 2 } }
      ]
    },
    {
      text: "你处理情绪问题的能力？",
      options: [
        { text: "很差", scores: { "情绪稳定": -2 } },
        { text: "一般", scores: { "情绪稳定": -1 } },
        { text: "较好", scores: { "情绪稳定": 1 } },
        { text: "非常好", scores: { "情绪稳定": 2 } }
      ]
    },
    {
      text: "面对新环境，你适应的速度？",
      options: [
        { text: "很慢", scores: { "新奇寻求": -2 } },
        { text: "一般", scores: { "新奇寻求": -1 } },
        { text: "较快", scores: { "新奇寻求": 1 } },
        { text: "非常快", scores: { "新奇寻求": 2 } }
      ]
    },
    {
      text: "你对细节的关注程度？",
      options: [
        { text: "很差", scores: { "尽责性": -2 } },
        { text: "一般", scores: { "尽责性": -1 } },
        { text: "较好", scores: { "尽责性": 1 } },
        { text: "非常好", scores: { "尽责性": 2 } }
      ]
    },
    {
      text: "你对团队合作的投入？",
      options: [
        { text: "很少", scores: { "社交亲和": -2 } },
        { text: "一般", scores: { "社交亲和": -1 } },
        { text: "较多", scores: { "社交亲和": 1 } },
        { text: "非常多", scores: { "社交亲和": 2 } }
      ]
    }
  ];

  // 状态变量
  let age = null;
  let gender = null;
  let currentQuestionIndex = 0;
  let scores = {};
  const sectionUserInfo = document.getElementById("sectionUserInfo");
  const sectionQuiz = document.getElementById("sectionQuiz");
  const sectionResult = document.getElementById("sectionResult");
  const inputAge = document.getElementById("inputAge");
  const selectGender = document.getElementById("selectGender");
  const btnStart = document.getElementById("btnStart");
  const questionText = document.getElementById("questionText");
  const optionsContainer = document.getElementById("optionsContainer");
  const btnNext = document.getElementById("btnNext");
  const progressText = document.getElementById("progressText");
  const promptOutput = document.getElementById("promptOutput");
  const btnRestart = document.getElementById("btnRestart");
  const radarCanvas = document.getElementById("radarChart");
  const totalScoreText = document.getElementById("scoreValue");

  let selectedOptionIndex = -1;
  let radarChart = null;

  btnStart.onclick = () => {
    const val = inputAge.value.trim();
    if (val === "" || isNaN(val)) {
      alert("请输入有效年龄");
      return;
    }
    age = Number(val);
    gender = selectGender.value;
    currentQuestionIndex = 0;
    selectedOptionIndex = -1;
    scores = {};
    geneDimensions.forEach(d => scores[d] = 0);

    sectionUserInfo.style.display = "none";
    sectionResult.style.display = "none";
    sectionQuiz.style.display = "block";
    btnNext.disabled = true;

    renderQuestion();
  };

  btnRestart.onclick = () => {
    sectionResult.style.display = "none";
    sectionUserInfo.style.display = "block";
    inputAge.value = "";
    selectGender.value = "male";
  };

  btnNext.onclick = () => {
    if(selectedOptionIndex < 0) return;
    const currentQuestion = questions[currentQuestionIndex];
    const selectedScores = currentQuestion.options[selectedOptionIndex].scores;
    for (const dim in selectedScores) {
      scores[dim] += selectedScores[dim];
    }
    currentQuestionIndex++;
    selectedOptionIndex = -1;
    btnNext.disabled = true;

    if(currentQuestionIndex >= questions.length){
      sectionQuiz.style.display = "none";
      showResult();
      sectionResult.style.display = "block";
    } else {
      renderQuestion();
    }
  };

  function renderQuestion(){
    const q = questions[currentQuestionIndex];
    questionText.textContent = q.text;
    optionsContainer.innerHTML = "";
    q.options.forEach((opt, i) => {
      const div = document.createElement("div");
      div.className = "option";
      div.textContent = opt.text;
      div.onclick = () => {
        selectedOptionIndex = i;
        btnNext.disabled = false;
        Array.from(optionsContainer.children).forEach((c,j)=>{
          c.classList.toggle("selected", i === j);
        });
      };
      optionsContainer.appendChild(div);
    });
    progressText.textContent = `题目 ${currentQuestionIndex+1} / ${questions.length}`;
  }

  function getColorByGeneType(age, gender){
    // 简单示例：年龄小于30蓝色，30-50绿色，大于50红色；男性加深颜色，女性淡色
    let baseColor;
    if(age < 30) baseColor = [67, 56, 202];      // 蓝色
    else if(age <= 50) baseColor = [34, 139, 34]; // 绿色
    else baseColor = [178, 34, 34];               // 红色

    if(gender === "female"){
      // 女性颜色淡化
      baseColor = baseColor.map(c => Math.min(255, Math.floor(c + (255 - c) * 0.5)));
    }
    return `rgba(${baseColor[0]}, ${baseColor[1]}, ${baseColor[2]}, 0.7)`;
  }

  function showResult(){
    // 归一化分数 -8 ~ 8 映射0-100
    const dimensionScoresNormalized = geneDimensions.map(dim => {
      const raw = scores[dim];
      return Math.round((raw + 8) / 16 * 100);
    });

    // 计算总分（每题满分2分*12题=24分最大，归一化为600分制）
    let rawTotal = 0;
    for(let d in scores){
      rawTotal += (scores[d] + 8); // 每维度0~16，6维最大96
    }
    const totalScore = Math.round(rawTotal / (16 * geneDimensions.length) * 600);
    totalScoreText.textContent = totalScore;

    const chartColor = getColorByGeneType(age, gender);

    if(radarChart){
      radarChart.destroy();
    }
    const ctx = radarCanvas.getContext("2d");
    radarChart = new Chart(ctx, {
      type: 'radar',
      data: {
        labels: geneDimensions,
        datasets: [{
          label: '基因维度得分',
          data: dimensionScoresNormalized,
          fill: true,
          backgroundColor: chartColor.replace("0.7", "0.3"),
          borderColor: chartColor.replace("0.7", "1"),
          pointBackgroundColor: chartColor.replace("0.7", "1")
        }]
      },
      options: {
        scales: {
          r: {
            min: 0,
            max: 100,
            ticks: { stepSize: 20, color: '#555' },
            grid: { color: '#ccc' },
            pointLabels: { color: '#222', font: { size: 14 } }
          }
        },
        plugins: {
          legend: { labels: { color: '#222', font: { size: 14, weight: 'bold' } } }
        }
      }
    });

    // 生成英文prompt
    let prompt = `Anime-style portrait of a character with genetic trait scores (0-100):\n`;
    geneDimensions.forEach((dim, i) => {
      prompt += `${dim}: ${dimensionScoresNormalized[i]}\n`;
    });
    prompt += `Age: ${age}\nGender: ${gender === "male" ? "Male" : "Female"}`;

    promptOutput.textContent = prompt;
    promptOutput.style.color = chartColor.replace("0.7", "1");
    totalScoreText.style.color = chartColor.replace("0.7", "1");
  }
</script>

</body>
</html>
