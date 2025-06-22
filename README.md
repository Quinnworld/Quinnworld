<!DOCTYPE html>
<html lang="zh">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>心理六维度基因行为测验与解析</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
  body { max-width: 720px; margin: auto; font-family: "微软雅黑", sans-serif; padding: 20px; background: #f7f9fc; }
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

<h1>心理六维度基因相关行为测验</h1>

<form id="quiz-form" aria-label="心理六维度基因行为测验">
  <!-- 题目自动生成 -->
</form>

<div id="radar-container" style="display:none;">
  <h2>心理六维度基因评分雷达图</h2>
  <canvas id="radarChart" aria-label="心理六维度基因评分雷达图" role="img"></canvas>
</div>

<div id="report" style="display:none;">
  <h2>基础解析报告</h2>
  <p id="total-score" aria-live="polite"></p>
  <p id="summary-text"></p>
  <h2>深度解析报告</h2>
  <ul id="detail-list"></ul>
</div>

<script>
  const dimensionLabels = {
    EmotionalStability: '情绪稳定性',
    Sociability: '社交性',
    RiskTolerance: '风险偏好',
    AttentionControl: '注意力控制',
    Impulsivity: '冲动性',
    StressResilience: '抗压能力'
  };

  // 12题，每题4选项，权重分0~10（示例）
  const questions = [
    {
      q: "当面对突发事件时，你通常的反应是？",
      options: [
        { text: "冷静分析，逐步解决", scores: {EmotionalStability:8, AttentionControl:7, StressResilience:7} },
        { text: "立刻采取行动，快速应对", scores: {RiskTolerance:8, Impulsivity:3, StressResilience:5} },
        { text: "依赖他人建议和帮助", scores: {Sociability:7, EmotionalStability:5, StressResilience:4} },
        { text: "容易紧张和慌乱", scores: {EmotionalStability:2, AttentionControl:3, StressResilience:2} }
      ]
    },
    {
      q: "你更倾向于怎样安排闲暇时间？",
      options: [
        { text: "参加社交活动，与朋友相聚", scores: {Sociability:9} },
        { text: "独自读书或深度思考", scores: {AttentionControl:8, EmotionalStability:6} },
        { text: "尝试冒险或新鲜事物", scores: {RiskTolerance:9, Impulsivity:5} },
        { text: "放松休息，避免压力", scores: {StressResilience:7, EmotionalStability:5} }
      ]
    },
    {
      q: "遇到压力时，你通常如何调整？",
      options: [
        { text: "通过运动或活动释放", scores: {StressResilience:8, AttentionControl:6} },
        { text: "寻求朋友倾诉支持", scores: {Sociability:8, EmotionalStability:6} },
        { text: "尽量避免面对压力源", scores: {RiskTolerance:3, EmotionalStability:4} },
        { text: "容易焦虑，难以自控", scores: {Impulsivity:2, EmotionalStability:2} }
      ]
    },
    {
      q: "面对新环境，你的适应速度如何？",
      options: [
        { text: "很快适应，充满兴趣", scores: {RiskTolerance:8, Sociability:7} },
        { text: "需要时间观察和习惯", scores: {AttentionControl:7, EmotionalStability:6} },
        { text: "感到不安，较难适应", scores: {EmotionalStability:3, StressResilience:3} },
        { text: "尽量避免变化，喜欢稳定", scores: {RiskTolerance:2, Impulsivity:7} }
      ]
    },
    {
      q: "你认为自己做决定时？",
      options: [
        { text: "细致分析，慎重决策", scores: {AttentionControl:8, Impulsivity:8} },
        { text: "依赖直觉和冲动", scores: {Impulsivity:2, RiskTolerance:7} },
        { text: "请教他人意见", scores: {Sociability:7} },
        { text: "拖延犹豫，难以决定", scores: {EmotionalStability:4, StressResilience:3} }
      ]
    },
    {
      q: "你喜欢参与哪类活动？",
      options: [
        { text: "户外运动和挑战", scores: {RiskTolerance:8} },
        { text: "读书、写作或研究", scores: {AttentionControl:9} },
        { text: "社交聚会和团队活动", scores: {Sociability:9} },
        { text: "安静独处和放松", scores: {EmotionalStability:7, StressResilience:6} }
      ]
    },
    {
      q: "你如何看待风险？",
      options: [
        { text: "愿意冒险，寻找机会", scores: {RiskTolerance:9, Impulsivity:6} },
        { text: "较为保守，喜欢安全", scores: {RiskTolerance:3, EmotionalStability:7} },
        { text: "看情况，灵活应对", scores: {AttentionControl:7, StressResilience:7} },
        { text: "通常避免风险", scores: {RiskTolerance:2, Sociability:5} }
      ]
    },
    {
      q: "当别人批评你时，你通常？",
      options: [
        { text: "虚心接受，努力改进", scores: {EmotionalStability:8, StressResilience:7} },
        { text: "情绪波动较大", scores: {EmotionalStability:3, Impulsivity:3} },
        { text: "忽视批评，不在意", scores: {RiskTolerance:7, Impulsivity:5} },
        { text: "寻求朋友安慰", scores: {Sociability:8} }
      ]
    },
    {
      q: "你处理多任务时？",
      options: [
        { text: "有条不紊，效率高", scores: {AttentionControl:9, EmotionalStability:7} },
        { text: "容易分心，效率低", scores: {AttentionControl:3, Impulsivity:4} },
        { text: "灵活切换，适应快", scores: {RiskTolerance:7, StressResilience:7} },
        { text: "避免多任务，专注单一", scores: {AttentionControl:6} }
      ]
    },
    {
      q: "你通常如何做计划？",
      options: [
        { text: "提前详细规划", scores: {AttentionControl:9, Impulsivity:8} },
        { text: "大致方向，灵活调整", scores: {RiskTolerance:7, StressResilience:7} },
        { text: "随机应变，不常规划", scores: {Impulsivity:3, Sociability:5} },
        { text: "缺乏计划，容易拖延", scores: {EmotionalStability:4, AttentionControl:3} }
      ]
    },
    {
      q: "你在团队中的角色通常是？",
      options: [
        { text: "组织协调者", scores: {Sociability:9, AttentionControl:7} },
        { text: "创新突破者", scores: {RiskTolerance:8, Impulsivity:6} },
        { text: "执行落实者", scores: {AttentionControl:8, EmotionalStability:7} },
        { text: "旁观支持者", scores: {EmotionalStability:5, Sociability:6} }
      ]
    },
    {
      q: "你对未来的态度是？",
      options: [
        { text: "充满信心和期待", scores: {EmotionalStability:8, RiskTolerance:7} },
        { text: "谨慎规划，稳步前进", scores: {AttentionControl:8, StressResilience:7} },
        { text: "焦虑担忧，容易犹豫", scores: {EmotionalStability:3, Impulsivity:3} },
        { text: "随遇而安，不刻意计划", scores: {Sociability:6, RiskTolerance:5} }
      ]
    }
  ];

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

  const submitBtn = document.createElement('button');
  submitBtn.id = 'submit-btn';
  submitBtn.type = 'button';
  submitBtn.textContent = '提交测验，查看结果';
  submitBtn.disabled = true;
  form.appendChild(submitBtn);

  // 监听答题完成情况，启用按钮
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
      const
