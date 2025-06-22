<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>标签问卷</title>
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
  #interpretation { white-space: pre-wrap; margin-top: 10px; font-size: 14px; color: #333; }
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
    <div id="interpretation"></div>
  </div>
  <button id="btnRestart">重新开始</button>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
  // 问卷题库（简化版，每题4选项，对应不同标签分值）
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

  // 初始化标签得分，默认0
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

  // 当前题索引
  let currentQuestionIndex = 0;

  // 页面元素
  const qTextEl = document.getElementById("questionText");
  const optionsEl = document.getElementById("optionsContainer");
  const btnNext = document.getElementById("btnNext");
  const sectionQuiz = document.getElementById("sectionQuiz");
  const sectionResult = document.getElementById("sectionResult");

  // 渲染当前题
  function renderQuestion() {
    const question = questionPool[currentQuestionIndex];
    qTextEl.textContent = question.text;
    optionsEl.innerHTML = "";
    btnNext.disabled = true;

    question.options.forEach((opt, idx) => {
      const div = document.createElement("div");
      div.className = "option";
      div.textContent = opt.text;
      div.onclick = () => {
        selectOption(idx);
      };
      optionsEl.appendChild(div);
    });
  }

  let selectedOptionIndex = null;

  // 选择选项
  function selectOption(idx) {
    selectedOptionIndex = idx;
    Array.from(optionsEl.children).forEach((el, i) => {
      el.classList.toggle("selected", i === idx);
    });
    btnNext.disabled = false;
  }

  // 点击下一题
  btnNext.onclick = () => {
    if (selectedOptionIndex === null) return;

    // 计算标签分数
    const question = questionPool[currentQuestionIndex];
    const selectedTags = question.options[selectedOptionIndex].tags;
    for (const tag in selectedTags) {
      tagScores[tag] += selectedTags[tag];
    }

    currentQuestionIndex++;
    selectedOptionIndex = null;

    if (currentQuestionIndex >= questionPool.length) {
      // 问卷结束，展示结果
      showResult();
    } else {
      renderQuestion();
    }
  };

  // 显示结果并渲染雷达图
  function showResult() {
    sectionQuiz.style.display = "none";
    sectionResult.style.display = "block";
    renderRadarChart(tagScores);
  }

  // 重新开始
  document.getElementById("btnRestart").onclick = () => {
    // 重置状态
    tagScores = Object.keys(tagScores).reduce((acc, key) => { acc[key] = 0; return acc; }, {});
    currentQuestionIndex = 0;
    selectedOptionIndex = null;
    sectionResult.style.display = "none";
    sectionQuiz.style.display = "block";
    renderQuestion();
  };

  // 生成雷达图函数
  let radarChartInstance = null;
  function renderRadarChart(scores) {
    const ctx = document.getElementById("radarChart").getContext("2d");
    if (radarChartInstance) {
      radarChartInstance.destroy();
    }
    radarChartInstance = new Chart(ctx, {
      type: "radar",
      data: {
        labels: [
          "外向性", "情绪稳定性", "新奇寻求", "责任感", "冲动性",
          "焦虑水平", "自控力", "开放性", "亲和性", "尽责性",
          "抗压性", "未来规划"
        ],
        datasets: [{
          label: "标签得分",
          data: [
            scores.extraversion,
            scores.emotion_stability,
            scores.novelty_seek,
            scores.responsibility,
            scores.impulsivity,
            scores.anxiety,
            scores.self_control,
            scores.openness,
            scores.agreeableness,
            scores.conscientiousness,
            scores.stress_resilience,
            scores.future_planning
          ],
          fill: true,
          backgroundColor: "rgba(59, 130, 246, 0.3)",
          borderColor: "rgba(59, 130, 246, 0.8)",
          pointBackgroundColor: "rgba(59, 130, 246, 1)",
          pointBorderColor: "#fff",
          pointHoverBackgroundColor: "#fff",
          pointHoverBorderColor: "rgba(59, 130, 246,1)"
        }]
      },
      options: {
        scales: {
          r: {
            min: -3,
            max: 3,
            ticks: {
              stepSize: 1
            }
          }
        }
      }
    });

    // 简要解读
    document.getElementById("interpretation").textContent = generateInterpretation(scores);
  }

  // 生成简要解读文字
  function generateInterpretation(tags) {
    let lines = [];
    const labelNames = {
      extraversion: "外向性",
      emotion_stability: "情绪稳定性",
      novelty_seek: "新奇寻求",
      responsibility: "责任感",
      impulsivity: "冲动性",
      anxiety: "焦虑水平",
      self_control: "自控力",
      openness: "开放性",
      agreeableness: "亲和性",
      conscientiousness: "尽责性",
      stress_resilience: "抗压性",
      future_planning: "未来规划"
    };
    for (const tag in tags) {
      const val = tags[tag];
      if (val >= 2) lines.push(`${labelNames[tag]}：显著偏高`);
      else if (val <= -2) lines.push(`${labelNames[tag]}：显著偏低`);
      else lines.push(`${labelNames[tag]}：正常范围`);
    }
    return lines.join("\n");
  }

  // 页面加载默认开始
  renderQuestion();
</script>

</body>
</html>
