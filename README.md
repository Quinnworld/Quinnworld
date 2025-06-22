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
  <h3>Generated Anime-style Prompt:</h3>
  <pre id="promptOutput" readonly></pre>
  <button id="btnRestart">重新开始</button>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
  // 基因维度，6个维度
  const geneDimensions = [
    "Cognitive Ability",
    "Physical Strength",
    "Emotional Stability",
    "Novelty Seeking",
    "Conscientiousness",
    "Social Agreeableness"
  ];

  // 12题题库，4选项，每选项关联维度分数（-2到2）
  const questions = [
    {
      text: "你觉得自己学习和理解新事物的能力如何？",
      options: [
        { text: "很差", scores: { "Cognitive Ability": -2 } },
        { text: "一般", scores: { "Cognitive Ability": -1 } },
        { text: "较好", scores: { "Cognitive Ability": 1 } },
        { text: "非常好", scores: { "Cognitive Ability": 2 } }
      ]
    },
    {
      text: "你在体育运动或体力活动中表现如何？",
      options: [
        { text: "很差", scores: { "Physical Strength": -2 } },
        { text: "一般", scores: { "Physical Strength": -1 } },
        { text: "较好", scores: { "Physical Strength": 1 } },
        { text: "非常好", scores: { "Physical Strength": 2 } }
      ]
    },
    {
      text: "面对压力和挑战时，你的情绪稳定性如何？",
      options: [
        { text: "很差，容易情绪波动", scores: { "Emotional Stability": -2 } },
        { text: "一般", scores: { "Emotional Stability": -1 } },
        { text: "较稳定", scores: { "Emotional Stability": 1 } },
        { text: "非常稳定", scores: { "Emotional Stability": 2 } }
      ]
    },
    {
      text: "你喜欢尝试新鲜和刺激的事情吗？",
      options: [
        { text: "不喜欢", scores: { "Novelty Seeking": -2 } },
        { text: "偶尔喜欢", scores: { "Novelty Seeking": -1 } },
        { text: "比较喜欢", scores: { "Novelty Seeking": 1 } },
        { text: "非常喜欢", scores: { "Novelty Seeking": 2 } }
      ]
    },
    {
      text: "你做事情是否有计划性和条理？",
      options: [
        { text: "很差", scores: { "Conscientiousness": -2 } },
        { text: "一般", scores: { "Conscientiousness": -1 } },
        { text: "较好", scores: { "Conscientiousness": 1 } },
        { text: "非常好", scores: { "Conscientiousness": 2 } }
      ]
    },
    {
      text: "你与他人相处时的友好和合作程度？",
      options: [
        { text: "不友好", scores: { "Social Agreeableness": -2 } },
        { text: "一般", scores: { "Social Agreeableness": -1 } },
        { text: "较友好", scores: { "Social Agreeableness": 1 } },
        { text: "非常友好", scores: { "Social Agreeableness": 2 } }
      ]
    },
    // 7-12题内容和前6题相似，强化各维度综合评分
    {
      text: "你学习新技能的速度如何？",
      options: [
        { text: "很慢", scores: { "Cognitive Ability": -2 } },
        { text: "一般", scores: { "Cognitive Ability": -1 } },
        { text: "较快", scores: { "Cognitive Ability": 1 } },
        { text: "非常快", scores: { "Cognitive Ability": 2 } }
      ]
    },
    {
      text: "你平时的身体耐力如何？",
      options: [
        { text: "很差", scores: { "Physical Strength": -2 } },
        { text: "一般", scores: { "Physical Strength": -1 } },
        { text: "较好", scores: { "Physical Strength": 1 } },
        { text: "非常好", scores: { "Physical Strength": 2 } }
      ]
    },
    {
      text: "你处理情绪问题的能力？",
      options: [
        { text: "很差", scores: { "Emotional Stability": -2 } },
        { text: "一般", scores: { "Emotional Stability": -1 } },
        { text: "较好", scores: { "Emotional Stability": 1 } },
        { text: "非常好", scores: { "Emotional Stability": 2 } }
      ]
    },
    {
      text: "面对新环境，你适应的速度？",
      options: [
        { text: "很慢", scores: { "Novelty Seeking": -2 } },
        { text: "一般", scores: { "Novelty Seeking": -1 } },
        { text: "较快", scores: { "Novelty Seeking": 1 } },
        { text: "非常快", scores: { "Novelty Seeking": 2 } }
      ]
    },
    {
      text: "你对细节的关注程度？",
      options: [
        { text: "很差", scores: { "Conscientiousness": -2 } },
        { text: "一般", scores: { "Conscientiousness": -1 } },
        { text: "较好", scores: { "Conscientiousness": 1 } },
        { text: "非常好", scores: { "Conscientiousness": 2 } }
      ]
    },
    {
      text: "你对团队合作的投入？",
      options: [
        { text: "很少", scores: { "Social Agreeableness": -2 } },
        { text: "一般", scores: { "Social Agreeableness": -1 } },
        { text: "较多", scores: { "Social Agreeableness": 1 } },
        { text: "非常多", scores: { "Social Agreeableness": 2 } }
      ]
    }
  ];

  // 状态变量
  let age = null;
  let gender = null;
  let currentQuestionIndex = 0;
  let scores = {}; // 维度分数累加

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

  let selectedOptionIndex = -1;
  let radarChart = null;

  btnStart.onclick = () => {
    const val = inputAge.value.trim();
    if (val === "" || isNaN(val)) {
      alert("请输入有效年龄");
      return;
    }
    age = val;
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
    // 累加当前选项分数
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

  function showResult(){
    // 计算满分 2 * 12题 / 6维度 => 每维最大理论分 4题 * 2分 = 8，最小-8，归一化到0-100
    const dimensionScoresNormalized = geneDimensions.map(dim => {
      const raw = scores[dim];
      const norm = (raw + 8) / 16 * 100; // -8~8 映射到0~100
      return Math.round(norm);
    });

    // 绘制雷达图
    if(radarChart){
      radarChart.destroy();
    }
    const ctx = document.getElementById("radarChart").getContext("2d");
    radarChart = new Chart(ctx, {
      type: 'radar',
      data: {
        labels: geneDimensions,
        datasets: [{
          label: '基因维度得分',
          data: dimensionScoresNormalized,
          fill: true,
          backgroundColor: 'rgba(67, 56, 202, 0.3)',
          borderColor: 'rgba(67, 56, 202, 0.9)',
          pointBackgroundColor: 'rgba(67, 56, 202, 1)'
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

    // 生成英文prompt（不可翻动）
    // 只描述维度和得分，格式固定
    let prompt = `Anime-style portrait of a character with the following genetic traits scores (0-100):\n`;
    geneDimensions.forEach((dim,i)=>{
      prompt += `${dim}: ${dimensionScoresNormalized[i]}\n`;
    });
    prompt += `Age: ${age}\nGender: ${gender === "male" ? "Male" : "Female"}`;

    promptOutput.textContent = prompt;
  }
</script>

</body>
</html>
