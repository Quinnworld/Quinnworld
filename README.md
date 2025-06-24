<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>虚拟基因人格画像问卷（含付费解锁）</title>
  <style>
    html, body {
      height: 100%;
      margin: 0; padding: 0;
      background: #f5f7fa;
      font-family: "微软雅黑", sans-serif;
      display: flex;
      justify-content: center;
      align-items: flex-start;
      padding: 30px 10px;
      box-sizing: border-box;
    }
    #mainbox {
      background: #fff;
      border-radius: 18px;
      box-shadow: 0 6px 24px #cdd2f1;
      width: 480px;
      max-width: 98vw;
      padding: 36px 24px 24px 24px;
      display: flex;
      flex-direction: column;
      align-items: center;
    }
    h2, h3 {
      color: #2b2f4a;
      text-align: center;
      margin: 8px 0 24px;
      font-weight: 600;
      letter-spacing: 1.5px;
    }
    .question {
      width: 100%;
      margin-bottom: 18px;
    }
    .question-text {
      font-weight: 700;
      margin-bottom: 10px;
      font-size: 17px;
      color: #222;
    }
    .question-img {
      width: 100%;
      height: auto;
      border-radius: 8px;
      margin-bottom: 15px;
    }
    .option {
      background: #f3f3fa;
      border: 1.5px solid #c7d2fe;
      border-radius: 8px;
      padding: 13px 11px;
      margin: 8px 0;
      font-size: 16px;
      cursor: pointer;
      user-select: none;
      transition: all 0.15s ease;
      position: relative;
    }
    .option:hover {
      background: #d1d8fd;
    }
    .option.selected {
      background: linear-gradient(90deg,#818cf8 70%,#60a5fa 100%);
      color: white;
      border-color: #6366f1;
      font-weight: bold;
      box-shadow: 0 2px 8px #dbeafe;
    }
    #btnSubmit, #btnRestart {
      margin-top: 18px;
      padding: 14px;
      font-size: 18px;
      font-weight: 600;
      border-radius: 8px;
      border: none;
      background: linear-gradient(90deg,#6366f1,#60a5fa 70%);
      color: white;
      cursor: pointer;
      width: 100%;
      box-shadow: 0 2px 8px #dbeafe;
      transition: background 0.2s ease;
    }
    #btnSubmit:disabled {
      background: #dbeafe;
      cursor: not-allowed;
      color: #90a4c6;
    }
    #resultSection {
      display: none;
      width: 100%;
      margin-top: 24px;
    }
    #promptOutput {
      background: #f4f6fb;
      border-radius: 8px;
      padding: 16px;
      white-space: pre-wrap;
      font-size: 15.6px;
      line-height: 1.4;
      color: #333;
      max-height: 200px;
      overflow-y: auto;
    }
    #paywall {
      margin-top: 28px;
      background: #fff4f4;
      border: 1px solid #ffd6d6;
      border-radius: 8px;
      padding: 20px;
      text-align: center;
      color: #c53030;
      font-weight: 600;
    }
    #paywall img {
      margin-top: 12px;
      width: 220px;
      user-select: none;
    }
    .radar-chart {
      width: 100%;
      height: 300px;
    }
  </style>
</head>
<body>
  <div id="mainbox">
    <h2>虚拟基因人格画像问卷</h2>
    <form id="quizForm">
      <!-- 12题问卷 -->
    </form>
    <button id="btnSubmit" disabled>提交并生成AI形象描述</button>

    <section id="resultSection">
      <h3>AI形象Prompt（基础版）</h3>
      <pre id="promptOutput">——请先完成问卷——</pre>
      <canvas id="radarChart" class="radar-chart"></canvas>

      <div id="paywall">
        <div>扫描下方二维码支付 ¥9.9 元，解锁<span style="color:#822;">深度人格分析报告完整版</span></div>
        <img src="https://pplx-res.cloudinary.com/image/upload/v1750655754/user_uploads/19241329/71824a55-7ec0-40e9-af8e-403b5e367882/mm_facetoface_collect_qrcode_1750513571858.jpg" alt="支付二维码" />
      </div>

      <button id="btnRestart">重新开始答题</button>
    </section>
  </div>

