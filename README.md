<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>因问卷与雷达图评分</title>
<style>
  body {
    max-width: 600px; margin: 20px auto; font-family: "微软雅黑", sans-serif;
    background: #f0f4f8; padding: 20px; color:#222;
  }
  h2, h3 { text-align: center; }
  label, select, input[type=number] {
    width: 100%; margin: 10px 0 15px; font-size: 16px;
  }
  select, input[type=number] {
    padding: 8px; border-radius: 6px; border: 1px solid #ccc;
  }
  button {
    width: 100%; padding: 12px; font-size: 16px; border: none;
    border-radius: 6px; cursor: pointer;
    color: white;
    background: #4338ca;
  }
  button:disabled {
    background: #a5b4fc; cursor: not-allowed;
  }
  .option {
    background: white; border: 1px solid #bbb; border-radius: 6px;
    padding: 10px; margin: 6px 0; cursor: pointer; user-select:none;
  }
  .option.selected {
    background: #4f46e5; color: white; border-color: #4338ca;
  }
  #promptOutput {
    white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
    background: #eee; padding: 10px; border-radius: 6px; font-family: monospace;
    user-select: all; /* 方便复制 */
  }
  #totalScoreText {
    text-align: center; font-weight: bold; margin: 10px 0;
    font-size: 18px;
  }
  canvas {
    display: block;
    margin: 0 auto 20px;
    max-width: 100%;
  }
  #progressText {
    font-size: 14px; color: #555; text-align: center; margin-top: 5px;
  }
</style>
</head>
<body>

<h2>因问卷 & 雷达图评分</h2>

<div id="sectionUserInfo">
  <label for="inputAge">年龄（不限）</label>
  <input type="number" id="inputAge" min="0" value="25" />
  <label for="selectGender">性别</label>
  <select id="selectGender">
    <option value="male">男性</option>
    <option value="female">女性</option>
  </select>
  <button id="btnStart">开始答题</button>
</div>

<div id="sectionQuiz" style="display:none;">
  <div id="questionText" class="question" style="font-weight:700; margin-bottom:10px;"></div>
  <div id="optionsContainer"></div>
  <button id="btnNext" disabled>下一题</button>
  <div id="progressText"></div>
</div>

<div id="sectionResult" style="display:none;">
  <h3>因风格评分雷达图</h3>
  <canvas id="radarChart" width="400" height="400"></canvas>
  <div id="totalScoreText"></div>
  <h3>因风格Prompt（点击可复制）</h3>
  <pre id="promptOutput" title="点击复制"></pre>
  <button id="btnRestart">重新开始</button>
</div>

