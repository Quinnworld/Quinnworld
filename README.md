<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>标签问卷示范+雷达图+Prompt生成</title>
<style>
  body { max-width: 600px; margin: 20px auto; font-family: "微软雅黑", sans-serif; background: #f9fbfd; padding: 20px; }
  h2, h3 { text-align: center; }
  .section { margin-bottom: 20px; }
  .question { font-weight: 700; margin-bottom: 10px; }
  .option { background: white; border: 1px solid #bbb; border-radius: 6px; padding: 10px; margin: 6px 0; cursor: pointer; user-select:none; }
  .option.selected { background: #60a5fa; color: white; border-color: #3b82f6; }
  button { width: 100%; padding: 12px; font-size: 16px; border: none; border-radius: 6px; background: #3b82f6; color: white; cursor: pointer; }
  button:disabled { background: #93c5fd; cursor: not-allowed; }
  #radarChartContainer { margin-top: 20px; text-align: center; }
  #interpretation, #promptOutput { white-space: pre-wrap; margin-top: 10px; font-size: 14px; color: #333; }
  #promptOutput { background: #eef6ff; border: 1px solid #3b82f6; padding: 10px; border-radius: 6px; user-select: all; cursor: pointer; }
  #scoreOutput { font-weight: 700; font-size: 18px; margin-top: 10px; color: #1e40af; }
</style>
</head>
<body>

<h2>标签问卷示范</h2>

<div id="sectionQuiz" class="section">
  <div id="questionText" class="question"></div>
  <div id="optionsContainer"></div>
  <button id="btnNext" disabled>下一题</button>
</div>

<div id="sectionResult" class="section" style="display:none;">
  <h3>分析结果</h3>
  <div id="radarChartContainer">
    <canvas id="radarChart" width="400" height="400"></canvas>
    <div id="scoreOutput"></div> 
    <div id="interpretation"></div>
  </div>
  <h3>生成的图像Prompt（点击复制）</h3>
  <div id="promptOutput" title="点击复制全部"></div>
  <button id="btnRestart">重新开始</button>
</div>

<!-- 这里用官方CDN加载Chart.js -->
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.3.0/dist/chart.umd.min.js"></script>
<script>
  // 问题和标签，简化版，每题对应一个标签，分数-2到+2
  const questionPool = [
    {
      id: "Q1",
      text: "你喜欢参加社交活动吗？",
      options: [
        { text: "非常不喜欢", tags: { extraversion: -2 } },
        { text: "不太喜欢", tags: { extraversion: -1 } },
        { text: "比较喜欢", tags: { extraversion: 1 } },
        { text: "非常喜欢", tags: { extraversion: 2 } }
      ]
    },
    {
      id: "Q2",
      text: "遇到困难时你的情绪如何？",
      options: [
        { text: "非常焦虑", tags: { emotion_stability: -2 } },
        { text: "有些紧张", tags: { emotion_stability: -1 } },
        { text: "基本稳定", tags: { emotion_stability: 1 } },
        { text: "非常冷静", tags: { emotion_stability: 2 } }
      ]
    },
    {
      id: "Q3",
      text: "你喜欢尝试新鲜事物吗？",
      options: [
        { text: "完全不喜欢", tags: { novelty_seek: -2 } },
        { text: "不太喜欢", tags: { novelty_seek: -1 } },
        { text: "比较喜欢", tags: { novelty_seek: 1 } },
        { text: "非常喜欢", tags: { novelty_seek: 2 } }
      ]
    },
    {
      id: "Q4",
      text: "你处理任务时的态度？",
      options: [
        { text: "经常拖延", tags: { responsibility: -2 } },
        { text: "偶尔拖延", tags: { responsibility: -1 } },
        { text: "比较负责", tags: { responsibility: 1 } },
        { text: "非常负责", tags: { responsibility: 2 } }
      ]
    }
  ];

  // 初始化标签分数
  let tagScores = {
    extraversion: 0,
    emotion_stability: 0,
    novelty_seek: 0,
    responsibility: 0
  };

  let currentQuestionIndex = 0;
  let selectedOptionIndex = null;

  const qTextEl = document.getElementById("questionText");
  const optionsEl = document.getElementById("optionsContainer");
  const btnNext = document.getElementById("btnNext");
  const sectionQuiz = document.getElementById("sectionQuiz");
  const sectionResult = document.getElementById("sectionResult");
  const interpretationEl = document.getElementById("interpretation");
  const promptOutput = document.getElementById("promptOutput");
  const scoreOutput = document.getElementById("scoreOutput");

  // Chart.js实例变量
  let radarChart = null;

  // 渲染题目
  function renderQuestion() {
    const question = questionPool[currentQuestionIndex];
    qTextEl.textContent = question.text;
    optionsEl.innerHTML = "";
    btnNext.disabled = true;
    selectedOptionIndex = null;
    question.options.forEach((opt, i) => {
      const div = document.createElement("div");
      div.className = "option";
      div.textContent = opt.text;
      div.onclick = () => {
        selectOption(i);
      };
      optionsEl.appendChild(div);
    });
  }

  // 选项点击
  function selectOption(idx) {
    selectedOptionIndex = idx;
    Array.from(optionsEl.children).forEach((el, i) => {
      el.classList.toggle("selected", i === idx);
    });
    btnNext.disabled = false;
  }

  btnNext.onclick = () => {
    if (selectedOptionIndex === null) return;
    const question = questionPool[currentQuestionIndex];
    const selectedOpt = question.options[selectedOptionIndex];
    for (const tag in selectedOpt.tags) {
      tagScores[tag] += selectedOpt.tags[tag];
    }
    currentQuestionIndex++;
    if (currentQuestionIndex >= questionPool.length) {
      showResult();
    } else {
      renderQuestion();
    }
  };

  // 渲染雷达图
  function renderRadarChart() {
    const ctx = document.getElementById("radarChart").getContext("2d");
    const labels = Object.keys(tagScores);
    const dataValues = labels.map(tag => tagScores[tag] + 2); // 使数据正向化（范围0~4）
    const maxScore = 4; 

    if (radarChart) {
      radarChart.destroy();
    }

    radarChart = new Chart(ctx, {
      type: "radar",
      data: {
        labels: labels,
        datasets: [{
          label: "标签评分",
          data: dataValues,
          fill: true,
          backgroundColor: "rgba(59, 130, 246, 0.2)",
          borderColor: "rgba(59, 130, 246, 1)",
          pointBackgroundColor: "rgba(59, 130, 246, 1)",
          pointBorderColor: "#fff",
          pointHoverBackgroundColor: "#fff",
          pointHoverBorderColor: "rgba(59, 130, 246, 1)"
        }]
      },
      options: {
        scales: {
          r: {
            min: 0,
            max: maxScore,
            ticks: { stepSize: 1 }
          }
        },
        plugins: {
          legend: { display: false }
        }
      }
    });
  }

  // 简要解读
  function renderInterpretation() {
    let lines = [];
    for (const tag in tagScores) {
      const val = tagScores[tag];
      if (val >= 2) lines.push(`${tag}：显著偏高`);
      else if (val <= -2) lines.push(`${tag}：显著偏低`);
      else lines.push(`${tag}：正常范围`);
    }
    interpretationEl.textContent = lines.join("\n");
  }

  // 计算综合得分（0~100）
  function renderScore() {
    // 分数范围[-2,2], 转成0~4
    let total = 0;
    let count = 0;
    for (const tag in tagScores) {
      total += tagScores[tag] + 2;
      count++;
    }
    const avg = total / count;
    const score = Math.round((avg / 4) * 100);
    scoreOutput.textContent = `综合得分（满分100）：${score}`;
  }

  // 生成Prompt文本
  function renderPrompt() {
    let traits = [];
    if (tagScores.extraversion >= 1) traits.push("outgoing and sociable");
    else if (tagScores.extraversion <= -1) traits.push("introverted and reserved");
    else traits.push("balanced social behavior");

    if (tagScores.emotion_stability >= 1) traits.push("emotionally stable");
    else if (tagScores.emotion_stability <= -1) traits.push("emotionally sensitive");
    else traits.push("moderate emotional responses");

    if (tagScores.novelty_seek >= 1) traits.push("curious and open to new experiences");
    else if (tagScores.novelty_seek <= -1) traits.push("prefers routine and familiarity");
    else traits.push("moderate openness");

    if (tagScores.responsibility >= 1) traits.push("responsible and conscientious");
    else if (tagScores.responsibility <= -1) traits.push("sometimes irresponsible");

    const prompt = `Anime-style portrait of a person who is ${traits.join(", ")}, with a balanced and natural appearance, realistic lighting, soft colors, detailed eyes and hair, modern casual clothing, digital art.`;

    promptOutput.textContent = prompt;

    promptOutput.onclick = () => {
      navigator.clipboard.writeText(prompt);
      alert("Prompt 已复制到剪贴板！");
    };
  }

  // 显示结果页
  function showResult() {
    sectionQuiz.style.display = "none";
    sectionResult.style.display = "block";
    renderRadarChart();
    renderInterpretation();
    renderPrompt();
    renderScore();
  }

  // 重置开始
  document.getElementById("btnRestart").onclick = () => {
    tagScores = {
      extraversion: 0,
      emotion_stability: 0,
      novelty_seek: 0,
      responsibility: 0
    };
    currentQuestionIndex = 0;
    sectionResult.style.display = "none";
    sectionQuiz.style.display = "block";
    renderQuestion();
  };

  // 页面加载自动开始
  renderQuestion();
</script>

</body>
</html>
