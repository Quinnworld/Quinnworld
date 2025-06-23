<!DOCTYPE html>
<html lang="zh">
<head>
  <meta charset="UTF-8">
  <title>动态问卷 - 单题显示（从15题库中随机选择12题）</title>
  <style>
    /* 禁止页面滚动，并保证全屏显示 */
    html, body {
      margin: 0;
      padding: 0;
      overflow: hidden;
      width: 100%;
      height: 100%;
      font-family: Arial, sans-serif;
    }
    /* 整个容器充满全屏 */
    #container {
      width: 100vw;
      height: 100vh;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      background: #f4f4f4;
      box-sizing: border-box;
      text-align: center;
      padding: 20px;
    }
    /* 问题显示区域 */
    .question-box {
      background: #fff;
      border: 1px solid #ddd;
      padding: 20px;
      box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
      width: 80%;
      max-width: 600px;
      text-align: left;
      box-sizing: border-box;
    }
    .question-box h2 {
      font-size: 20px;
      margin: 0 0 15px 0;
    }
    .options {
      margin-top: 10px;
    }
    .option-label {
      display: block;
      margin-bottom: 10px;
      font-size: 16px;
    }
    /* 导航按钮区 */
    .nav-buttons {
      margin-top: 20px;
      display: flex;
      justify-content: space-between;
      width: 80%;
      max-width: 600px;
      box-sizing: border-box;
    }
    button {
      padding: 10px 20px;
      font-size: 16px;
      cursor: pointer;
    }
    /* 结果展示区域 */
    #resultContainer {
      margin-top: 20px;
      background: #fff;
      border: 1px solid #ccc;
      padding: 10px;
      max-width: 600px;
      width: 80%;
      box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
      text-align: left;
      box-sizing: border-box;
    }
  </style>
