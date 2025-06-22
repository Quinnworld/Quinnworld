<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>基因标签动态题库问卷示范</title>
<style>
  body { max-width: 600px; margin: 20px auto; font-family: "微软雅黑", sans-serif; background: #f9fbfd; padding: 20px; }
  h2 { text-align: center; }
  .section { margin-bottom: 20px; }
  .question { font-weight: 700; margin-bottom: 10px; }
  .option { background: white; border: 1px solid #bbb; border-radius: 6px; padding: 10px; margin: 6px 0; cursor: pointer; user-select:none; }
  .option.selected { background: #60a5fa; color: white; border-color: #3b82f6; }
  button { width: 100%; padding: 12px; font-size: 16px; border: none; border-radius: 6px; background: #3b82f6; color: white; cursor: pointer; }
  button:disabled { background: #93c5fd; cursor: not-allowed; }
  pre { background: #eee; padding: 10px; border-radius: 6px; overflow-x: auto; }
</style>
</head>
<body>

<h2>基因标签动态题库问卷示范</h2>

<div id="sectionQuiz" class="section">
  <div id="questionText" class="question"></div>
  <div id="optionsContainer"></div>
  <button id="btnNext" disabled>下一题</button>
</div>

<div id="sectionResult" class="section" style="display:none;">
  <h3>答题结束，基因标签得分：</h3>
  <pre id="tagScoresOutput"></pre>
  <h3>简要解读：</h3>
  <div id="interpretation"></div>
  <button id="btnRestart">重新开始</button>
</div>

<script>
  // 题库按基因标签分组
  const questionPool = {
    extraversion: [
      { id: "E1", text: "你喜欢参加聚会和社交活动吗？", options: [
        { text: "非常不喜欢", tags: { extraversion: -2 } },
        { text: "不太喜欢", tags: { extraversion: -1 } },
        { text: "比较喜欢", tags: { extraversion: 1 } },
        { text: "非常喜欢", tags: { extraversion: 2 } }
      ]},
      { id: "E2", text: "你会主动认识新朋友吗？", options: [
        { text: "几乎不会", tags: { extraversion: -2 } },
        { text: "偶尔会", tags: { extraversion: -1 } },
        { text: "经常会", tags: { extraversion: 1 } },
        { text: "总是会", tags: { extraversion: 2 } }
      ]},
    ],
    emotion_stability: [
      { id: "EM1", text: "遇到困难时你的情绪如何？", options: [
        { text: "非常焦虑", tags: { emotion_stability: -2 } },
        { text: "有些紧张", tags: { emotion_stability: -1 } },
        { text: "基本稳定", tags: { emotion_stability: 1 } },
        { text: "非常冷静", tags: { emotion_stability: 2 } }
      ]},
      { id: "EM2", text: "你容易感到压力吗？", options: [
        { text: "非常容易", tags: { emotion_stability: -2 } },
        { text: "有时容易", tags: { emotion_stability: -1 } },
        { text: "不太容易", tags: { emotion_stability: 1 } },
        { text: "几乎不会", tags: { emotion_stability: 2 } }
      ]}
    ],
    novelty_seek: [
      { id: "N1", text: "你喜欢尝试新鲜事物吗？", options: [
        { text: "完全不喜欢", tags: { novelty_seek: -2 } },
        { text: "不太喜欢", tags: { novelty_seek: -1 } },
        { text: "比较喜欢", tags: { novelty_seek: 1 } },
        { text: "非常喜欢", tags: { novelty_seek: 2 } }
      ]},
      { id: "N2", text: "你愿意接受生活中的变化吗？", options: [
        { text: "非常抗拒", tags: { novelty_seek: -2 } },
        { text: "比较抗拒", tags: { novelty_seek: -1 } },
        { text: "比较接受", tags: { novelty_seek: 1 } },
        { text: "非常接受", tags: { novelty_seek: 2 } }
      ]}
    ],
    responsibility: [
      { id: "R1", text: "你处理任务时的态度？", options: [
        { text: "经常拖延", tags: { responsibility: -2 } },
        { text: "偶尔拖延", tags: { responsibility: -1 } },
        { text: "比较负责", tags: { responsibility: 1 } },
        { text: "非常负责", tags: { responsibility: 2 } }
      ]},
      { id: "R2", text: "你会主动承担责任吗？", options: [
        { text: "几乎不会", tags: { responsibility: -2 } },
        { text: "有时会", tags: { responsibility: -1 } },
        { text: "经常会", tags: { responsibility: 1 } },
        { text: "总是会", tags: { responsibility: 2 } }
      ]}
    ]
  };

  // 当前标签分数
  let tagScores = {
    extraversion: 0,
    emotion_stability: 0,
    novelty_seek: 0,
    responsibility: 0
  };

  // 记录已答题ID防重复
  let askedQuestions = new Set();

  // 当前题
  let currentQuestion = null;

  // 预设总题数
  const maxQuestions = 8;
  let questionsAskedCount = 0;

  // 页面元素
  const qTextEl = document.getElementById("questionText");
  const optionsEl = document.getElementById("optionsContainer");
  const btnNext = document.getElementById("btnNext");
  const sectionQuiz = document.getElementById("sectionQuiz");
  const sectionResult = document.getElementById("sectionResult");
  const tagScoresOutput = document.getElementById("tagScoresOutput");
  const interpretationEl = document.getElementById("interpretation");
  const btnRestart = document.getElementById("btnRestart");

  let selectedOptionIndex = null;

  // 按标签分数选下个题目类别（极值优先）
  function selectNextCategory() {
    let entries = Object.entries(tagScores);
    // 按绝对值降序
    entries.sort((a,b) => Math.abs(b[1]) - Math.abs(a[1]));
    // 尝试依次选一个有剩余题的类别
    for(let [category] of entries){
      if (hasAvailableQuestions(category)) return category;
    }
    // 如果都没了，返回null
    return null;
  }

  // 判断某类别是否有未答题
  function hasAvailableQuestions(category) {
    return questionPool[category].some(q => !askedQuestions.has(q.id));
  }

  // 随机从类别中选题
  function selectNextQuestion(category) {
    const available = questionPool[category].filter(q => !askedQuestions.has(q.id));
    if(available.length === 0) return null;
    const q = available[Math.floor(Math.random()*available.length)];
    askedQuestions.add(q.id);
    return q;
  }

  // 渲染题目
  function renderQuestion(question) {
    currentQuestion = question;
    qTextEl.textContent = question.text;
    optionsEl.innerHTML = "";
    selectedOptionIndex = null;
    btnNext.disabled = true;
    question.options.forEach((opt,i) => {
      const div = document.createElement("div");
      div.className = "option";
      div.textContent = opt.text;
      div.onclick = () => {
        selectOption(i);
      };
      optionsEl.appendChild(div);
    });
  }

  // 选项点击
  function selectOption(idx) {
    selectedOptionIndex = idx;
    Array.from(optionsEl.children).forEach((el,i) => {
      el.classList.toggle("selected", i===idx);
    });
    btnNext.disabled = false;
  }

  // 下一题按钮事件
  btnNext.onclick = () => {
    if(selectedOptionIndex === null) return;
    // 累积tag分
    const selectedOpt = currentQuestion.options[selectedOptionIndex];
    for(let tag in selectedOpt.tags){
      tagScores[tag] = (tagScores[tag] || 0) + selectedOpt.tags[tag];
    }
    questionsAskedCount++;
    if(questionsAskedCount >= maxQuestions){
      showResult();
    } else {
      const nextCategory = selectNextCategory();
      if(!nextCategory){
        // 无题目可选，提前结束
        showResult();
        return;
      }
      const nextQ = selectNextQuestion(nextCategory);
      if(!nextQ){
        showResult();
        return;
      }
      renderQuestion(nextQ);
    }
  };

  // 显示结果
  function showResult() {
    sectionQuiz.style.display = "none";
    sectionResult.style.display = "block";
    tagScoresOutput.textContent = JSON.stringify(tagScores, null, 2);

    // 简单解读
    let interp = "";
    if(tagScores.extraversion > 2) interp += "你偏外向，喜欢社交和活跃。\n";
    else if(tagScores.extraversion < -2) interp += "你较内向，喜欢安静和独处。\n";
    else interp += "你的外向性较为中和。\n";

    if(tagScores.emotion_stability > 2) interp += "你情绪稳定，抗压能力较强。\n";
    else if(tagScores.emotion_stability < -2) interp += "你情绪敏感，易受压力影响。\n";
    else interp += "你的情绪稳定性一般。\n";

    if(tagScores.novelty_seek > 2) interp += "你喜欢尝试新事物，开放且好奇。\n";
    else if(tagScores.novelty_seek < -2) interp += "你较为保守，喜欢熟悉的环境。\n";

    if(tagScores.responsibility > 2) interp += "你责任心强，做事认真负责。\n";
    else if(tagScores.responsibility < -2) interp += "你可能有拖延倾向，需要提升责任感。\n";

    interpretationEl.textContent = interp;
  }

  btnRestart.onclick = () => {
    // 重置状态
    tagScores = { extraversion:0, emotion_stability:0, novelty_seek:0, responsibility:0 };
    askedQuestions.clear();
    questionsAskedCount = 0;
    sectionResult.style.display = "none";
    sectionQuiz.style.display = "block";

    // 首题随机选类别且题目
    const categories = Object.keys(questionPool);
    const firstCategory = categories[Math.floor(Math.random()*categories.length)];
    const firstQ = selectNextQuestion(firstCategory);
    renderQuestion(firstQ);
  };

  // 页面加载默认启动
  btnRestart.click();

</script>

</body>
</html>
