<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>基因科学动态问卷 + 动漫Prompt生成</title>
<style>
  body { max-width: 600px; margin: 20px auto; font-family: "微软雅黑", sans-serif; background: #f0f4f8; padding: 20px; }
  h2 { text-align: center; }
  label { display: block; margin: 10px 0 5px; font-weight: 600; }
  select, input[type=number] { width: 100%; padding: 8px; font-size: 16px; margin-bottom: 15px; border-radius: 6px; border: 1px solid #ccc; }
  .section { margin-bottom: 20px; }
  .question { font-weight: 700; margin-bottom: 10px; }
  .option { background: white; border: 1px solid #bbb; border-radius: 6px; padding: 10px; margin: 6px 0; cursor: pointer; user-select:none; }
  .option.selected { background: #4f46e5; color: white; border-color: #4338ca; }
  button { width: 100%; padding: 12px; font-size: 16px; border: none; border-radius: 6px; background: #4338ca; color: white; cursor: pointer; }
  button:disabled { background: #a5b4fc; cursor: not-allowed; }
  pre { background: #eee; padding: 10px; border-radius: 6px; overflow-x: auto; }
</style>
</head>
<body>

<h2>基因科学动态问卷 + 动漫Prompt生成</h2>

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
  <h3>答题结束，基因标签得分：</h3>
  <pre id="tagScoresOutput"></pre>
  <h3>简要解读：</h3>
  <div id="interpretation"></div>
  <h3>生成的动漫风格Prompt：</h3>
  <pre id="promptOutput"></pre>
  <button id="btnRestart">重新开始</button>
</div>

<script>
  // 核心基因标签列表
  const geneTags = [
    "extraversion", "emotion_stability", "novelty_seek", "responsibility",
    "impulsivity", "anxiety", "self_control", "openness", "agreeableness", "conscientiousness"
  ];

  // 题库示例，12题，每题4选项，每选项对应多个基因标签得分
  // 分值范围 -2 ~ 2
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

  // 年龄性别调整权重函数
  function ageGenderAdjust(tags, age, gender){
    // 简单示范，年轻人novelty_seek和openness权重+20%，女性anxiety权重+15%
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

  // 当前答题状态
  let tagScores = {};
  let askedIds = new Set();
  let currentQuestion = null;
  let selectedOptionIndex = null;
  let questionCount = 0;
  const maxQuestions = 12;
  let age = 25;
  let gender = "male";

  // 页面元素
  const sectionUserInfo = document.getElementById("sectionUserInfo");
  const sectionQuiz = document.getElementById("sectionQuiz");
  const sectionResult = document.getElementById("sectionResult");
  const qTextEl = document.getElementById("questionText");
  const optionsEl = document.getElementById("optionsContainer");
  const btnNext = document.getElementById("btnNext");
  const progressText = document.getElementById("progressText");
  const tagScoresOutput = document.getElementById("tagScoresOutput");
  const interpretationEl = document.getElementById("interpretation");
  const promptOutput = document.getElementById("promptOutput");
  const btnStart = document.getElementById("btnStart");
  const btnRestart = document.getElementById("btnRestart");
  const inputAge = document.getElementById("inputAge");
  const selectGender = document.getElementById("selectGender");

  // 选项点击处理
  function selectOption(idx){
    selectedOptionIndex = idx;
    Array.from(optionsEl.children).forEach((el,i)=>{
      el.classList.toggle("selected", i === idx);
    });
    btnNext.disabled = false;
  }

  // 动态选题策略：优先选择标签分数绝对值最大对应类别的未答题
  function selectNextCategory(){
    // 按标签分绝对值降序排列
    let entries = Object.entries(tagScores);
    if(entries.length === 0){
      // 初始化时随机选一个类别
      const randomCategory = geneTags[Math.floor(Math.random()*geneTags.length)];
      return randomCategory;
    }
    entries.sort((a,b)=>Math.abs(b[1]) - Math.abs(a[1]));
    for(let [cat] of entries){
      if(questionPool.some(q=>q.id && !askedIds.has(q.id) && q.options.some(o=>o.tags && o.tags[cat] !== undefined))){
        return cat;
      }
    }
    // 找不到合适的，随机题
    for(let q of questionPool){
      if(!askedIds.has(q.id)) return q;
    }
    return null;
  }

  // 选题
  function selectNextQuestion(){
    const category = selectNextCategory();
    if(typeof category === "string"){
      // 按类别找题
      let candidates = questionPool.filter(q=>!askedIds.has(q.id) && q.options.some(o=>o.tags && o.tags[category] !== undefined));
      if(candidates.length === 0){
        // 退化为任意未答题
        candidates = questionPool.filter(q=>!askedIds.has(q.id));
      }
      if(candidates.length === 0) return null;
      // 随机选一个
      const q = candidates[Math.floor(Math.random()*candidates.length)];
      return q;
    } else if(category && typeof category === "object"){
      return category;
    }
    return null;
  }

  // 渲染题目
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

  // 下一题按钮点击事件
  btnNext.onclick = ()=>{
    if(selectedOptionIndex === null) return;
    // 叠加标签分数
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

  // 生成prompt文本（动漫风格）
  function generatePrompt(tags){
    // 先做年龄性别动态调整
    const adjustedTags = ageGenderAdjust(tags, age, gender);

    let descs = [];
    if((adjustedTags.extraversion || 0) > 1) descs.push("energetic and outgoing personality");
    else if((adjustedTags.extraversion || 0) < -1) descs.push("calm and introverted demeanor");
    else descs.push("balanced social behavior");

    if((adjustedTags.emotion_stability || 0) > 1) descs.push("emotionally stable and confident");
    else if((adjustedTags.emotion_stability || 0) < -1) descs.push("sensitive and prone to stress");

    if((adjustedTags.novelty_seek || 0) > 1) descs.push("open to new experiences and curious");
    else if((adjustedTags.novelty_seek || 0) < -1) descs.push("prefers familiarity and routine");

    if((adjustedTags.responsibility || 0) > 1) descs.push("responsible and reliable");
    else if((adjustedTags.responsibility || 0) < -1) descs.push("sometimes careless or procrastinating");

    if((adjustedTags.impulsivity || 0) > 1) descs.push("impulsive and spontaneous");
    else if((adjustedTags.impulsivity || 0) < -1) descs.push("cautious and self-controlled");

    if((adjustedTags.anxiety || 0) > 1) descs.push("often anxious or worried");
    else if((adjustedTags.anxiety || 0) < -1) descs.push("calm and relaxed");

    if((adjustedTags.self_control || 0) > 1) descs.push("excellent emotional regulation");
    else if((adjustedTags.self_control || 0) < -1) descs.push("difficulty controlling emotions");

    if((adjustedTags.openness || 0) > 1) descs.push("creative and imaginative");
    else if((adjustedTags.openness || 0) < -1) descs.push("practical and down-to-earth");

    if((adjustedTags.agreeableness || 0) > 1) descs.push("kind and cooperative");
    else if((adjustedTags.agreeableness || 0) < -1) descs.push("competitive or skeptical");

    if((adjustedTags.conscientiousness || 0) > 1) descs.push("organized and diligent");
    else if((adjustedTags.conscientiousness || 0) < -1) descs.push("disorganized or negligent");

    // 组合prompt
    const prompt = `Anime style portrait of a young adult with ${descs.join(", ")}, detailed shading, vibrant colors, sharp eyes, stylish modern clothes`;

    return prompt;
  }

  // 显示结果
  function showResult(){
    sectionQuiz.style.display = "none";
    sectionResult.style.display = "block";
    // 应用年龄性别修正
    const adjustedTags = ageGenderAdjust(tagScores, age, gender);
    tagScoresOutput.textContent = JSON.stringify(adjustedTags, null, 2);

    // 简单解读
    let interpretation = [];
    for(let t of geneTags){
      let v = adjustedTags[t] || 0;
      if(v > 1) interpretation.push(`${t}：高`);
      else if(v < -1) interpretation.push(`${t}：低`);
      else interpretation.push(`${t}：中等`);
    }
    interpretationEl.innerHTML = interpretation.map(s=>`<div>${s}</div>`).join("");

    // 生成Prompt
    promptOutput.textContent = generatePrompt(adjustedTags);
  }

  // 开始答题按钮
  btnStart.onclick = ()=>{
    age = parseInt(inputAge.value);
    gender = selectGender.value;
    if(isNaN(age) || age < 18 || age > 80){
      alert("请输入有效的年龄（18-80）");
      return;
    }
    tagScores = {};
    askedIds.clear();
    questionCount = 0;
    sectionUserInfo.style.display = "none";
    sectionResult.style.display = "none";
    sectionQuiz.style.display = "block";
    const firstQ = selectNextQuestion();
    renderQuestion(firstQ);
  };

  // 重新开始按钮
  btnRestart.onclick = ()=>{
    sectionResult.style.display = "none";
    sectionUserInfo.style.display = "block";
  };
</script>

</body>
</html>
