<!DOCTYPE html>
<html lang="zh">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>基因行为测验问卷</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body { font-family: "微软雅黑", sans-serif; padding: 20px; background: #f7f9fc; max-width: 720px; margin: auto; }
    h1, h2 { text-align: center; color: #222; }
    form { background: #fff; padding: 20px; border-radius: 12px; box-shadow: 0 0 12px rgba(0,0,0,0.1); }
    .question { margin-bottom: 18px; }
    .question p { font-weight: 600; color: #444; margin-bottom: 6px; }
    label { display: block; margin-bottom: 6px; cursor: pointer; color: #555; }
    input[type=radio] { margin-right: 8px; }
    #radar-container, #report { margin-top: 30px; background: #fff; padding: 20px; border-radius: 12px; box-shadow: 0 0 12px rgba(0,0,0,0.1); }
    #total-score { font-size: 1.3em; font-weight: bold; text-align: center; margin-bottom: 12px; color: #007acc; }
    ul { line-height: 1.6; padding-left: 20px; }
    li strong { color: #007acc; }
    #submit-btn { display: block; width: 100%; padding: 12px; font-size: 1.1em; background: #007acc; color: white; border: none; border-radius: 8px; cursor: pointer; margin-top: 10px; }
    #submit-btn:disabled { background: #ccc; cursor: not-allowed; }
  </style>
</head>
<body>

<h1>基因行为测验问卷</h1>

<div id="question-container"></div>

<div id="radar-container" style="display:none;">
  <h2>基因评分雷达图</h2>
  <canvas id="radarChart"></canvas>
</div>

<div id="report" style="display:none;">
  <h2>基础解析报告</h2>
  <p id="total-score"></p>
  <p id="summary-text"></p>
  <h2>深度解析报告</h2>
  <ul id="detail-list"></ul>
</div>

<script>
// 维度标签与基因对应
const dimensionLabels = {
  EmotionalStability: '情绪稳定性',
  Sociability: '社交性',
  RiskTolerance: '风险偏好',
  AttentionControl: '注意力控制',
  Impulsivity: '冲动性',
  StressResilience: '抗压能力'
};

// 12个问题与基因对应评分
const questions = [
  { q: "当你面对挫折时，你通常会？", options: [
    { text: "快速调整情绪，积极应对", scores: { EmotionalStability: 8, StressResilience: 7 } },
    { text: "感到沮丧，但可以逐渐恢复", scores: { EmotionalStability: 4, StressResilience: 5 } },
    { text: "长时间无法振作，情绪容易波动", scores: { EmotionalStability: 2, StressResilience: 3 } },
    { text: "寻求他人帮助，暂时逃避", scores: { Sociability: 6, EmotionalStability: 3 } }
  ] },
  { q: "在做决定时，你通常？", options: [
    { text: "仔细思考后做决定", scores: { Impulsivity: 3, AttentionControl: 7 } },
    { text: "快速做决定，倾向于立即行动", scores: { Impulsivity: 8, AttentionControl: 4 } },
    { text: "依赖他人的建议和意见", scores: { Sociability: 6, EmotionalStability: 5 } },
    { text: "反复琢磨，迟迟无法决定", scores: { EmotionalStability: 4, Impulsivity: 2 } }
  ] },
  { q: "你喜欢与陌生人交流吗？", options: [
    { text: "非常喜欢，感到很有趣", scores: { Sociability: 9 } },
    { text: "偶尔，取决于情境", scores: { Sociability: 5 } },
    { text: "不太喜欢，感到不自在", scores: { Sociability: 2 } },
    { text: "我倾向于避免交流", scores: { Sociability: 1 } }
  ] },
  { q: "你通常如何应对压力？", options: [
    { text: "通过锻炼或运动释放压力", scores: { StressResilience: 8, EmotionalStability: 6 } },
    { text: "倾诉给朋友或家人", scores: { Sociability: 7, StressResilience: 5 } },
    { text: "沉默不语，独自承担", scores: { EmotionalStability: 5, StressResilience: 4 } },
    { text: "逃避，避免面对压力源", scores: { StressResilience: 2, EmotionalStability: 3 } }
  ] },
  { q: "你通常怎样处理焦虑情绪？", options: [
    { text: "通过冥想或深呼吸放松自己", scores: { EmotionalStability: 8, StressResilience: 7 } },
    { text: "找朋友聊一聊，分散注意力", scores: { Sociability: 7, EmotionalStability: 5 } },
    { text: "坚持自己的方式，继续工作", scores: { Impulsivity: 6, AttentionControl: 6 } },
    { text: "消极情绪一直缠绕，难以走出困境", scores: { EmotionalStability: 3, StressResilience: 4 } }
  ] },
  { q: "你对待新任务的态度是？", options: [
    { text: "充满好奇，迅速开始行动", scores: { Impulsivity: 7, Sociability: 5 } },
    { text: "谨慎小心，先了解清楚再行动", scores: { EmotionalStability: 8, AttentionControl: 7 } },
    { text: "拖延，直到最后一分钟才开始做", scores: { StressResilience: 4, Impulsivity: 3 } },
    { text: "如果不感兴趣，就会直接放弃", scores: { Sociability: 4, StressResilience: 5 } }
  ] },
  { q: "你通常如何应对失败？", options: [
    { text: "总结经验，重新调整计划", scores: { EmotionalStability: 8, StressResilience: 6 } },
    { text: "失落很久，难以接受", scores: { EmotionalStability: 4, StressResilience: 3 } },
    { text: "向他人寻求安慰和鼓励", scores: { Sociability: 6, EmotionalStability: 5 } },
    { text: "放弃尝试，避免失败的情境", scores: { StressResilience: 2, EmotionalStability: 3 } }
  ] },
  { q: "你喜欢在团队中担任什么角色？", options: [
    { text: "领导角色，主导整个团队", scores: { Sociability: 9, Impulsivity: 7 } },
    { text: "执行者，听从领导安排", scores: { Sociability: 5, AttentionControl: 6 } },
    { text: "协调者，帮助大家沟通合作", scores: { Sociability: 8, EmotionalStability: 6 } },
    { text: "独立工作者，不喜欢干扰", scores: { Sociability: 3, StressResilience: 5 } }
  ] },
  { q: "你觉得自己在压力下能保持冷静吗？", options: [
    { text: "能保持冷静，寻找解决办法", scores: { StressResilience: 8, EmotionalStability: 7 } },
    { text: "偶尔能保持冷静，偶尔情绪失控", scores: { StressResilience: 5, EmotionalStability: 4 } },
    { text: "容易情绪化，难以冷静思考", scores: { StressResilience: 3, EmotionalStability: 2 } },
    { text: "完全失控，感到无法应对", scores: { StressResilience: 1, EmotionalStability: 1 } }
  ] },
  { q: "你在群体中的社交倾向如何？", options: [
    { text: "非常外向，喜欢成为焦点", scores: { Sociability: 9 } },
    { text: "较为内向，喜欢小范围交流", scores: { Sociability: 5 } },
    { text: "很少主动，通常处于旁观者", scores: { Sociability: 3 } },
    { text: "完全不喜欢社交，尽量避开人群", scores: { Sociability: 1 } }
  ] },
  { q: "你通常如何处理情绪波动？", options: [
    { text: "通过冥想、运动等方式平复情绪", scores: { EmotionalStability: 8, StressResilience: 7 } },
    { text: "与朋友交流，寻求安慰", scores: { Sociability: 6, EmotionalStability: 5 } },
    { text: "自己默默消化，时间过去后会好转", scores: { EmotionalStability: 4, StressResilience: 3 } },
    { text: "情绪波动大，容易失控", scores: { EmotionalStability: 2, StressResilience: 2 } }
  ] },
  { q: "你如何看待挑战性的任务？", options: [
    { text: "喜欢挑战，乐于迎接新任务", scores: { RiskTolerance: 8, Sociability: 6 } },
    { text: "有点害怕，但还是会尝试", scores: { RiskTolerance: 5, EmotionalStability: 4 } },
    { text: "不太喜欢，倾向于避免复杂任务", scores: { RiskTolerance: 3, StressResilience: 3 } },
    { text: "完全不喜欢，避免任何挑战", scores: { RiskTolerance: 1, EmotionalStability: 2 } }
  ] },
  { q: "你对于未来的规划是否清晰？", options: [
    { text: "非常清晰，已设定明确目标", scores: { AttentionControl: 8, EmotionalStability: 7 } },
    { text: "有大致方向，但没有明确的计划", scores: { AttentionControl: 5, Sociability: 4 } },
    { text: "没有具体计划，只是随遇而安", scores: { AttentionControl: 3, Sociability: 5 } },
    { text: "完全没有计划，甚至感到迷茫", scores: { AttentionControl: 2, EmotionalStability: 3 } }
  ] }
];

// 随机打乱数组
function shuffleArray(arr) {
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [arr[i], arr[j]] = [arr[j], arr[i]];
  }
}

// 随机打乱问题和选项顺序
shuffleArray(questions);
questions.forEach(question => shuffleArray(question.options));

let userAnswers = [];
let currentQuestionIndex = 0;

// 渲染问题
function renderQuestion() {
  const question = questions[currentQuestionIndex];
  const container = document.getElementById('question-container');
  container.innerHTML = `
    <div class="question">
      <p><strong>问题 ${currentQuestionIndex + 1}</strong>: ${question.q}</p>
      ${question.options.map((opt, idx) => `
        <label>
          <input type="radio" name="question" value="${idx}" required>
          ${opt.text}
        </label>
      `).join('')}
      <button id="submit-btn">提交</button>
    </div>
  `;

  // 绑定点击事件
  const submitBtn = document.getElementById('submit-btn');
  submitBtn.addEventListener('click', handleSubmit);
}

// 处理提交
function handleSubmit() {
  const selectedOption = document.querySelector('input[name="question"]:checked');
  if (!selectedOption) {
    alert('请先选择一个选项');
    return;
  }

  // 存储用户的选择
  const selectedIndex = parseInt(selectedOption.value);
  userAnswers.push(questions[currentQuestionIndex].options[selectedIndex].scores);

  // 判断是否为最后一个问题
  if (currentQuestionIndex < questions.length - 1) {
    currentQuestionIndex++;
    renderQuestion();  // 渲染下一个问题
  } else {
    displayResults();  // 显示最终结果
  }
}

// 显示结果
function displayResults() {
  const totalScores = userAnswers.reduce((acc, scoreObj) => {
    for (const key in scoreObj) {
      acc[key] = (acc[key] || 0) + scoreObj[key];
    }
    return acc;
  }, {});

  const totalScore = Object.values(totalScores).reduce((sum, score) => sum + score, 0) / Object.keys(totalScores).length;

  // 显示雷达图
  const ctx = document.getElementById('radarChart').getContext('2d');
  new Chart(ctx, {
    type: 'radar',
    data: {
      labels: Object.values(dimensionLabels),
      datasets: [{
        label: '基因评分',
        data: Object.values(totalScores),
        backgroundColor: 'rgba(255, 99, 132, 0.2)',
        borderColor: 'rgba(255, 99, 132, 1)',
        borderWidth: 1
      }]
    },
    options: {
      scales: {
        r: { beginAtZero: true, suggestedMin: 0, suggestedMax: 100 }
      }
    }
  });

  // 显示报告
  document.getElementById('radar-container').style.display = 'block';
  document.getElementById('report').style.display = 'block';
  document.getElementById('total-score').textContent = `综合总分：${totalScore.toFixed(1)} / 100`;

  let summaryText = '';
  if (totalScore > 70) {
    summaryText = '整体心理基因表现良好，具备较强的情绪调节和社交能力，适应环境和压力的能力较强。';
  } else if (totalScore > 40) {
    summaryText = '心理基因表现中等，有一定优势，但部分维度存在提升空间，建议关注情绪和注意力管理。';
  } else {
    summaryText = '心理基因表现偏低，可能易感情绪波动或注意力不集中，建议加强心理健康和抗压能力。';
  }
  document.getElementById('summary-text').textContent = summaryText;

  const detailsList = Object.keys(totalScores).map(key => {
    return `<li><strong>${dimensionLabels[key]}</strong>: ${totalScores[key]}</li>`;
  }).join('');
  document.getElementById('detail-list').innerHTML = detailsList;
}

// 初始化
renderQuestion();  // 初次渲染第一个问题
</script>

</body>
</html>