<script>
  // 六个因维度（中文）
  const geneDimensions = [
    { key: "extraversion", label: "外向性" },
    { key: "emotion_stability", label: "情绪稳定性" },
    { key: "novelty_seek", label: "新奇寻求" },
    { key: "responsibility", label: "责任感" },
    { key: "self_control", label: "自控力" },
    { key: "openness", label: "开放性" }
  ];

  // 12题题库，每题4选项，对应六个因维度评分-2~2
  const questionPool = [
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
        { text: "非常紧张和焦虑", tags: { emotion_stability: -2, self_control: -1 } },
        { text: "有些不安", tags: { emotion_stability: -1, self_control: 0 } },
        { text: "情绪稳定", tags: { emotion_stability: 1, self_control: 1 } },
        { text: "非常冷静", tags: { emotion_stability: 2, self_control: 2 } }
      ]
    },
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
        { text: "经常拖延", tags: { responsibility: -2 } },
        { text: "偶尔拖延", tags: { responsibility: -1 } },
        { text: "大部分时间按时", tags: { responsibility: 1 } },
        { text: "总是按时完成", tags: { responsibility: 2 } }
      ]
    },
    {
      id: "Q5", text: "你做决定时是否容易冲动？",
      options: [
        { text: "完全不会冲动", tags: { self_control: 2 } },
        { text: "不太冲动", tags: { self_control: 1 } },
        { text: "有时冲动", tags: { self_control: -1 } },
        { text: "非常冲动", tags: { self_control: -2 } }
      ]
    },
    {
      id: "Q6", text: "你喜欢艺术和创造性活动吗？",
      options: [
        { text: "完全不喜欢", tags: { openness: -2 } },
        { text: "不太喜欢", tags: { openness: -1 } },
        { text: "比较喜欢", tags: { openness: 1 } },
        { text: "非常喜欢", tags: { openness: 2 } }
      ]
    },
    {
      id: "Q7", text: "你是否喜欢有条理地生活和工作？",
      options: [
        { text: "不喜欢", tags: { responsibility: -2, openness: -1 } },
        { text: "偶尔", tags: { responsibility: -1, openness: 0 } },
        { text: "经常", tags: { responsibility: 1, openness: 1 } },
        { text: "总是", tags: { responsibility: 2, openness: 2 } }
      ]
    },
    {
      id: "Q8", text: "你控制情绪的能力如何？",
      options: [
        { text: "很差", tags: { emotion_stability: -2, self_control: -2 } },
        { text: "一般", tags: { emotion_stability: -1, self_control: -1 } },
        { text: "较好", tags: { emotion_stability: 1, self_control: 1 } },
        { text: "非常好", tags: { emotion_stability: 2, self_control: 2 } }
      ]
    },
    {
      id: "Q9", text: "你喜欢探索未知？",
      options: [
        { text: "完全不喜欢", tags: { novelty_seek: -2 } },
        { text: "不太喜欢", tags: { novelty_seek: -1 } },
        { text: "比较喜欢", tags: { novelty_seek: 1 } },
        { text: "非常喜欢", tags: { novelty_seek: 2 } }
      ]
    },
    {
      id: "Q10", text: "你是否容易焦虑？",
      options: [
        { text: "从不", tags: { emotion_stability: 2 } },
        { text: "偶尔", tags: { emotion_stability: 1 } },
        { text: "有时", tags: { emotion_stability: -1 } },
        { text: "经常", tags: { emotion_stability: -2 } }
      ]
    },
    {
      id: "Q11", text: "你待人是否友好与合作？",
      options: [
        { text: "不太友好", tags: { self_control: -2 } },
        { text: "有时冷淡", tags: { self_control: -1 } },
        { text: "比较友好", tags: { self_control: 1 } },
        { text: "非常友好", tags: { self_control: 2 } }
      ]
    },
    {
      id: "Q12", text: "你愿意为他人着想吗？",
      options: [
        { text: "完全不愿", tags: { self_control: -2 } },
        { text: "偶尔愿意", tags: { self_control: -1 } },
        { text: "比较愿意", tags: { self_control: 1 } },
        { text: "总是愿意", tags: { self_control: 2 } }
      ]
    }
  ];

  // 计算分数
  let tagScores = {};
  let askedIds = new Set();
  let questionCount = 0;
  const maxQuestions = 12;
  let age = 25;
  let gender = "male";

  const sectionUserInfo = document.getElementById("sectionUserInfo");
  const sectionQuiz = document.getElementById("sectionQuiz");
  const sectionResult = document.getElementById("sectionResult");
  const qTextEl = document.getElementById("questionText");
  const optionsEl = document.getElementById("optionsContainer");
  const btnNext = document.getElementById("btnNext");
  const progressText = document.getElementById("progressText");
  const totalScoreText = document.getElementById("totalScoreText");
  const promptOutput = document.getElementById("promptOutput");

  // 年龄与性别的调整
  function adjustTags(tags) {
    const adjusted = { ...tags };

    if (age < 30) {
      adjusted.novelty_seek *= 1.2;
      adjusted.openness *= 1.2;
    }

    if (gender === "female") {
      adjusted.emotion_stability *= 1.1;
      adjusted.self_control *= 1.1;
    }

    return adjusted;
  }

  // 渲染问题
  function renderQuestion(q) {
    qTextEl.textContent = q.text;
    optionsEl.innerHTML = '';
    for (let i = 0; i < q.options.length; i++) {
      const option = document.createElement("div");
      option.className = "option";
      option.textContent = q.options[i].text;
      option.onclick = () => selectOption(i, q);
      optionsEl.appendChild(option);
    }
    btnNext.disabled = true;
    progressText.textContent = `第 ${questionCount + 1} / ${maxQuestions} 题`;
  }

  // 选择选项
  function selectOption(index, q) {
    const selectedOption = q.options[index];
    for (let key in selectedOption.tags) {
      tagScores[key] = (tagScores[key] || 0) + selectedOption.tags[key];
    }
    askedIds.add(q.id);
    questionCount++;
    if (questionCount < maxQuestions) {
      const nextQ = questionPool[Math.floor(Math.random() * questionPool.length)];
      renderQuestion(nextQ);
    } else {
      showResult();
    }
    btnNext.disabled = false;
  }

  // 计算并显示结果
  function showResult() {
    sectionQuiz.style.display = "none";
    sectionResult.style.display = "block";

    // 计算得分
    const normalizedScores = geneDimensions.map(dim => {
      const raw = tagScores[dim.key] || 0;
      let score = (raw + 8) / 16 * 100; // 范围从0到100
      score = Math.min(100, Math.max(0, score));
      return score;
    });

    // 计算总分（平均）
    const totalScore = (normalizedScores.reduce((sum, score) => sum + score, 0) / normalizedScores.length).toFixed(1);

    totalScoreText.textContent = `总分: ${totalScore}`;

    // 绘制雷达图
    drawRadarChart(normalizedScores);

    // 生成prompt
    let prompt = `年龄: ${age}, 性别: ${gender === "male" ? "男性" : "女性"}\n`;
    normalizedScores.forEach((score, idx) => {
      prompt += `${geneDimensions[idx].label}: ${score.toFixed(1)} / 100\n`;
    });

    promptOutput.textContent = prompt.trim();
  }

  // 绘制雷达图
  function drawRadarChart(scores) {
    const canvas = document.getElementById("radarChart");
    const ctx = canvas.getContext("2d");
    const radius = 150;
    const angleStep = (Math.PI * 2) / scores.length;
    const cx = canvas.width / 2;
    const cy = canvas.height / 2;

    // 清空画布
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // 画六边形（外圈）
    ctx.beginPath();
    for (let i = 0; i < scores.length; i++) {
      const r = scores[i] * radius / 100;
      const x = cx + r * Math.sin(i * angleStep);
      const y = cy - r * Math.cos(i * angleStep);
      if (i === 0) ctx.moveTo(x, y);
      else ctx.lineTo(x, y);
    }
    ctx.closePath();
    ctx.fillStyle = "rgba(75, 192, 192, 0.2)";
    ctx.strokeStyle = "rgba(75, 192, 192, 1)";
    ctx.fill();
    ctx.stroke();

    // 画节点
    ctx.fillStyle = "rgba(75, 192, 192, 1)";
    for (let i = 0; i < scores.length; i++) {
      const r = scores[i] * radius / 100;
      const x = cx + r * Math.sin(i * angleStep);
      const y = cy - r * Math.cos(i * angleStep);
      ctx.beginPath();
      ctx.arc(x, y, 5, 0, 2 * Math.PI);
      ctx.fill();
    }
  }

  // 重新开始
  document.getElementById("btnRestart").onclick = () => {
    tagScores = {};
    askedIds.clear();
    questionCount = 0;
    sectionResult.style.display = "none";
    sectionUserInfo.style.display = "block";
  };

  // 开始答题
  document.getElementById("btnStart").onclick = () => {
    age = parseInt(document.getElementById("inputAge").value);
    gender = document.getElementById("selectGender").value;
    sectionUserInfo.style.display = "none";
    sectionQuiz.style.display = "block";
    const firstQ = questionPool[Math.floor(Math.random() * questionPool.length)];
    renderQuestion(firstQ);
  };
</script>

</body>
</html>
