<!DOCTYPE html>
<html lang="zh">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>心理六维度基因评分雷达图与解析报告</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
  body { max-width: 700px; margin: auto; font-family: Arial, sans-serif; padding: 20px; background: #f9f9f9; }
  h1, h2 { text-align: center; color: #333; }
  #radar-container { width: 100%; max-width: 600px; margin: 20px auto; background: #fff; padding: 20px; border-radius: 10px; box-shadow: 0 0 8px rgba(0,0,0,0.1); }
  #report { margin-top: 30px; background: #fff; padding: 20px; border-radius: 10px; box-shadow: 0 0 8px rgba(0,0,0,0.1); }
  .section { margin-bottom: 20px; }
  ul { line-height: 1.6; }
  li strong { color: #007acc; }
  #total-score { font-size: 1.2em; font-weight: bold; color: #444; text-align: center; margin-bottom: 15px; }
</style>
</head>
<body>

<h1>心理六维度基因评分雷达图</h1>
<div id="radar-container">
  <canvas id="radarChart"></canvas>
</div>

<div id="report">
  <div class="section" id="summary-report">
    <h2>基础解析报告</h2>
    <p id="total-score"></p>
    <p id="summary-text"></p>
  </div>
  <div class="section" id="detailed-report">
    <h2>深度解析报告</h2>
    <ul id="detail-list"></ul>
  </div>
</div>

<script>
  const geneScores = {
    EmotionalStability: 65,
    Sociability: 80,
    RiskTolerance: 45,
    AttentionControl: 70,
    Impulsivity: 30,
    StressResilience: 55
  };

  const labels = {
    EmotionalStability: '情绪稳定性',
    Sociability: '社交性',
    RiskTolerance: '风险偏好',
    AttentionControl: '注意力控制',
    Impulsivity: '冲动性',
    StressResilience: '抗压能力'
  };

  const dataLabels = Object.values(labels);
  const dataScores = Object.keys(geneScores).map(k => geneScores[k]);

  // 计算总分（算术平均）
  function calcTotalScore(scores) {
    const vals = Object.values(scores);
    const sum = vals.reduce((a,b) => a + b, 0);
    return (sum / vals.length).toFixed(1);
  }

  // 生成基础解析报告文本（基于总分）
  function generateSummary(scores) {
    const total = calcTotalScore(scores);
    let desc = '';
    if (total > 70) desc = '整体心理基因表现良好，具备较强的情绪调节和社交能力，适应环境和压力的能力较强。';
    else if (total > 40) desc = '心理基因表现中等，有一定优势，但部分维度存在提升空间，建议关注情绪和注意力管理。';
    else desc = '心理基因表现偏低，可能易感情绪波动或注意力不集中，建议加强心理健康和抗压能力。';
    return { total, desc };
  }

  // 生成深度解析报告（针对每维度）
  function generateDetails(scores) {
    const descriptions = {
      EmotionalStability: '情绪稳定性：调节情绪波动的能力，分数越高越稳定。',
      Sociability: '社交性：个体与他人交流的倾向和能力。',
      RiskTolerance: '风险偏好：面对不确定性和风险时的决策倾向。',
      AttentionControl: '注意力控制：集中和维持注意力的能力。',
      Impulsivity: '冲动性：行为冲动程度，分数越高冲动越低，越理性。',
      StressResilience: '抗压能力：面对压力时的心理适应和恢复能力。'
    };
    let lis = '';
    for (const key in scores) {
      const score = scores[key];
      let level = '';
      if (score >= 70) level = '高';
      else if (score >= 40) level = '中';
      else level = '低';
      lis += `<li><strong>${labels[key]}</strong> (${level})：${descriptions[key]} 当前得分：${score}</li>`;
    }
    return lis;
  }

  // 初始化雷达图
  const ctx = document.getElementById('radarChart').getContext('2d');
  const radarChart = new Chart(ctx, {
    type: 'radar',
    data: {
      labels: dataLabels,
      datasets: [{
        label: '心理维度评分',
        data: dataScores,
        backgroundColor: 'rgba(255, 99, 132, 0.25)',
        borderColor: 'rgba(255, 99, 132, 1)',
        borderWidth: 2,
        pointBackgroundColor: 'rgba(255, 99, 132, 1)'
      }]
    },
    options: {
      scales: {
        r: {
          suggestedMin: 0,
          suggestedMax: 100,
          ticks: { stepSize: 20 },
          pointLabels: { font: { size: 14 } }
        }
      },
      plugins: {
        legend: { display: false }
      },
      elements: {
        line: { tension: 0.3 }
      }
    }
  });

  // 渲染报告
  const summary = generateSummary(geneScores);
  document.getElementById('total-score').innerText = `综合总分：${summary.total} / 100`;
  document.getElementById('summary-text').innerText = summary.desc;
  document.getElementById('detail-list').innerHTML = generateDetails(geneScores);

</script>

</body>
</html>
