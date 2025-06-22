<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>标签问卷+雷达图+图像Prompt示范</title>
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
  #promptOutput { background: #eef6ff; border: 1px solid #3b82f6; padding: 10px; border-radius: 6px; user-select: all; }
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
    <div id="scoreOutput"></div> <!-- 新增综合得分显示 -->
    <div id="interpretation"></div>
  </div>
  <h3>生成的图像Prompt</h3>
  <div id="promptOutput" title="点击复制全部"></div>
  <button id="btnRestart">重新开始</button>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
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
    },
    {
      id: "Q5",
      text: "你做决定时是否冲动？",
      options: [
        { text: "非常冲动", tags: { impulsivity: 2 } },
        { text: "偶尔冲动", tags: { impulsivity: 1 } },
        { text: "比较冷静", tags: { impulsivity: -1 } },
        { text: "非常冷静", tags: { impulsivity: -2 } }
      ]
    },
    {
      id: "Q6",
      text: "你容易感到焦虑吗？",
      options: [
        { text: "非常容易", tags: { anxiety: 2 } },
        { text: "偶尔感到", tags: { anxiety: 1 } },
        { text: "不太容易", tags: { anxiety: -1 } },
        { text: "几乎不会", tags: { anxiety: -2 } }
      ]
    },
    {
      id: "Q7",
      text: "你的自控力如何？",
      options: [
        { text: "非常差", tags: { self_control: -2 } },
        { text: "一般", tags: { self_control: -1 } },
        { text: "较好", tags: { self_control: 1 } },
        { text: "非常好", tags: { self_control: 2 } }
      ]
    },
    {
      id: "Q8",
      text: "你喜欢开放新观点吗？",
      options: [
        { text: "非常排斥", tags: { openness: -2 } },
        { text: "有点抵触", tags: { openness: -1 } },
        { text: "比较接受", tags: { openness: 1 } },
        { text: "非常开放", tags: { openness: 2 } }
      ]
    },
    {
      id: "Q9",
      text: "你是一个容易信任别人的人吗？",
      options: [
        { text: "非常不信任", tags: { agreeableness: -2 } },
        { text: "不太信任", tags: { agreeableness: -1 } },
        { text: "比较信任", tags: { agreeableness: 1 } },
        { text: "非常信任", tags: { agreeableness: 2 } }
      ]
    },
    {
      id: "Q10",
      text: "你对工作是否尽责？",
      options: [
        { text: "非常不尽责", tags: { conscientiousness: -2 } },
        { text: "有时不尽责", tags: { conscientiousness: -1 } },
        { text: "比较尽责", tags: { conscientiousness: 1 } },
        { text: "非常尽责", tags: { conscientiousness: 2 } }
      ]
    },
    {
      id: "Q11",
      text: "你在压力下表现如何？",
      options: [
        { text: "极易崩溃", tags: { stress_resilience: -2 } },
        { text: "有时崩溃", tags: { stress_resilience: -1 } },
        { text: "较好应对", tags: { stress_resilience: 1 } },
        { text: "非常稳定", tags: { stress_resilience: 2 } }
      ]
    },
    {
      id: "Q12",
      text: "你是否喜欢计划未来？",
      options: [
        { text: "完全不喜欢", tags: { future_planning: -2 } },
        { text: "不太喜欢", tags: { future_planning: -1 } },
        { text: "比较喜欢", tags: { future_planning: 1 } },
        { text: "非常喜欢", tags: { future_planning: 2 } }
      ]
    }
  ];

  let tagScores = {
    extraversion: 0,
    emotion_stability: 0,
    novelty_seek: 0,
    responsibility: 0,
    impulsivity: 0,
    anxiety: 0,
    self_control: 0,
    openness: 0,
    agreeableness: 0,
    conscientiousness: 0,
    stress_resilience: 0,
    future_planning: 0
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

  function renderQuestion() {
    const question = questionPool[currentQuestionIndex];
    qTextEl.textContent = question.text;
    optionsEl.innerHTML = "";
    btnNext.disabled = true;
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
    selectedOptionIndex = null;
    if (currentQuestionIndex >= questionPool.length) {
      showResult();
    } else {
      renderQuestion();
    }
  };

  function showResult() {
    sectionQuiz.style.display = "none";
    sectionResult.style.display = "block";
    renderRadarChart();
    renderInterpretation();
    renderPrompt();
    renderScore(); // 新增综合得分显示调用
  }

  function renderInterpretation() {
    let lines = [];
    for (const tag in tagScores) {
      const val = tagScores[tag];
      if (val >= 6) lines.push(`${tag}：显著偏高`);
      else if (val <= -6) lines.push(`${tag}：显著偏低`);
      else lines.push(`${tag}：正常范围`);
    }
    interpretationEl.textContent = lines.join("\n");
  }

  // 新增：计算综合得分，满分100
  function renderScore() {
    // 每个标签满分为 12 (因为12题每题最大+2或-2，12*2=24，满分=12*2=24;这里按绝对值24算，平移到0-100分)
    // 先把所有标签分数归一化到0-24 (即加12)，然后算平均
    // tagScores 可能负，范围-24到+24，因为12题，每题最大2分，但实际我们用了12题，每个标签分最大2，题中只有一题对应一个标签，所以标签分最大约为2*次数
    // 简单起见，取分数范围[-12,12]（因为每标签大约对应1题，每题最大+2），转成0-24
    // 但为了精度，我们先算所有标签绝对值最大12，转为0-24
    // 实际上标签分数范围-24到+24不等，样例中最多±8，简化用±12

    const maxAbsScore = 12;
    let totalNormalized = 0;
    let count = 0;
    for (const tag in tagScores) {
      // 归一化到0~24
      let norm = tagScores[tag] + maxAbsScore;
      if (norm < 0) norm = 0;
      if (norm > 24) norm = 24;
      totalNormalized += norm;
      count++;
    }
    // 平均归一化分数（0~24）
    const avgNorm = totalNormalized / count;
    // 转为0~100分
    const score100 = Math.round((avgNorm / 24) * 100);

    scoreOutput.textContent = `综合得分（满分100）：${score100}`;
  }

  function renderPrompt() {
    let traits = [];
    if (tagScores.extraversion >= 2) traits.push("outgoing and sociable");
    else if (tagScores.extraversion <= -2) traits.push("introverted and reserved");
    else traits.push("balanced social behavior");

    if (tagScores.emotion_stability >= 2) traits.push("emotionally stable");
    else if (tagScores.emotion_stability <= -2) traits.push("emotionally sensitive");
    else traits.push("moderate emotional responses");

    if (tagScores.novelty_seek >= 2) traits.push("curious and open to new experiences");
    else if (tagScores.novelty_seek <= -2) traits.push("prefers routine and familiarity");
    else traits.push("moderate openness");

    if (tagScores.responsibility >= 2) traits.push("responsible and conscientious");
    else if (tagScores.responsibility <= -2) traits.push("sometimes irresponsible");

    if (tagScores.impulsivity >= 2) traits.push("impulsive behavior");
    else if (tagScores.impulsivity <= -2) traits.push("calm and deliberate");

    if (tagScores.anxiety >= 2) traits.push("anxious demeanor");
    else if (tagScores.anxiety <= -2) traits.push("calm under pressure");

    if (tagScores.self_control >= 2) traits.push("strong self-control");
    else if (tagScores.self_control <= -2) traits.push("poor self-control");

    if (tagScores.openness >= 2) traits.push("very open-minded");
    else if (tagScores.openness <= -2) traits.push("closed-minded");

    if (tagScores.agreeableness >= 2) traits.push("very trusting and cooperative");
    else if (tagScores.agreeableness <= -2) traits.push("distrustful");

    if (tagScores.conscientiousness >= 2) traits.push("highly conscientious");
    else if (tagScores.conscientiousness <= -2) traits.push("lack of conscientiousness");

    if (tagScores.stress_resilience >= 2) traits.push("stress-resilient");
    else if (tagScores.stress_resilience <= -2) traits.push("stress-sensitive");

    if (tagScores.future_planning >= 2) traits.push("future-oriented planner");
    else if (tagScores.future_planning <= -2) traits.push("present-focused");

    const prompt = `Anime-style portrait of a person who is ${traits.join(", ")}, with a balanced and natural appearance, realistic lighting, soft colors, detailed eyes and hair, modern casual clothing, digital art.`;

    promptOutput.textContent = prompt;
    promptOutput.onclick = () => {
      navigator.clipboard.writeText(prompt);
      alert("Prompt 已复制到剪贴板！");
    };
  }

  document.getElementById("btnRestart").onclick = () => {
    tagScores = {
      extraversion: 0,
      emotion_stability: 0,
      novelty_seek: 0,
      responsibility: 0,
      impulsivity: 0,
      anxiety: 0,
      self_control: 0,
      openness: 0,
      agreeableness: 0,
      conscientiousness: 0,
      stress_resilience: 0,
      future_planning: 0
    };
    currentQuestionIndex = 0;
    sectionResult.style.display = "none";
    sectionQuiz.style.display = "block";
    renderQuestion();
  };

  renderQuestion();
</script>

</body>
</html>