<script>
  // 12题问卷数据，含选项和对应多维雷达评分（仅示范核心）
  const questions = [
    {
      text: "你喜欢什么样的户外活动？",
      img: "https://example.com/beach.jpg", // 添加图片URL
      options: [
        { text: "海边日光浴", radar: { pigment:2, advent:1, happiness:1 }},
        { text: "森林徒步", radar: { bone:1, stable:1, stamina:1 }},
        { text: "极限运动", radar: { body:1, advent:2, stamina:2 }},
        { text: "更喜欢室内", radar: { pigment:-1, body:-1, stable:1, create:1 }}
      ]
    },
    // 其他问题略...
  ];

  // 多维雷达维度
  const radarDims = [
    {key:"pigment", name:"色素特征"},
    {key:"bone", name:"骨相立体"},
    {key:"body", name:"体型"},
    {key:"hair", name:"毛发特征"},
    {key:"advent", name:"冒险/外倾"},
    {key:"stable", name:"情绪稳定"},
    {key:"social", name:"社交能力"},
    {key:"self", name:"自律/意志"},
    {key:"create", name:"创造力"},
    {key:"empathy", name:"共情能力"},
    {key:"happiness", name:"主观幸福感"},
    {key:"stamina", name:"耐力/活力"}
  ];

  // 用户选择存储
  const answers = new Array(questions.length).fill(null);

  const quizForm = document.getElementById("quizForm");
  const btnSubmit = document.getElementById("btnSubmit");
  const promptOutput = document.getElementById("promptOutput");
  const resultSection = document.getElementById("resultSection");
  const btnRestart = document.getElementById("btnRestart");
  const radarChart = document.getElementById("radarChart");

  // 渲染所有题目
  function renderQuestions(){
    quizForm.innerHTML = '';
    questions.forEach((q,i) => {
      const qDiv = document.createElement('div');
      qDiv.className = 'question';
      const qText = document.createElement('div');
      qText.className = 'question-text';
      qText.textContent = `Q${i+1}. ${q.text}`;
      qDiv.appendChild(qText);

      if (q.img) {
        const imgElement = document.createElement('img');
        imgElement.src = q.img;
        imgElement.className = 'question-img';
        qDiv.appendChild(imgElement);
      }

      q.options.forEach((opt,j) => {
        const optDiv = document.createElement('div');
        optDiv.className = 'option';
        optDiv.textContent = opt.text;
        if(answers[i] === j) optDiv.classList.add('selected');
        optDiv.onclick = () => {
          answers[i] = j;
          updateSelection(i);
          checkAllAnswered();
        };
        qDiv.appendChild(optDiv);
      });

      quizForm.appendChild(qDiv);
    });
  }

  // 更新某题选项样式
  function updateSelection(questionIndex){
    const questionDiv = quizForm.children[questionIndex];
    Array.from(questionDiv.querySelectorAll('.option')).forEach((el,optIndex) => {
      el.classList.toggle('selected', optIndex === answers[questionIndex]);
    });
  }

  // 检查是否所有题目都已答，启用提交按钮
  function checkAllAnswered(){
    const allAnswered = answers.every(a => a !== null);
    btnSubmit.disabled = !allAnswered;
  }

  // 生成AI Prompt描述
  function genPrompt(radar){
    let parts = [];
    // 根据雷达分数生成描述（略）
    return parts.join(" ");
  }

  // 计算并绘制雷达图
  function drawRadarChart(radarScore) {
    const ctx = radarChart.getContext('2d');
    const data = {
      labels: radarDims.map(dim => dim.name),
      datasets: [{
        label: '个人特征',
        data: radarDims.map(dim => radarScore[dim.key]),
        backgroundColor: 'rgba(99, 102, 241, 0.2)',
        borderColor: 'rgba(99, 102, 241, 1)',
        borderWidth: 1
      }]
    };
    new Chart(ctx, {
      type: 'radar',
      data: data,
      options: {
        scales: {
          r: {
            min: -4,
            max: 4
          }
        }
      }
    });
  }

  // 提交事件
  btnSubmit.onclick = () => {
    const radarScore = calcRadarScore();
    const prompt = genPrompt(radarScore);
    promptOutput.textContent = prompt;
    drawRadarChart(radarScore);
    resultSection.style.display = 'block';
    btnSubmit.disabled = true;
    window.scrollTo({top: document.body.scrollHeight, behavior: 'smooth'});
  };

  // 重新开始事件
  btnRestart.onclick = () => {
    answers.fill(null);
    renderQuestions();
    btnSubmit.disabled = true;
    promptOutput.textContent = '——请先完成问卷——';
    resultSection.style.display = 'none';
    window.scrollTo({top: 0, behavior: 'smooth'});
  };

  // 初始化
  renderQuestions();
</script>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</body>
</html>
