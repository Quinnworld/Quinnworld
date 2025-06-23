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

      <div id="paywall">
        <div>扫描下方二维码支付 ¥9.9 元，解锁<span style="color:#822;">深度人格分析报告完整版</span></div>
        <img src="https://api.qrserver.com/v1/create-qr-code/?size=200x200&data=YOUR_PAYMENT_LINK" alt="支付二维码" />
      </div>

      <button id="btnRestart">重新开始答题</button>
    </section>
  </div>

<script>
  // 12题问卷数据，含选项和对应多维雷达评分（仅示范核心）
  const questions = [
    {
      text: "你喜欢什么样的户外活动？",
      options: [
        { text: "海边日光浴", radar: { pigment:2, advent:1, happiness:1 }},
        { text: "森林徒步", radar: { bone:1, stable:1, stamina:1 }},
        { text: "极限运动", radar: { body:1, advent:2, stamina:2 }},
        { text: "更喜欢室内", radar: { pigment:-1, body:-1, stable:1, create:1 }}
      ]
    },
    {
      text: "你喜欢什么类型的饮食？",
      options: [
        { text: "高蛋白健身餐", radar: { body:-2, self:1, stamina:1 }},
        { text: "高碳水主食", radar: { body:2, happiness:1 }},
        { text: "喜欢甜食", radar: { body:2, stable:-1, happiness:1 }},
        { text: "清淡素食", radar: { body:-1, pigment:1, empathy:1 }}
      ]
    },
    {
      text: "你在社交中最常被夸奖什么？",
      options: [
        { text: "五官立体", radar: { bone:2 }},
        { text: "皮肤健康", radar: { pigment:2 }},
        { text: "气质出众", radar: { advent:1, stable:1, create:1 }},
        { text: "身材匀称", radar: { body:2, self:1 }}
      ]
    },
    {
      text: "你喜欢什么风格的服饰？",
      options: [
        { text: "欧美时尚", radar: { pigment:1, bone:1, create:1 }},
        { text: "日韩清新", radar: { hair:2, social:1 }},
        { text: "运动休闲", radar: { body:1, advent:1, stamina:1 }},
        { text: "民族风", radar: { pigment:1, advent:1, empathy:1 }}
      ]
    },
    {
      text: "你在压力下更倾向？",
      options: [
        { text: "主动社交", radar: { advent:2, social:2 }},
        { text: "独自消化", radar: { stable:2, self:1 }},
        { text: "运动释放", radar: { body:1, advent:1, stamina:2 }},
        { text: "吃东西缓解", radar: { body:2, stable:-1, happiness:1 }}
      ]
    },
    {
      text: "你最喜欢的季节？",
      options: [
        { text: "夏天", radar: { pigment:2, happiness:1 }},
        { text: "冬天", radar: { body:1, stable:1, self:1 }},
        { text: "春天", radar: { hair:1, pigment:1, empathy:1 }},
        { text: "秋天", radar: { bone:1, stable:1, happiness:1 }}
      ]
    },
    {
      text: "你对自己的头发最满意哪些方面？",
      options: [
        { text: "发质柔顺", radar: { hair:2 }},
        { text: "自然卷曲", radar: { hair:1, bone:1, create:1 }},
        { text: "头发颜色独特", radar: { hair:2, pigment:1, create:2 }},
        { text: "不太满意", radar: { hair:-2, self:-1 }}
      ]
    },
    {
      text: "你笑起来时最突出的特点？",
      options: [
        { text: "酒窝明显", radar: { bone:2, happiness:1 }},
        { text: "牙齿整齐", radar: { bone:1, hair:1, self:1 }},
        { text: "颧骨分明", radar: { bone:2, create:1 }},
        { text: "脸型圆润", radar: { bone:1, body:1, empathy:1 }}
      ]
    },
    {
      text: "你更容易被什么吸引？",
      options: [
        { text: "与众不同的外表", radar: { pigment:2, hair:1, create:2 }},
        { text: "深邃的气质", radar: { bone:1, advent:1, empathy:1 }},
        { text: "有趣的谈吐", radar: { advent:2, create:2 }},
        { text: "温和的性格", radar: { stable:2, empathy:2 }}
      ]
    },
    {
      text: "你偏爱的发色？",
      options: [
        { text: "自然黑/棕", radar: { hair:1 }},
        { text: "金色/浅色", radar: { hair:2, pigment:2, create:1 }},
        { text: "红色/紫色", radar: { hair:2, pigment:2, create:2 }},
        { text: "其他亮色", radar: { hair:1, pigment:1, create:1 }}
      ]
    },
    {
      text: "你在童年最显著的外貌变化？",
      options: [
        { text: "开始长雀斑", radar: { pigment:2, happiness:1 }},
        { text: "牙齿或颧骨变化", radar: { bone:2, hair:1, self:1 }},
        { text: "体型明显改变", radar: { body:2, stamina:1 }},
        { text: "基本没变化", radar: { bone:-1, body:-1, stable:1 }}
      ]
    },
    {
      text: "你面对新环境时？",
      options: [
        { text: "兴奋好奇", radar: { advent:2, create:1, social:1 }},
        { text: "谨慎观察", radar: { stable:2, self:1 }},
        { text: "主动融入", radar: { advent:1, stable:1, social:2 }},
        { text: "不适应", radar: { advent:-2, stable:-2, happiness:-1 }}
      ]
    }
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
    // 色素
    if(radar.pigment > 2) parts.push("fair skin, vivid eyes, striking hair color,");
    else if(radar.pigment < -1) parts.push("tan skin, natural skin tone,");

    // 骨相
    if(radar.bone > 2) parts.push("high cheekbones, defined jaw, 3D face,");
    else if(radar.bone < -1) parts.push("round face, soft facial lines,");

    // 体型
    if(radar.body > 2) parts.push("full figure, healthy body,");
    else if(radar.body < -1) parts.push("slim body, slender,");

    // 毛发
    if(radar.hair > 2) parts.push("voluminous hair, unique hairstyle,");
    else if(radar.hair < -1) parts.push("thin soft hair,");

    // 行为与气质
    if(radar.advent > 2) parts.push("adventurous, confident, lively,");
    else if(radar.advent < -1) parts.push("reserved, introverted, calm,");

    if(radar.stable > 2) parts.push("emotionally stable, poised,");
    else if(radar.stable < -1) parts.push("sensitive, gentle temperament,");

    if(radar.social > 2) parts.push("very social, charismatic,");
    else if(radar.social < -1) parts.push("private, prefers solitude,");

    if(radar.create > 2) parts.push("creative, imaginative, unique style,");

    if(radar.self > 2) parts.push("disciplined, persistent,");

    if(radar.stamina > 2) parts.push("high stamina, energetic,");

    if(radar.empathy > 2) parts.push("empathetic, warm-hearted,");

    if(radar.happiness > 2) parts.push("bright, sunny disposition,");

    parts.push("beautiful lighting, best quality, ultra detailed");
    return parts.join(" ");
  }

  // 计算综合雷达得分
  function calcRadarScore(){
    let radarScore = {};
    radarDims.forEach(dim => radarScore[dim.key] = 0);
    answers.forEach((ans, i) => {
      if(ans !== null){
        const radar = questions[i].options[ans].radar;
        for(const k in radar){
          if(radarScore[k] === undefined) radarScore[k] = 0;
          radarScore[k] += radar[k];
        }
      }
    });
    // 限制范围并四舍五入
    for(const k in radarScore){
      if(radarScore[k] > 4) radarScore[k] = 4;
      else if(radarScore[k] < -4) radarScore[k] = -4;
      radarScore[k] = Math.round(radarScore[k]);
    }
    return radarScore;
  }

  // 提交事件
  btnSubmit.onclick = () => {
    const radarScore = calcRadarScore();
    const prompt = genPrompt(radarScore);
    promptOutput.textContent = prompt;
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

</body>
</html>
