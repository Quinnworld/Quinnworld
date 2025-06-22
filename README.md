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
    .payment-btn { display: block; width: 100%; padding: 12px; font-size: 1.1em; background: #ff9900; color: white; border: none; border-radius: 8px; cursor: pointer; margin-top: 20px; }
    .payment-btn:disabled { background: #ccc; cursor: not-allowed; }
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
  <div id="depth-report-content" style="display:none;">
    <ul id="detail-list"></ul>
    <h3>您的动物象征：</h3>
    <p id="animal-symbol"></p>
    <h3>发展建议：</h3>
    <p id="development-suggestion"></p>
  </div>
  <button id="payment-btn" class="payment-btn">解锁深度发展报告（￥10）</button>
  <p id="payment-status" style="text-align:center; margin-top:10px;"></p>
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
  { q: "你喜欢在团队中担任什么角色？", options: [
    { text: "领导角色，主导整个团队", scores: { Sociability: 9, Impulsivity: 7 } },
    { text: "执行者，听从领导安排", scores: { Sociability: 5, AttentionControl: 6 } },
    { text: "协调者，帮助大家沟通合作", scores: { Sociability: 8, EmotionalStability: 6 } },
    { text: "独立工作者，不喜欢干扰", scores: { Sociability: 3, StressResilience: 5 } }
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

// 动物匹配逻辑
const animalProfiles = {
  lion: {
    name: "狮子",
    suggestion: "作为狮子，您具备领导力与高度的社交能力，但可能在压力面前容易感到焦虑。建议您通过冥想、运动来增强情绪稳定性。"
  },
  owl: {
    name: "猫头鹰",
    suggestion: "猫头鹰代表智慧与深思熟虑，您擅长规划和决策，但有时可能过于内向。可以尝试更多的社交互动来提升自己的人际关系。"
  },
  dolphin: {
    name: "海豚",
    suggestion: "海豚象征着高社交性和情绪调节能力。您与他人的互动十分自然，但在应对压力时可能有些脆弱。加强抗压训练将有助于您实现更好的自我。"
  },
  turtle: {
    name: "乌龟",
    suggestion: "乌龟象征着稳重与耐性，您在面临复杂任务时展现出谨慎的态度，但有时可能过于保守。建议您提高自己的风险容忍度，适当挑战自己。"
  }
};

// 随机分配动物
function assignAnimal(totalScore) {
  if (totalScore > 70) return animalProfiles.lion;
  if (totalScore > 50) return animalProfiles.dolphin;
  if (totalScore > 30) return animalProfiles.owl;
  return animalProfiles.turtle;
}

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

  // 动物匹配
  const animal = assignAnimal(totalScore);
  document.getElementById('animal-symbol').textContent = `您的动物象征是：${animal.name}`;
  document.getElementById('development-suggestion').textContent = animal.suggestion;

  // 解锁深度发展报告
  if (totalScore > 60) {
    document.getElementById('payment-btn').disabled = false;
  }
}

// 支付解锁深度报告
document.getElementById('payment-btn').addEventListener('click', function() {
  // 模拟支付过程
  const paymentStatus = document.getElementById('payment-status');
  paymentStatus.textContent = '支付处理中...';

  setTimeout(function() {
    // 假设支付成功
    paymentStatus.textContent = '支付成功，您已解锁深度发展报告！';
    document.getElementById('depth-report-content').style.display = 'block';  // 解锁深度报告内容
  }, 2000);  // 模拟2秒支付过程
});

// 初始化
renderQuestion();  // 初次渲染第一个问题
</script>

</body>
</html>
