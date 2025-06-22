<!DOCTYPE html>
<html lang="zh">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>基因型行为测评</title>
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

<h1>基因型行为测评</h1>

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
// 假设每个基因的影响与心理维度的对应关系（模拟数据）
const geneProfiles = {
  5HTTLPR: { EmotionalStability: 8 },  // 情绪稳定性基因
  OXTR: { Sociability: 7 },  // 社交性基因
  DRD4: { Impulsivity: 6 },  // 冲动性基因
  COMT: { AttentionControl: 5 },  // 注意力控制基因
  MAOA: { StressResilience: 9 }  // 抗压能力基因
};

// 维度标签与基因对应
const dimensionLabels = {
  EmotionalStability: '情绪稳定性',
  Sociability: '社交性',
  Impulsivity: '冲动性',
  AttentionControl: '注意力控制',
  StressResilience: '抗压能力'
};

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

// 计算得分
function calculateScores(geneData) {
  const scores = { EmotionalStability: 0, Sociability: 0, Impulsivity: 0, AttentionControl: 0, StressResilience: 0 };

  for (const gene in geneData) {
    const influence = geneData[gene];
    for (const dimension in influence) {
      scores[dimension] += influence[dimension];
    }
  }

  return scores;
}

// 渲染报告
function displayResults() {
  const geneData = geneProfiles;
  const scores = calculateScores(geneData);
  
  const totalScore = Object.values(scores).reduce((sum, score) => sum + score, 0) / Object.keys(scores).length;

  // 显示雷达图
  const ctx = document.getElementById('radarChart').getContext('2d');
  new Chart(ctx, {
    type: 'radar',
    data: {
      labels: Object.values(dimensionLabels),
      datasets: [{
        label: '基因评分',
        data: Object.values(scores),
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

// 初始化显示结果
displayResults();
</script>

</body>
</html>
