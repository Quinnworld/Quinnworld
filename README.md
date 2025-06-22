<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>基因科学动态问卷 + 雷达图展示 + 动漫Prompt生成</title>
<style>
  body { max-width: 600px; margin: 20px auto; font-family: "微软雅黑", sans-serif; background: #f0f4f8; padding: 20px; }
  h2, h3 { text-align: center; }
  label { display: block; margin: 10px 0 5px; font-weight: 600; }
  select, input[type=number] { width: 100%; padding: 8px; font-size: 16px; margin-bottom: 15px; border-radius: 6px; border: 1px solid #ccc; }
  .section { margin-bottom: 20px; }
  .question { font-weight: 700; margin-bottom: 10px; }
  .option { background: white; border: 1px solid #bbb; border-radius: 6px; padding: 10px; margin: 6px 0; cursor: pointer; user-select:none; }
  .option.selected { background: #4f46e5; color: white; border-color: #4338ca; }
  button { width: 100%; padding: 12px; font-size: 16px; border: none; border-radius: 6px; background: #4338ca; color: white; cursor: pointer; }
  button:disabled { background: #a5b4fc; cursor: not-allowed; }
  pre { background: #eee; padding: 10px; border-radius: 6px; overflow-x: auto; white-space: pre-wrap; word-break: break-word; max-height: 200px; overflow-y: auto; }
  canvas { display: block; margin: 0 auto 20px auto; }
  #totalScore { text-align: center; font-weight: 700; font-size: 18px; margin-bottom: 10px; }
</style>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>

<h2>基因科学动态问卷 + 雷达图展示 + 动漫Prompt生成</h2>

<div class="section" id="sectionUserInfo">
  <label for="inputAge">年龄（18-80）</label>
  <input type="number" id="inputAge" min="18" max="80" value="25" />
  <label for="selectGender">性别</label>
  <select id="selectGender">
    <option value="male">男性</option>
    <option value="female">女性</option>
  </select>
  <button id="btnStart">开始答题</button>
</div>

<div class="section" id="sectionQuiz" style="display:none;">
  <div id="questionText" class="question"></div>
  <div id="optionsContainer"></div>
  <button id="btnNext" disabled>下一题</button>
  <div style="margin-top:10px; font-size:14px; color:#555;" id="progressText"></div>
</div>

<div class="section" id="sectionResult" style="display:none;">
  <h3>答题结束，基因标签雷达图评分：</h3>
  <div id="totalScore">总分：<span id="totalScoreValue">0</span>/1000</div> <!-- 总分区 -->

  <canvas id="radarChart" width="400" height="400"></canvas>

  <h3>生成的动漫风格Prompt：</h3>
  <pre id="promptOutput"></pre>
  <button id="btnRestart">重新开始</button>
</div>

<script>
  const geneTags = [
    "外向性", "情绪稳定性", "新奇寻求", "责任感",
    "冲动性", "焦虑水平", "自我控制", "开放性", "宜人性", "尽责性"
  ];

  // 题库同之前，省略重复部分，保持不变
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
        { text: "非常紧张和焦虑", tags: { emotion_stability: -2, anxiety: 2 } },
        { text: "有些不安", tags: { emotion_stability: -1, anxiety: 1 } },
        { text: "情绪稳定", tags: { emotion_stability: 1, anxiety: -1 } },
        { text: "非常冷静", tags: { emotion_stability: 2, anxiety: -2 } }
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
        { text: "经常拖延", tags: { responsibility: -2, conscientiousness: -2 } },
        { text: "偶尔拖延", tags: { responsibility: -1, conscientiousness: -1 } },
        { text: "大部分时间按时", tags: { responsibility: 1, conscientiousness: 1 } },
        { text: "总是按时完成", tags: { responsibility: 2, conscientiousness: 2 } }
      ]
    },
    {
      id: "Q5", text: "你做决定时是否容易冲动？",
      options: [
        { text: "完全不会冲动", tags: { impulsivity: -2, self_control: 2 } },
        { text: "不太冲动", tags: { impulsivity: -1, self_control: 1 } },
        { text: "有时冲动", tags: { impulsivity: 1, self_control: -1 } },
        { text: "非常冲动", tags: { impulsivity: 2, self_control: -2 } }
      ]
    },
    {
      id: "Q6", text: "你是否容易感到焦虑？",
      options: [
        { text: "几乎不焦虑", tags: { anxiety: -2, emotion_stability: 2 } },
        { text: "偶尔焦虑", tags: { anxiety: -1, emotion_stability: 1 } },
        { text: "经常焦虑", tags: { anxiety: 1, emotion_stability: -1 } },
        { text: "非常焦虑", tags: { anxiety: 2, emotion_stability: -2 } }
      ]
    },
    {
      id: "Q7", text: "你喜欢艺术和创造性活动吗？",
      options: [
        { text: "完全不喜欢", tags: { openness: -2 } },
        { text: "不太喜欢", tags: { openness: -1 } },
        { text: "比较喜欢", tags: { openness: 1 } },
        { text: "非常喜欢", tags: { openness: 2 } }
      ]
    },
    {
      id: "Q8", text: "你待人是否友好与合作？",
      options: [
        { text: "不太友好", tags: { agreeableness: -2 } },
        { text: "有时冷淡", tags: { agreeableness: -1 } },
        { text: "比较友好", tags: { agreeableness: 1 } },
        { text: "非常友好", tags: { agreeableness: 2 } }
      ]
    },
    {
      id: "Q9", text: "你平时做事是否细致且有条理？",
      options: [
        { text: "非常粗心", tags: { conscientiousness: -2 } },
        { text: "有些粗心", tags: { conscientiousness: -1 } },
        { text: "比较细心", tags: { conscientiousness: 1 } },
        { text: "非常细心", tags: { conscientiousness: 2 } }
      ]
    },
    {
      id: "Q10", text: "你控制情绪的能力如何？",
      options: [
        { text: "很差", tags: { self_control: -2 } },
        { text: "一般", tags: { self_control: -1 } },
        { text: "较好", tags: { self_control: 1 } },
        { text: "非常好", tags: { self_control: 2 } }
      ]
    },
    {
      id: "Q11", text: "你是否经常拖延？",
      options: [
        { text: "经常拖延", tags: { responsibility: -2 } },
        { text: "偶尔拖延", tags: { responsibility: -1 } },
        { text: "较少拖延", tags: { responsibility: 1 } },
        { text: "几乎不拖延", tags: { responsibility: 2 } }
      ]
    },
    {
      id: "Q12", text: "你喜欢接受新观点和变化吗？",
      options: [
        { text: "完全不喜欢", tags: { openness: -2 } },
        { text: "不太喜欢", tags: { openness: -1 } },
        { text: "比较喜欢", tags: { openness: 1 } },
        { text: "非常喜欢", tags: { openness: 2 } }
      ]
    }
  ];

  // 映射中文标签到英文基因标签key，方便数据叠加
  const tagKeyMap = {
    "外向性": "extraversion",
    "情绪稳定性": "emotion_stability",
    "新奇寻求": "novelty_seek",
    "责任感": "responsibility",
    "冲动性": "impulsivity",
    "焦虑水平": "anxiety",
    "自我控制": "self_control",
    "开放性": "openness",
    "宜人性": "agreeableness",
    "尽责性": "conscientiousness"
  };

  function ageGenderAdjust(tags, age, gender){
    let adjusted = {...tags};
    if(age < 30){
      adjusted.novelty_seek = (adjusted.novelty_seek || 0) * 1.2;
      adjusted.openness = (adjusted.openness || 0) * 1.2;
    }
    if(gender === "female"){
      adjusted.anxiety = (adjusted.anxiety || 0) * 1.15;
    }
    return adjusted;
  }

  let tagScores = {};
  let askedIds = new Set();
  let currentQuestion = null;
  let selectedOptionIndex = null;
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
  const promptOutput = document.getElementById("promptOutput");
  const btnStart = document.getElementById("btnStart");
  const btnRestart = document.getElementById("btnRestart");
  const inputAge = document.getElementById("inputAge");
  const selectGender = document.getElementById("selectGender");
  const totalScoreValue = document.getElementById("totalScoreValue");

  function selectOption(idx){
    selectedOptionIndex = idx;
    Array.from(optionsEl.children).forEach((el,i)=>{
      el.classList.toggle("selected", i === idx);
    });
    btnNext.disabled = false;
  }

  function selectNextCategory(){
    let entries = Object.entries(tagScores);
    if(entries.length === 0){
      return geneTags[Math.floor(Math.random()*geneTags.length)];
    }
    entries.sort((a,b)=>Math.abs(b[1]) - Math.abs(a[1]));
    for(let [cat] of entries){
      if(questionPool.some(q=>q.id && !askedIds.has(q.id) && q.options.some(o=>o.tags && o.tags[catKeyMap[cat]] !== undefined))){
        return catKeyMap[cat];
      }
    }
    for(let q of questionPool){
      if(!askedIds.has(q.id)) return q;
    }
    return null;
  }

  function selectNextQuestion(){
    const category = selectNextCategory();
    if(typeof category === "string"){
      let candidates = questionPool.filter(q=>!askedIds.has(q.id) && q.options.some(o=>o.tags && o.tags[category] !== undefined));
      if(candidates.length === 0){
        candidates = questionPool.filter(q=>!askedIds.has(q.id));
      }
      if(candidates.length === 0) return null;
      const q = candidates[Math.floor(Math.random()*candidates.length)];
      return q;
    } else if(category && typeof category === "object"){
      return category;
    }
    return null;
  }

  function renderQuestion(q){
    currentQuestion = q;
    qTextEl.textContent = q.text;
    optionsEl.innerHTML = "";
    selectedOptionIndex = null;
    btnNext.disabled = true;
    for(let i=0; i<q.options.length; i++){
      const opt = document.createElement("div");
      opt.className = "option";
      opt.textContent = q.options[i].text;
      opt.onclick = ()=>selectOption(i);
      optionsEl.appendChild(opt);
    }
    progressText.textContent = `第 ${questionCount+1} / ${maxQuestions} 题`;
  }

  btnNext.onclick = ()=>{
    if(selectedOptionIndex === null) return;
    const selTags = currentQuestion.options[selectedOptionIndex].tags;
    for(let t in selTags){
      tagScores[t] = (tagScores[t] || 0) + selTags[t];
    }
    askedIds.add(currentQuestion.id);
    questionCount++;
    if(questionCount >= maxQuestions){
      showResult();
    } else {
      const nextQ = selectNextQuestion();
      if(!nextQ){
        showResult();
      } else {
        renderQuestion(nextQ);
      }
    }
  };

  function generatePrompt(tags){
    const adjustedTags = ageGenderAdjust(tags, age, gender);

    let descs = [];
    if((adjustedTags.extraversion || 0) > 1) descs.push("活泼外向的性格");
    else if((adjustedTags.extraversion || 0) < -1) descs.push("沉静内向的气质");
    else descs.push("社交行为均衡");

    if((adjustedTags.emotion_stability || 0) > 1) descs.push("情绪稳定且自信");
    else if((adjustedTags.emotion_stability || 0) < -1) descs.push("敏感且易受压力影响");

    if((adjustedTags.novelty_seek || 0) > 1) descs.push("喜欢新奇与探索");
    else if((adjustedTags.novelty_seek || 0) < -1) descs.push("偏好熟悉和规律");

    if((adjustedTags.responsibility || 0) > 1) descs.push("责任感强且可靠");
    else if((adjustedTags.responsibility || 0) < -1) descs.push("有时粗心或拖延");

    if((adjustedTags.impulsivity || 0) > 1) descs.push("冲动且自发");
    else if((adjustedTags.impulsivity || 0) < -1) descs.push("谨慎且自律");

    if((adjustedTags.anxiety || 0) > 1) descs.push("经常焦虑和担忧");
    else if((adjustedTags.anxiety || 0) < -1) descs.push("压力下冷静");

    if((adjustedTags.self_control || 0) > 1) descs.push("极佳的自我控制力");
    else if((adjustedTags.self_control || 0) < -1) descs.push("有时难以自律");

    if((adjustedTags.openness || 0) > 1) descs.push("创造力丰富且富想象力");
    else if((adjustedTags.openness || 0) < -1) descs.push("实用且传统");

    if((adjustedTags.agreeableness || 0) > 1) descs.push("友善且合作");
    else if((adjustedTags.agreeableness || 0) < -1) descs.push("竞争且怀疑");

    if((adjustedTags.conscientiousness || 0) > 1) descs.push("有条理且勤勉");
    else if((adjustedTags.conscientiousness || 0) < -1) descs.push("有时粗心");

    const style = "动漫风格，色彩明快，眼睛细致，现代休闲服饰";

    return `动漫角色描述：一个${descs.join("，")}的人物。风格：${style}。`;
  }

  function showResult(){
    sectionQuiz.style.display = "none";
    sectionResult.style.display = "block";

    const adjustedTags = ageGenderAdjust(tagScores, age, gender);

    // 计算雷达图数据并映射0-100分
    const radarData = geneTags.map(label => {
      const key = tagKeyMap[label];
      const val = adjustedTags[key] || 0;
      let score = Math.round(((val + 24) / 48) * 100);
      if(score > 100) score = 100;
      if(score < 0) score = 0;
      return score;
    });

    // 计算总分（满分100 * 标签数）
    const total = radarData.reduce((a,b) => a+b, 0);
    totalScoreValue.textContent = `${total} / ${radarData.length * 100}`;

    // 销毁旧图表（如果有）
    const ctx = document.getElementById('radarChart').getContext('2d');
    if(window.myRadarChart) window.myRadarChart.destroy();

    // 新建雷达图，中文标签，显示点分数
    window.myRadarChart = new Chart(ctx, {
      type: 'radar',
      data: {
        labels: geneTags,
        datasets: [{
          label: '基因标签得分',
          data: radarData,
          fill: true,
          backgroundColor: 'rgba(67,56,202,0.2)',
          borderColor: 'rgba(67,56,202,1)',
          pointBackgroundColor: 'rgba(67,56,202,1)',
          pointBorderColor: '#fff',
          pointHoverBackgroundColor: '#fff',
          pointHoverBorderColor: 'rgba(67,56,202,1)'
        }]
      },
      options: {
        scales: {
          r: {
            angleLines: { display: true },
            suggestedMin: 0,
            suggestedMax: 100,
            ticks: {
              stepSize: 20,
              color: '#333',
              showLabelBackdrop: false
            },
            pointLabels: {
              font: { size: 14 },
              color: '#555'
            }
          }
        },
        plugins: {
          legend: { display: true, position: 'top' },
          tooltip: {
            enabled: true,
            callbacks: {
              label: function(context) {
                return context.dataset.label + ': ' + context.parsed.r + ' 分';
              }
            }
          },
          datalabels: {
            display: true
          }
        },
        responsive: false,
        maintainAspectRatio: false,
        elements: {
          point: {
            radius: 5,
            hoverRadius: 7
          }
        }
      }
    });

    // 输出prompt，支持换行滚动
    promptOutput.textContent = generatePrompt(adjustedTags);
  }

  btnStart.onclick = () => {
    let a = parseInt(inputAge.value);
    if(isNaN(a) || a < 18 || a > 80){
      alert("请输入18-80之间的有效年龄");
      return;
    }
    age = a;
    gender = selectGender.value;
    tagScores = {};
    askedIds.clear();
    questionCount = 0;
    sectionUserInfo.style.display = "none";
    sectionResult.style.display = "none";
    sectionQuiz.style.display = "block";
    const q = selectNextQuestion();
    renderQuestion(q);
  };

  btnRestart.onclick = () => {
    sectionResult.style.display = "none";
    sectionUserInfo.style.display = "block";
  };

</script>

</body>
</html>
