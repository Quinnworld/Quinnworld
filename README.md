<!DOCTYPE html>
<html lang="zh">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>心理六维度基因行为测验</title>
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

<h1>心理六维度基因行为测验</h1>

<form id="quiz-form" aria-label="心理六维度基因行为测验">
  <!-- 题目动态生成 -->
</form>

<div id="radar-container" style="display:none;">
  <h2>心理六维度基因评分雷达图</h2>
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
// 维度标签
const dimensionLabels = {
  EmotionalStability: '情绪稳定性',
  Sociability: '社交性',
  RiskTolerance: '风险偏好',
  AttentionControl: '注意力控制',
  Impulsivity: '冲动性',
  StressResilience: '抗压能力'
};

// 12题每题4选项，每个选项包含得分
const questions = [
  { q: "当面对突发事件时，你通常的反应是？", options: [
    { text: "冷静分析，逐步解决", scores: { EmotionalStability: 8, AttentionControl: 7, StressResilience: 7 } },
    { text: "立刻采取行动，快速应对", scores: { RiskTolerance: 8, Impulsivity: 3, StressResilience: 5 } },
    { text: "依赖他人建议和帮助", scores: { Sociability: 7, EmotionalStability: 5, StressResilience: 4 } },
    { text: "容易紧张和慌乱", scores: { EmotionalStability: 2, AttentionControl: 3, StressResilience: 2 } }
  ] },
  { q: "你更倾向于怎样安排闲暇时间？", options: [
    { text: "参加社交活动，与朋友相聚", scores: { Sociability: 9 } },
    { text: "独自读书或深度思考", scores: { AttentionControl: 8, EmotionalStability: 6 } },
    { text: "尝试冒险或新鲜事物", scores: { RiskTolerance: 9, Impulsivity: 5 } },
    { text: "放松休息，避免压力", scores: { StressResilience: 7, EmotionalStability: 5 } }
  ] },
  { q: "遇到压力时，你通常如何调整？", options: [
    { text: "通过运动或活动释放", scores: { StressResilience: 8, AttentionControl: 6 } },
    { text: "寻求朋友倾诉支持", scores: { Sociability: 8, EmotionalStability: 6 } },
    { text: "尽量避免面对压力源", scores: { RiskTolerance: 3, EmotionalStability: 4 } },
    { text: "容易焦虑，难以自控", scores: { Impulsivity: 2, EmotionalStability: 2 } }
  ] },
  // 此处省略了其他题目，实际可以继续添加...
];

// 渲染题目
const form = document.getElementById('quiz-form');
questions.forEach((item, i) => {
  const div = document.createElement('div');
  div.className = 'question';
  div.innerHTML = `<p>Q${i + 1}. ${item.q}</p>` + item.options.map((opt, j) => `
    <label>
      <input type="radio" name="q${i}" value="${j}" required>
      ${opt.text}
    </label>
  `).join('');
  form.appendChild(div);
});

// 提交按钮
const submitBtn = document.getElementById('submit-btn');
form.addEventListener('change', () => {
  const answeredCount = [...form.elements].filter(el => el.checked && el.type === "radio").length;
  submitBtn.disabled = answeredCount !== questions.length;
});

// 计算分数
function calcScores() {
  const scores = {
    EmotionalStability: 0,
    Sociability: 0,
    RiskTolerance: 0,
    AttentionControl: 0,
    Impulsivity: 0,
    StressResilience: 0
  };
  for (let i = 0; i < questions.length; i++) {
    const selected = form.querySelector(`input[name=q${i}]:checked`);
    if (!selected) continue;
    const optionIndex = Number(selected.value);
    const optionScores = questions[i].options[optionIndex].scores;
    for (const key in optionScores) {
      if (scores.hasOwnProperty(key)) {
        scores[key] += optionScores[key];
      }
    }
  }
  return scores;
}

// 计算总分
function calcTotalScore(scores) {
  const vals = Object.values(scores);
  const sum = vals.reduce((a, b) => a + b, 0);
  return (sum / vals.length).toFixed(1);
}

// 生成报告
submitBtn.addEventListener('click', () => {
  const scores = calcScores();

  // 更新报告显示
  const totalScore = calcTotalScore(scores);
  document.getElementById('total-score').innerText = `综合总分：${totalScore} / 100`;

  // 生成报告摘要
  let summaryText = '';
  if (totalScore > 70) {
    summaryText = '整体心理基因表现良好，具备较强的情绪调节和社交能力，适应环境和压力的能力较强。';
  } else if (totalScore > 40) {
    summaryText = '心理基因表现中等，有一定优势，但部分维度存在提升空间，建议关注情绪和注意力管理。';
  } else {
    summaryText = '心理基因表现偏低，可能易感情绪波动或注意力不集中，建议加强心理健康和抗压能力。';
  }
  document.getElementById('summary-text').innerText = summaryText;

  // 深度报告
  const detailsList = Object.keys(scores).map(key => {
    return `<li><strong>${dimensionLabels[key]}</strong>: ${scores[key]}</li>`;
  }).join('');
  document.getElementById('detail-list').innerHTML = detailsList;

  // 显示雷达图
  const ctx = document.getElementById('radarChart').getContext('2d');
  const radarChart = new Chart(ctx, {
    type: 'radar',
    data: {
      labels: Object.values(dimensionLabels),
      datasets: [{
        label: '心理六维度基因评分',
        data: Object.values(scores),
        backgroundColor: 'rgba(255, 99, 132, 0.2)',
        borderColor: 'rgba(255, 99, 132, 1)',
        borderWidth: 1
      }]
    },
    options: {
      scales: {
        r: {
          beginAtZero: true,
          suggestedMin: 0,
          suggestedMax: 100
        }
      }
    }
  });

  // 显示结果
  document.getElementById('radar-container').style.display = 'block';
  document.getElementById('report').style.display = 'block';
});
</script>

</body>
</html>
