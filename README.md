<!DOCTYPE html>
<html lang="zh">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>心理六维度基因评分雷达图与解析报告</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
  body { max-width: 700px; margin: auto; font-family: Arial, sans-serif; padding: 20px; }
  h1, h2 { text-align: center; }
  #radar-container { width: 100%; max-width: 600px; margin: 0 auto; }
  #report { margin-top: 30px; }
  .section { margin-bottom: 20px; }
</style>
</head>
<body>

<h1>心理六维度基因评分雷达图</h1>
<div id="radar-container">
  <canvas id="radarChart"></canvas>
</div>

<div id="report">
  <div class="section" id="summary-report">
    <h2>初步解析报告</h2>
    <p id="summary-text"></p>
  </div>
  <div class="section" id="detailed-report">
    <h2>基础解析报告</h2>
    <ul id="detail-list"></ul>
  </div>
</div>

<script>
  // 示例心理六维度分数 0~100
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
  const dataScores = Object.keys(geneScores).map(key => geneScores[key]);

  // 生成雷达图
  const ctx = document.getElementById('radarChart').getContext('2d');
  const radarChart = new Chart(ctx, {
    type: 'radar',
    data: {
      labels: dataLabels,
      datasets: [{
        label: '心理维度评分',
        data: dataScores,
        backgroundColor: 'rgba(255, 99, 132, 0.2)',
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
          ticks: { stepSize: 20 }
        }
      },
      plugins: { legend: { display: false } }
    }
  });

  // 初步解析报告
  function generateSummary(scores) {
    const avg = Object.values(scores).reduce((a,b) => a + b, 0) / dataLabels.length;
    if(avg > 70) return '整体心理基因表现较为均衡，具备良好的情绪调节和社交能力。';
    if(avg > 40) return '心理基因表现中等，部分维度存在提升空间，建议关注情绪与注意力管理。';
    return '心理基因评分偏低，可能易感情绪波动或注意力不集中，建议加强心理健康维护。';
  }

  // 细化解析
  function generateDetails(scores) {
    const descriptions = {
      EmotionalStability: '情绪稳定性：调节情绪波动的能力，分数越高越稳定。',
      Sociability: '社交性：与他人交往的倾向和能力。',
      RiskTolerance: '风险偏好：面对不确定性时的决策风格。',
      AttentionControl: '注意力控制：集中和保持注意力的能力。',
      Impulsivity: '冲动性：控制冲动行为的能力，分数越高冲动越低。',
      StressResilience: '抗压能力：面对压力的适应和恢复力。'
    };
    let lis = '';
    for(let key in scores){
      const score = scores[key];
      let level = score >= 70 ? '高' : (score >= 40 ? '中' : '低');
      lis += `<li><strong>${labels[key]}</strong> (${level})：${descriptions[key]} 当前得分：${score}</li>`;
    }
    return lis;
  }

  // 渲染报告
  document.getElementById('summary-text').innerText = generateSummary(geneScores);
  document.getElementById('detail-list').innerHTML = generateDetails(geneScores);

</script>

</body>
</html>
