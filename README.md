<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>基因科学问卷 + 雷达图 + 动漫Prompt生成</title>
<style>
  body { max-width: 600px; margin: 20px auto; font-family: "微软雅黑", sans-serif; background: #f0f4f8; padding: 20px; }
  h2 { text-align: center; }
  label { display: block; margin: 10px 0 5px; font-weight: 600; }
  select, input[type=number] { width: 100%; padding: 8px; font-size: 16px; margin-bottom: 15px; border-radius: 6px; border: 1px solid #ccc; }
  .section { margin-bottom: 20px; }
  .question { font-weight: 700; margin-bottom: 10px; }
  .option { background: white; border: 1px solid #bbb; border-radius: 6px; padding: 10px; margin: 6px 0; cursor: pointer; user-select:none; }
  .option.selected { background: #4f46e5; color: white; border-color: #4338ca; }
  button { width: 100%; padding: 12px; font-size: 16px; border: none; border-radius: 6px; background: #4338ca; color: white; cursor: pointer; }
  button:disabled { background: #a5b4fc; cursor: not-allowed; }
  pre { background: #eee; padding: 10px; border-radius: 6px; overflow-x: hidden; white-space: pre-wrap; word-break: break-word; user-select: text; }
  #promptOutput { max-height: 130px; overflow: hidden; }
  #totalScoreText { font-size: 24px; font-weight: bold; margin-top: 10px; text-align: center; }
  canvas { display: block; margin: 20px auto; max-width: 100%; }
</style>
</head>
<body>

<h2>基因科学问卷 + 雷达图 + 动漫Prompt生成</h2>

<div class="section" id="sectionUserInfo">
  <label for="inputAge">年龄（不限）</label>
  <input type="number" id="inputAge" placeholder="请输入年龄" />
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
  <h3>答题结束，基因维度雷达图：</h3>
  <canvas id="radarCanvas" width="400" height="400"></canvas>
  <div id="totalScoreText"></div>
  <h3>生成的动漫风格Prompt：</h3>
  <pre id="promptOutput"></pre>
  <button id="btnRestart">重新开始</button>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
  // 六个基因维度中文标签
  const geneDimensions = [
    "外向性",
    "情绪稳定性",
    "新奇寻求",
    "责任心",
    "冲动控制",
    "开放性"
  ];

  // 12题题库，四选项，每选项对基因维度得分 -2~2
  const questionPool = [
    { id:"Q1", text:"你喜欢参加社交活动吗？", options:[
      {text:"非常不喜欢", tags: {外向性: -2}},
      {text:"不太喜欢", tags: {外向性: -1}},
      {text:"比较喜欢", tags: {外向性: 1}},
      {text:"非常喜欢", tags: {外向性: 2}}
    ]},
    { id:"Q2", text:"面对压力时你的情绪表现？", options:[
      {text:"非常紧张和焦虑", tags: {情绪稳定性: -2}},
      {text:"有些不安", tags: {情绪稳定性: -1}},
      {text:"情绪稳定", tags: {情绪稳定性: 1}},
      {text:"非常冷静", tags: {情绪稳定性: 2}}
    ]},
    { id:"Q3", text:"你喜欢尝试新鲜事物吗？", options:[
      {text:"完全不喜欢", tags: {新奇寻求: -2}},
      {text:"不太喜欢", tags: {新奇寻求: -1}},
      {text:"比较喜欢", tags: {新奇寻求: 1}},
      {text:"非常喜欢", tags: {新奇寻求: 2}}
    ]},
    { id:"Q4", text:"你通常是否按时完成任务？", options:[
      {text:"经常拖延", tags: {责任心: -2}},
      {text:"偶尔拖延", tags: {责任心: -1}},
      {text:"大部分时间按时", tags: {责任心: 1}},
      {text:"总是按时完成", tags: {责任心: 2}}
    ]},
    { id:"Q5", text:"你做决定时是否容易冲动？", options:[
      {text:"完全不会冲动", tags: {冲动控制: 2}},
      {text:"不太冲动", tags: {冲动控制: 1}},
      {text:"有时冲动", tags: {冲动控制: -1}},
      {text:"非常冲动", tags: {冲动控制: -2}}
    ]},
    { id:"Q6", text:"你喜欢艺术和创造性活动吗？", options:[
      {text:"完全不喜欢", tags: {开放性: -2}},
      {text:"不太喜欢", tags: {开放性: -1}},
      {text:"比较喜欢", tags: {开放性: 1}},
      {text:"非常喜欢", tags: {开放性: 2}}
    ]},
    { id:"Q7", text:"你是否乐于承担责任？", options:[
      {text:"不喜欢", tags: {责任心: -2}},
      {text:"偶尔喜欢", tags: {责任心: -1}},
      {text:"一般", tags: {责任心: 1}},
      {text:"非常喜欢", tags: {责任心: 2}}
    ]},
    { id:"Q8", text:"你遇到新环境时是否容易适应？", options:[
      {text:"很难适应", tags: {新奇寻求: -2}},
      {text:"适应慢", tags: {新奇寻求: -1}},
      {text:"适应较快", tags: {新奇寻求: 1}},
      {text:"非常容易适应", tags: {新奇寻求: 2}}
    ]},
    { id:"Q9", text:"你是否经常控制不住自己的情绪？", options:[
      {text:"经常不能控制", tags: {冲动控制: -2}},
      {text:"偶尔不能控制", tags: {冲动控制: -1}},
      {text:"大多数时候能控制", tags: {冲动控制: 1}},
      {text:"总是能控制", tags: {冲动控制: 2}}
    ]},
    { id:"Q10", text:"你是否喜欢主动与陌生人交流？", options:[
      {text:"完全不喜欢", tags: {外向性: -2}},
      {text:"不太喜欢", tags: {外向性: -1}},
      {text:"比较喜欢", tags: {外向性: 1}},
      {text:"非常喜欢", tags: {外向性: 2}}
    ]},
    { id:"Q11", text:"你如何评价自己的创造力？", options:[
      {text:"很差", tags: {开放性: -2}},
      {text:"一般", tags: {开放性: -1}},
      {text:"较好", tags: {开放性: 1}},
      {text:"非常好", tags: {开放性: 2}}
    ]},
    { id:"Q12", text:"你是否注重生活中的细节和条理？", options:[
      {text:"完全不注重", tags: {责任心: -2}},
      {text:"不太注重", tags: {责任心: -1}},
      {text:"比较注重", tags: {责任心: 1}},
      {text:"非常注重", tags: {责任心: 2}}
    ]}
  ];

  // 状态变量
  let scores = {};
  let askedIds = new Set();
  let currentQuestion = null;
  let selectedOptionIndex = null;
  let questionCount = 0;
  const maxQuestions = 12;
  let age = 0;
  let gender = "male";

  // 页面元素
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
  const inputAge = document.getElementById("inputAge");
  const selectGender = document.getElementById("selectGender");
  const radarCanvas = document.getElementById("radarCanvas");
  const totalScoreText = document.getElementById("totalScoreText");

  let radarChart = null;

  // 选项点击处理
  function selectOption(idx){
    selectedOptionIndex = idx;
    Array.from(optionsEl.children).forEach((el,i)=>{
      el.classList.toggle("selected", i === idx);
    });
    btnNext.disabled = false;
  }

  // 选题 - 按顺序
  function selectNextQuestion(){
    for(let q of questionPool){
      if(!askedIds.has(q.id)){
        return q;
      }
    }
    return null;
  }

  // 渲染题目
  function renderQuestion(q){
    currentQuestion = q;
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
    progressText.textContent = `第 ${questionCount + 1} / ${maxQuestions} 题`;
  }

  // 生成颜色（根据年龄性别简单调色）
  function getColorByGeneType(age, gender){
    // 男性蓝，女性粉，年轻更亮
    let baseColor = gender === "male" ? "rgba(67,56,202," : "rgba(219,112,147,";
    let alpha = age < 30 ? 0.7 : 0.5;
    return baseColor + alpha + ")";
  }

  // 计算并显示结果
  function showResult(){
    sectionQuiz.style.display = "none";
    sectionResult.style.display = "block";

    // 归一化分数 -24 ~ 24 映射0-100（因12题最大2分*1题维度，实际这里最多12*2=24）
    // 但每维度不一定有12题，题目对应维度不同，这里简单假设每维度满分+8，最低-8（总计16范围）
    // 按照题目设计，最大2分*多题，加减范围估计±8
    // 为保证显示合理，用-8~8映射0~100
    const dimensionScoresNormalized = geneDimensions.map(dim => {
      const raw = scores[dim] || 0;
      return Math.min(100, Math.max(0, (raw + 8) / 16 * 100));
    });

    // 总分为6维度平均，保留1位小数
    const totalScoreRaw = dimensionScoresNormalized.reduce((a,b) => a + b, 0) / dimensionScoresNormalized.length;
    const totalScore = totalScoreRaw.toFixed(1);
    totalScoreText.textContent = `总分: ${totalScore}`;

    const chartColor = getColorByGeneType(age, gender);

    // 绘制雷达图
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
          pointBackgroundColor: chartColor.replace("0.7", "1"),
          pointBorderColor: chartColor.replace("0.7", "1"),
          borderWidth: 2,
          pointRadius: 5
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
          legend: { labels: { color: '#222', font: { size: 14, weight: 'bold' } } },
          tooltip: {
            enabled: true,
            callbacks: {
              label: ctx => ctx.formattedValue
            }
          }
        },
        responsive: true,
        maintainAspectRatio: false
      }
    });

    // 生成prompt文本，不滚动，颜色匹配
    let prompt = `Anime-style portrait of a character with genetic trait scores (0-100):\n`;
    geneDimensions.forEach((dim, i) => {
      prompt += `${dim}: ${dimensionScoresNormalized[i].toFixed(0)}\n`;
    });
    prompt += `Age: ${age}\nGender: ${gender === "male" ? "Male" : "Female"}`;

    promptOutput.textContent = prompt;
    promptOutput.style.color = chartColor.replace("0.7", "1");
    totalScoreText.style.color = chartColor.replace("0.7", "1");
  }

  // 下一题按钮事件
  btnNext.onclick = ()=>{
    if(selectedOptionIndex === null) return;
    // 累计得分
    const selTags = currentQuestion.options[selectedOptionIndex].tags;
    for(let t in selTags){
      scores[t] = (scores[t] || 0) + selTags[t];
    }
    askedIds.add(currentQuestion.id);
    questionCount++;
    if(questionCount >= maxQuestions){
      showResult();
    } else {
      const nextQ = selectNextQuestion();
      if(!nextQ){
        showResult();
      } else {
        renderQuestion(nextQ);
      }
    }
  };

  // 重新开始按钮事件
  btnRestart.onclick = ()=>{
    scores = {};
    askedIds.clear();
    questionCount = 0;
    sectionResult.style.display = "none";
    sectionUserInfo.style.display = "block";
  };

  // 开始答题按钮事件
  btnStart.onclick = ()=>{
    const val = inputAge.value.trim();
    age = val === "" ? 0 : parseInt(val);
    if(isNaN(age) || age < 0){
      alert("请输入有效年龄（非负整数）");
      return;
    }
    gender = selectGender.value;
    scores = {};
    askedIds.clear();
    questionCount = 0;
    sectionUserInfo.style.display = "none";
    sectionResult.style.display = "none";
    sectionQuiz.style.display = "block";
    const firstQ = selectNextQuestion();
    renderQuestion(firstQ);
  };
</script>

</body>
</html>
