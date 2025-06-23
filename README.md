<!DOCTYPE html>
<html lang="zh">
<head>
  <meta charset="UTF-8">
  <title>动态趣味问卷</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
    }
    h1 {
      text-align: center;
    }
    .question {
      margin-bottom: 20px;
      padding: 10px;
      border: 1px solid #ddd;
    }
    .question h3 {
      margin: 0 0 10px;
      font-size: 18px;
    }
    .option {
      margin: 5px 0;
      display: block;
    }
    .submit-btn {
      margin-top: 20px;
      padding: 10px 20px;
      font-size: 16px;
      cursor: pointer;
    }
    .result {
      margin-top: 30px;
      border: 1px solid #ccc;
      padding: 10px;
    }
  </style>
</head>
<body>

<h1>动态趣味问卷</h1>
<form id="questionnaireForm">
  <div id="questionsContainer"></div>
  <button type="submit" class="submit-btn">提交问卷</button>
</form>
<div id="resultContainer" class="result"></div>

<script>
// Shuffle 数组的函数（Fisher-Yates 算法）
function shuffle(array) {
  for (let i = array.length - 1; i > 0; i--) {
    let j = Math.floor(Math.random() * (i + 1));
    [array[i], array[j]] = [array[j], array[i]];
  }
}

// 定义问卷数据，所有 12 题，包含每个题目的描述、选项文本及对应的基因权重（数值仅为示例）
const questions = [
  {
    id: 1,
    questionText: "如果你的内心世界可以用颜色来表现，你最倾向于哪种色彩组合？",
    options: [
      { optionText: "温暖红与活力橙", geneWeights: { PAX3: 0.25, EDAR: 0.20, MC1R: 0.15 } },
      { optionText: "宁静蓝与深邃紫", geneWeights: { RUNX2: 0.20, "OCA2/HERC2": 0.25, SLC24A5: 0.15 } },
      { optionText: "生机绿与明快黄", geneWeights: { TBX15: 0.20, SOX9: 0.20, BMP2: 0.15 } },
      { optionText: "神秘黑与闪耀银", geneWeights: { DCHS2: 0.20, SLC45A2: 0.15, GDF6: 0.15 } }
    ]
  },
  {
    id: 2,
    questionText: "假如你能乘坐一种神奇的交通工具穿越时空，你最向往哪种体验？",
    options: [
      { optionText: "时光机 —— 探索过去与未来", geneWeights: { RUNX2: 0.20, EDAR: 0.15, "OCA2/HERC2": 0.15 } },
      { optionText: "热气球 —— 随风漫游的浪漫之旅", geneWeights: { PAX3: 0.20, SOX9: 0.15, MC1R: 0.10 } },
      { optionText: "飞毯 —— 魔法与神秘并存的飞行", geneWeights: { TBX15: 0.25, SLC24A5: 0.10, BMP2: 0.15 } },
      { optionText: "潜水艇 —— 探索深邃未知的海域", geneWeights: { DCHS2: 0.20, SLC45A2: 0.15, GDF6: 0.15 } }
    ]
  },
  {
    id: 3,
    questionText: "一天结束后，你最希望沉浸在怎样的音乐风格中来平复心情？",
    options: [
      { optionText: "悠扬古典 —— 如同缓缓流动的乐章", geneWeights: { RUNX2: 0.15, EDAR: 0.20, BMP2: 0.10 } },
      { optionText: "流行旋律 —— 节奏明快且充满活力", geneWeights: { MC1R: 0.15, SLC24A5: 0.20, TBX15: 0.10 } },
      { optionText: "梦幻电子 —— 科幻律动、节奏交织", geneWeights: { SLC45A2: 0.20, "OCA2/HERC2": 0.15, GDF6: 0.10 } },
      { optionText: "温馨民谣 —— 讲述温柔且真实的故事", geneWeights: { PAX3: 0.15, SOX9: 0.20, DCHS2: 0.10 } }
    ]
  },
  {
    id: 4,
    questionText: "面对重大抉择时，你通常依赖哪种方式作出决定？",
    options: [
      { optionText: "依靠内心直觉", geneWeights: { SOX9: 0.20, EDAR: 0.15, GDF6: 0.10 } },
      { optionText: "仔细逻辑分析", geneWeights: { RUNX2: 0.25, TBX15: 0.15, BMP2: 0.10 } },
      { optionText: "倾听亲友建议", geneWeights: { PAX3: 0.20, MC1R: 0.15, SLC24A5: 0.10 } },
      { optionText: "偶尔随机冒险、听天由命", geneWeights: { DCHS2: 0.20, SLC45A2: 0.15, "OCA2/HERC2": 0.10 } }
    ]
  },
  {
    id: 5,
    questionText: "如果让你的灵魂独自去旅行，你最向往下列哪个场景？",
    options: [
      { optionText: "沉寂幽深的古林", geneWeights: { TBX15: 0.20, SOX9: 0.15, EDAR: 0.10 } },
      { optionText: "闪烁霓虹的都市夜景", geneWeights: { MC1R: 0.20, "OCA2/HERC2": 0.15, SLC24A5: 0.10 } },
      { optionText: "宁静辽阔的海边日落", geneWeights: { RUNX2: 0.20, BMP2: 0.15, SLC45A2: 0.10 } },
      { optionText: "清新自然的山谷风光", geneWeights: { PAX3: 0.20, DCHS2: 0.15, GDF6: 0.10 } }
    ]
  },
  {
    id: 6,
    questionText: "你是否会突然产生创意闪现，就像一道灵光划破黑暗？",
    options: [
      { optionText: "经常有，创意如泉涌", geneWeights: { TBX15: 0.20, EDAR: 0.15, GDF6: 0.10 } },
      { optionText: "偶尔突现灵感", geneWeights: { RUNX2: 0.15, SLC24A5: 0.20, BMP2: 0.10 } },
      { optionText: "很少体验这种瞬间", geneWeights: { PAX3: 0.15, MC1R: 0.20, SOX9: 0.10 } },
      { optionText: "基本没有，习惯循规蹈矩", geneWeights: { DCHS2: 0.15, SLC45A2: 0.20, "OCA2/HERC2": 0.10 } }
    ]
  },
  {
    id: 7,
    questionText: "在作出决策时，你觉得自己的思维节奏更倾向于哪种？",
    options: [
      { optionText: "稳定平缓，思维沉稳", geneWeights: { RUNX2: 0.20, EDAR: 0.15, BMP2: 0.10 } },
      { optionText: "快速直率，反应迅捷", geneWeights: { MC1R: 0.20, TBX15: 0.15, SLC24A5: 0.10 } },
      { optionText: "缓慢深思，反复推敲", geneWeights: { PAX3: 0.20, SOX9: 0.15, GDF6: 0.10 } },
      { optionText: "起伏波动，难以预测", geneWeights: { DCHS2: 0.20, SLC45A2: 0.15, "OCA2/HERC2": 0.10 } }
    ]
  },
  {
    id: 8,
    questionText: "在日常生活中，你对尝试新奇体验和探索未知事物的渴望程度如何？",
    options: [
      { optionText: "总是充满好奇，积极尝试新事物", geneWeights: { TBX15: 0.20, EDAR: 0.15, SLC24A5: 0.10 } },
      { optionText: "偶尔寻求改变和尝试", geneWeights: { PAX3: 0.20, MC1R: 0.15, SLC45A2: 0.10 } },
      { optionText: "更偏好稳定熟悉的环境", geneWeights: { RUNX2: 0.20, "OCA2/HERC2": 0.15, BMP2: 0.10 } },
      { optionText: "谨慎小心，很少主动冒险", geneWeights: { DCHS2: 0.20, SOX9: 0.15, GDF6: 0.10 } }
    ]
  },
  {
    id: 9,
    questionText: "假如你能为内心绘制一张地图，你觉得最核心的区域最能象征什么？",
    options: [
      { optionText: "一片宁静的湖泊", geneWeights: { RUNX2: 0.15, EDAR: 0.20, BMP2: 0.10 } },
      { optionText: "茂密神秘的森林", geneWeights: { PAX3: 0.15, TBX15: 0.20, SOX9: 0.10 } },
      { optionText: "繁华热闹的市区", geneWeights: { MC1R: 0.15, SLC24A5: 0.20, "OCA2/HERC2": 0.10 } },
      { optionText: "广袤自由的原野", geneWeights: { DCHS2: 0.15, SLC45A2: 0.20, GDF6: 0.10 } }
    ]
  },
  {
    id: 10,
    questionText: "在你的梦境中，是否总会反复出现某个符号或图像，让你觉得意味深长？",
    options: [
      { optionText: "闪烁的星辰", geneWeights: { TBX15: 0.20, EDAR: 0.15, "OCA2/HERC2": 0.10 } },
      { optionText: "温柔的月亮", geneWeights: { PAX3: 0.20, MC1R: 0.15, SLC24A5: 0.10 } },
      { optionText: "独特的螺旋图案", geneWeights: { RUNX2: 0.20, SLC45A2: 0.15, BMP2: 0.10 } },
      { optionText: "抽象交织的形状", geneWeights: { DCHS2: 0.20, SOX9: 0.15, GDF6: 0.10 } }
    ]
  },
  {
    id: 11,
    questionText: "对你来说，未来最理想的故事情节会是什么样的？",
    options: [
      { optionText: "充满科幻冒险、突破常规", geneWeights: { RUNX2: 0.15, EDAR: 0.20, BMP2: 0.10 } },
      { optionText: "如童话般梦幻美好", geneWeights: { PAX3: 0.15, TBX15: 0.20, SOX9: 0.10 } },
      { optionText: "深沉古朴、蕴含智慧哲理", geneWeights: { MC1R: 0.15, SLC24A5: 0.20, "OCA2/HERC2": 0.10 } },
      { optionText: "刺激传奇、充满未知奇遇", geneWeights: { DCHS2: 0.15, SLC45A2: 0.20, GDF6: 0.10 } }
    ]
  },
  {
    id: 12,
    questionText: "如果将你内心深处那段刻骨铭心的经历比作一场剧场表演，你认为它更像哪一种风格？",
    options: [
      { optionText: "盛大史诗", geneWeights: { RUNX2: 0.20, EDAR: 0.15, TBX15: 0.10 } },
      { optionText: "温情浪漫", geneWeights: { PAX3: 0.20, MC1R: 0.15, SOX9: 0.10 } },
      { optionText: "神秘幻想", geneWeights: { DCHS2: 0.20, SLC45A2: 0.15, "OCA2/HERC2": 0.10 } },
      { optionText: "孤独独角戏", geneWeights: { GDF6: 0.20, BMP2: 0.15, SLC24A5: 0.10 } }
    ]
  }
];