</head>
<body>
  <div id="container">
    <div id="questionContainer" class="question-box">
      <!-- 当前问题显示处 -->
    </div>
    <div class="nav-buttons">
      <button id="prevButton" style="display: none;">上一题</button>
      <button id="nextButton">下一题</button>
    </div>
    <div id="resultContainer" style="display: none;"></div>
  </div>
  
  <script>
    // Fisher-Yates 洗牌算法
    function shuffle(array) {
      for (let i = array.length - 1; i > 0; i--) {
        let j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
    }
    
    // 定义 15 道问卷题目（所有题目均不涉及视觉图片特征，仅关注内在体验、思维方式和情感倾向）
    // 注意：这里仅举例 15 道题目，你可以根据实际需求调整题目内容和基因权重。
    const allQuestions = [
      {
        id: 1,
        questionText: "在独处静思时，你的内在体验更接近哪种状态？",
        options: [
          { optionText: "平和稳重，心如止水", geneWeights: { "PAX3": 0.25, "EDAR": 0.20, "MC1R": 0.15 } },
          { optionText: "理性冷静，深藏智慧", geneWeights: { "RUNX2": 0.20, "OCA2/HERC2": 0.25, "SLC24A5": 0.15 } },
          { optionText: "温暖活跃，充满朝气", geneWeights: { "TBX15": 0.20, "SOX9": 0.20, "BMP2": 0.15 } },
          { optionText: "内心矛盾，情感复杂", geneWeights: { "DCHS2": 0.20, "SLC45A2": 0.15, "GDF6": 0.15 } }
        ]
      },
      {
        id: 2,
        questionText: "面对未来的不确定性时，你更倾向于采取哪种应对策略？",
        options: [
          { optionText: "积极规划，提前筹备", geneWeights: { "RUNX2": 0.20, "EDAR": 0.15, "OCA2/HERC2": 0.15 } },
          { optionText: "顺其自然，信赖直觉", geneWeights: { "PAX3": 0.20, "SOX9": 0.15, "MC1R": 0.10 } },
          { optionText: "勇于突破，冒险探索", geneWeights: { "TBX15": 0.25, "SLC24A5": 0.10, "BMP2": 0.15 } },
          { optionText: "谨慎行事，稳扎稳打", geneWeights: { "DCHS2": 0.20, "SLC45A2": 0.15, "GDF6": 0.15 } }
        ]
      },
      {
        id: 3,
        questionText: "结束一天忙碌后，你通常采用何种方式放松身心？",
        options: [
          { optionText: "安静阅读，沉浸书香", geneWeights: { "RUNX2": 0.15, "EDAR": 0.20, "BMP2": 0.10 } },
          { optionText: "积极运动，释放能量", geneWeights: { "MC1R": 0.15, "SLC24A5": 0.20, "TBX15": 0.10 } },
          { optionText: "独自沉思，整理思绪", geneWeights: { "SLC45A2": 0.20, "OCA2/HERC2": 0.15, "GDF6": 0.10 } },
          { optionText: "与友交谈，分享心情", geneWeights: { "PAX3": 0.15, "SOX9": 0.20, "DCHS2": 0.10 } }
        ]
      },
      {
        id: 4,
        questionText: "面对重大决策时，你通常依赖哪种方式作出决定？",
        options: [
          { optionText: "依靠直觉，快速抉择", geneWeights: { "SOX9": 0.20, "EDAR": 0.15, "GDF6": 0.10 } },
          { optionText: "运用逻辑，仔细分析", geneWeights: { "RUNX2": 0.25, "TBX15": 0.15, "BMP2": 0.10 } },
          { optionText: "参考他人意见，群策群力", geneWeights: { "PAX3": 0.20, "MC1R": 0.15, "SLC24A5": 0.10 } },
          { optionText: "敢于冒险，尝试试错", geneWeights: { "DCHS2": 0.20, "SLC45A2": 0.15, "OCA2/HERC2": 0.10 } }
        ]
      },
      {
        id: 5,
        questionText: "在面对挑战和压力时，你的内在动力通常以何种方式表现？",
        options: [
          { optionText: "保持冷静，坚持不懈", geneWeights: { "TBX15": 0.20, "SOX9": 0.15, "EDAR": 0.10 } },
          { optionText: "乐观向上，积极转化", geneWeights: { "MC1R": 0.20, "OCA2/HERC2": 0.15, "SLC24A5": 0.10 } },
          { optionText: "深思熟虑，稳中求变", geneWeights: { "RUNX2": 0.20, "BMP2": 0.15, "SLC45A2": 0.10 } },
          { optionText: "果断行动，迎难而上", geneWeights: { "PAX3": 0.20, "DCHS2": 0.15, "GDF6": 0.10 } }
        ]
      },
      {
        id: 6,
        questionText: "你是否经常会突然获得灵感，找到解决问题的新思路？",
        options: [
          { optionText: "总是灵感不断，思路清晰", geneWeights: { "TBX15": 0.20, "EDAR": 0.15, "GDF6": 0.10 } },
          { optionText: "偶尔闪现新颖想法", geneWeights: { "RUNX2": 0.15, "SLC24A5": 0.20, "BMP2": 0.10 } },
          { optionText: "较少有灵感迸发", geneWeights: { "PAX3": 0.15, "MC1R": 0.20, "SOX9": 0.10 } },
          { optionText: "几乎依赖传统方法", geneWeights: { "DCHS2": 0.15, "SLC45A2": 0.20, "OCA2/HERC2": 0.10 } }
        ]
      },
      {
        id: 7,
        questionText: "在决策过程中，你更倾向于采用哪种思维方式？",
        options: [
          { optionText: "思维稳健，步调一致", geneWeights: { "RUNX2": 0.20, "EDAR": 0.15, "BMP2": 0.10 } },
          { optionText: "迅速决断，凭直觉判断", geneWeights: { "MC1R": 0.20, "TBX15": 0.15, "SLC24A5": 0.10 } },
          { optionText: "反复推敲，深思熟虑", geneWeights: { "PAX3": 0.20, "SOX9": 0.15, "GDF6": 0.10 } },
          { optionText: "思维波动，多变不定", geneWeights: { "DCHS2": 0.20, "SLC45A2": 0.15, "OCA2/HERC2": 0.10 } }
        ]
      },
      {
        id: 8,
        questionText: "对于新事物和未知领域，你通常持怎样的态度？",
        options: [
          { optionText: "主动探索，乐于尝试", geneWeights: { "TBX15": 0.20, "EDAR": 0.15, "SLC24A5": 0.10 } },
          { optionText: "偶尔尝试，但偏向保守", geneWeights: { "PAX3": 0.20, "MC1R": 0.15, "SLC45A2": 0.10 } },
          { optionText: "倾向稳定，鲜少冒险", geneWeights: { "RUNX2": 0.20, "OCA2/HERC2": 0.15, "BMP2": 0.10 } },
          { optionText: "极力避免风险，谨慎行事", geneWeights: { "DCHS2": 0.20, "SOX9": 0.15, "GDF6": 0.10 } }
        ]
      },
      {
        id: 9,
        questionText: "如果用一个抽象概念来代表你最核心的内在特质，你会选择哪种？",
        options: [
          { optionText: "坚守信念，持之以恒", geneWeights: { "RUNX2": 0.15, "EDAR": 0.20, "BMP2": 0.10 } },
          { optionText: "内敛智慧，深藏不露", geneWeights: { "PAX3": 0.15, "TBX15": 0.20, "SOX9": 0.10 } },
          { optionText: "生气勃勃，充满活力", geneWeights: { "MC1R": 0.15, "SLC24A5": 0.20, "OCA2/HERC2": 0.10 } },
          { optionText: "灵活多变，持续成长", geneWeights: { "DCHS2": 0.15, "SLC45A2": 0.20, "GDF6": 0.10 } }
        ]
      },
      {
        id: 10,
        questionText: "当面对困难时，你是否会自发找到新的解决方案？",
        options: [
          { optionText: "总能迅速破局，迎刃而解", geneWeights: { "TBX15": 0.20, "EDAR": 0.15, "OCA2/HERC2": 0.10 } },
          { optionText: "偶有新想法，能带来突破", geneWeights: { "PAX3": 0.20, "MC1R": 0.15, "SLC24A5": 0.10 } },
          { optionText: "较少依赖创新，偏好传统", geneWeights: { "RUNX2": 0.20, "SLC45A2": 0.15, "BMP2": 0.10 } },
          { optionText: "完全依赖传统方法，不轻易变动", geneWeights: { "DCHS2": 0.20, "SOX9": 0.15, "GDF6": 0.10 } }
        ]
      },
      {
        id: 11,
        questionText: "对于未来的发展，你希望自己的人生呈现怎样的进程？",
        options: [
          { optionText: "有条不紊，稳步前行", geneWeights: { "RUNX2": 0.15, "EDAR": 0.20, "BMP2": 0.10 } },
          { optionText: "不断突破，充满创新", geneWeights: { "PAX3": 0.15, "TBX15": 0.20, "SOX9": 0.10 } },
          { optionText: "深思熟虑，务实稳重", geneWeights: { "MC1R": 0.15, "SLC24A5": 0.20, "OCA2/HERC2": 0.10 } },
          { optionText: "迎接挑战，追求极致", geneWeights: { "DCHS2": 0.15, "SLC45A2": 0.20, "GDF6": 0.10 } }
        ]
      },
      {
        id: 12,
        questionText: "将你的一段重要经历概括为一种内在体验，你认为它更像哪种过程？",
        options: [
          { optionText: "庄重严肃，充满仪式感", geneWeights: { "RUNX2": 0.20, "EDAR": 0.15, "TBX15": 0.10 } },
          { optionText: "温和流畅，令人共鸣", geneWeights: { "PAX3": 0.20, "MC1R": 0.15, "SOX9": 0.10 } },
          { optionText: "神秘复杂，引人深思", geneWeights: { "DCHS2": 0.20, "SLC45A2": 0.15, "OCA2/HERC2": 0.10 } },
          { optionText: "孤立自省，深思熟虑", geneWeights: { "GDF6": 0.20, "BMP2": 0.15, "SLC24A5": 0.10 } }
        ]
      },
      // 以下三题为新增题，共15题
      {
        id: 13,
        questionText: "在与他人交流时，你更倾向于展现哪种内在品质？",
        options: [
          { optionText: "自信果敢", geneWeights: { "KITLG": 0.20, "TYR": 0.15, "PAX3": 0.10 } },
          { optionText: "温柔体贴", geneWeights: { "IRF4": 0.20, "MC1R": 0.15, "BNC2": 0.10 } },
          { optionText: "明智理性", geneWeights: { "ASIP": 0.20, "MITF": 0.15, "SLC24A5": 0.10 } },
          { optionText: "独立创新", geneWeights: { "TYRP1": 0.20, "OCA2/HERC2": 0.15, "DCHS2": 0.10 } }
        ]
      },
      {
        id: 14,
        questionText: "如果你的个性能够以诗表达，你更认同哪种风格的诗句？",
        options: [
          { optionText: "激昂慷慨、气势磅礴", geneWeights: { "RUNX2": 0.15, "EDAR": 0.15, "KITLG": 0.10 } },
          { optionText: "柔情细腻、娓娓动听", geneWeights: { "PAX3": 0.15, "SOX9": 0.20, "IRF4": 0.10 } },
          { optionText: "简洁现代、直白真挚", geneWeights: { "TBX15": 0.15, "SLC45A2": 0.20, "MITF": 0.10 } },
          { optionText: "玄妙深邃、隐约含蓄", geneWeights: { "DCHS2": 0.15, "GDF6": 0.20, "TYR": 0.10 } }
        ]
      },
      {
        id: 15,
        questionText: "在人群中，你觉得自己最突出的内在特质是什么？",
        options: [
          { optionText: "卓越领导力", geneWeights: { "RUNX2": 0.20, "KITLG": 0.15, "TYRP1": 0.10 } },
          { optionText: "亲和力与温暖", geneWeights: { "PAX3": 0.20, "MC1R": 0.15, "ASIP": 0.10 } },
          { optionText: "独立思考和创新", geneWeights: { "TBX15": 0.20, "SOX9": 0.15, "MITF": 0.10 } },
          { optionText: "敏锐观察与洞察", geneWeights: { "DCHS2": 0.20, "SLC45A2": 0.15, "OCA2/HERC2": 0.10 } }
        ]
      }
    ];
    
    // 随机排列整个题库
    shuffle(allQuestions);
    // 动态选择前 12 道题进行问卷（从 15 道题目中随机抽取 12 道）
    const questions = allQuestions.slice(0, 12);
    
    // 同时随机排列每道题的选项顺序
    questions.forEach(q => shuffle(q.options));
    
    // 用于存储每题答案的数组
    const userAnswers = new Array(questions.length).fill(null);
    
    let currentQuestionIndex = 0;
    
    // 渲染当前问题
    function renderCurrentQuestion() {
      const container = document.getElementById("questionContainer");
      container.innerHTML = "";
      const currentQ = questions[currentQuestionIndex];
      
      // 显示题号和问题内容
      const header = document.createElement("h2");
      header.textContent = `题目 ${currentQuestionIndex + 1}: ${currentQ.questionText}`;
      container.appendChild(header);
      
      // 显示选项
      const optionsDiv = document.createElement("div");
      optionsDiv.className = "options";
      currentQ.options.forEach((option, index) => {
        const label = document.createElement("label");
        label.className = "option-label";
        const radio = document.createElement("input");
        radio.type = "radio";
        radio.name = "question_" + currentQ.id;
        radio.value = index;
        if (userAnswers[currentQuestionIndex] === index) {
          radio.checked = true;
        }
        label.appendChild(radio);
        label.appendChild(document.createTextNode(" " + option.optionText));
        optionsDiv.appendChild(label);
      });
      container.appendChild(optionsDiv);
      updateNavButtons();
    }
    
    // 更新导航按钮显示
    function updateNavButtons() {
      const prevButton = document.getElementById("prevButton");
      const nextButton = document.getElementById("nextButton");
      prevButton.style.display = currentQuestionIndex > 0 ? "inline-block" : "none";
      nextButton.textContent = (currentQuestionIndex === questions.length - 1) ? "提交" : "下一题";
    }
    
    // 保存当前问题答案
    function saveCurrentAnswer() {
      const radios = document.getElementsByName("question_" + questions[currentQuestionIndex].id);
      let selected = null;
      for (let radio of radios) {
        if (radio.checked) {
          selected = parseInt(radio.value);
          break;
        }
      }
      userAnswers[currentQuestionIndex] = selected;
    }
    
    // 导航按钮事件处理
    document.getElementById("prevButton").addEventListener("click", function() {
      saveCurrentAnswer();
      if (currentQuestionIndex > 0) {
        currentQuestionIndex--;
        renderCurrentQuestion();
      }
    });
    
    document.getElementById("nextButton").addEventListener("click", function() {
      saveCurrentAnswer();
      if (userAnswers[currentQuestionIndex] === null) {
        alert("请先选择一个选项！");
        return;
      }
      if (currentQuestionIndex < questions.length - 1) {
        currentQuestionIndex++;
        renderCurrentQuestion();
      } else {
        processSubmission();
      }
    });
    
    // 处理提交：累计所有选项对应的基因权重，并显示结果
    function processSubmission() {
      let geneResults = {};
      // 初始化所有目标基因得分
      const fullGeneList = [
        "PAX3", "EDAR", "MC1R", "RUNX2", "OCA2/HERC2", "SLC24A5",
        "TBX15", "SOX9", "BMP2", "DCHS2", "SLC45A2", "GDF6",
        "BNC2", "KITLG", "IRF4", "TYR", "TYRP1", "ASIP", "MITF"
      ];
      fullGeneList.forEach(gene => { geneResults[gene] = 0; });
      
      // 遍历每题，累计选项对应的基因权重
      questions.forEach((q, qIndex) => {
        const answerIndex = userAnswers[qIndex];
        if (answerIndex !== null) {
          const selectedOption = q.options[answerIndex];
          for (const gene in selectedOption.geneWeights) {
            geneResults[gene] += selectedOption.geneWeights[gene];
          }
        }
      });
      
      // 隐藏问卷显示区域和导航按钮区
      document.getElementById("questionContainer").style.display = "none";
      document.querySelector(".nav-buttons").style.display = "none";
      
      // 显示累计结果
      const resultContainer = document.getElementById("resultContainer");
      let html = "<h2>问卷结果（基因权重累计）</h2><ul>";
      for (const gene in geneResults) {
        html += `<li>${gene}: ${geneResults[gene].toFixed(2)}</li>`;
      }
      html += "</ul>";
      resultContainer.innerHTML = html;
      resultContainer.style.display = "block";
    }
    
    // 初次渲染当前问题
    renderCurrentQuestion();
  </script>
</body>
</html>