// 随机排列题目顺序
shuffle(questions);
// 同时随机排列每道题的选项顺序
questions.forEach(q => shuffle(q.options));

// 渲染问卷到页面中
function renderQuestions() {
  const container = document.getElementById("questionsContainer");
  container.innerHTML = "";
  questions.forEach((q, index) => {
    // 创建题目容器
    const questionDiv = document.createElement("div");
    questionDiv.className = "question";
    // 标题
    const questionHeader = document.createElement("h3");
    questionHeader.textContent = `题目 ${index + 1}: ${q.questionText}`;
    questionDiv.appendChild(questionHeader);
    // 选项
    const optionsDiv = document.createElement("div");
    q.options.forEach((option, optIndex) => {
      const label = document.createElement("label");
      label.className = "option";
      const radio = document.createElement("input");
      radio.type = "radio";
      radio.name = "question_" + q.id;
      radio.value = optIndex; // 存储选项的索引值
      label.appendChild(radio);
      label.appendChild(document.createTextNode(" " + option.optionText));
      optionsDiv.appendChild(label);
    });
    questionDiv.appendChild(optionsDiv);
    container.appendChild(questionDiv);
  });
}

renderQuestions();

// 处理表单提交，累加所有选项对应的基因权重
document.getElementById("questionnaireForm").addEventListener("submit", function(e) {
  e.preventDefault();
  // 初始化所有目标基因的累计得分
  const geneList = ["PAX3", "EDAR", "MC1R", "RUNX2", "OCA2/HERC2", "SLC24A5", "TBX15", "SOX9", "BMP2", "DCHS2", "SLC45A2", "GDF6"];
  let geneResults = {};
  geneList.forEach(gene => { geneResults[gene] = 0; });
  
  // 遍历每道题，获取选中选项
  questions.forEach(q => {
    const radios = document.getElementsByName("question_" + q.id);
    let selectedIndex = -1;
    radios.forEach(radio => {
      if (radio.checked) {
        selectedIndex = parseInt(radio.value);
      }
    });
    if (selectedIndex >= 0) {
      const selectedOption = q.options[selectedIndex];
      // 累加各基因的权重
      for (const gene in selectedOption.geneWeights) {
        geneResults[gene] += selectedOption.geneWeights[gene];
      }
    }
  });
  
  // 显示结果
  const resultContainer = document.getElementById("resultContainer");
  let resultHTML = "<h2>问卷结果（基因权重累计）</h2><ul>";
  for (const gene in geneResults) {
    resultHTML += `<li>${gene}: ${geneResults[gene].toFixed(2)}</li>`;
  }
  resultHTML += "</ul>";
  resultContainer.innerHTML = resultHTML;
});
</script>

</body>
</html>