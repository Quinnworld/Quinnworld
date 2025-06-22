<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>外貌基因型智能推断测评</title>
<style>
  body { max-width: 600px; margin: 20px auto; font-family: "微软雅黑", sans-serif; background: #fff; padding: 20px; color:#000; }
  h2,h3 { text-align:center; }
  label, select, input[type=number] { width: 100%; margin:10px 0 15px; font-size:16px; }
  select, input[type=number] { padding:8px; border-radius:6px; border:1px solid #ccc; }
  button { width:48%; padding:12px; font-size:16px; border:none; border-radius:6px; cursor:pointer; color:#fff; background:#4338ca; margin:5px 1%; box-sizing:border-box; }
  button:disabled { background:#a5b4fc; cursor:not-allowed; }
  .option { background:#fff; border:1px solid #bbb; border-radius:6px; padding:10px; margin:6px 0; cursor:pointer; user-select:none; color:#000; transition: background-color 0.3s, color 0.3s; }
  .option.selected { background:#4f46e5; color:#fff; border-color:#4338ca; }
  #paySection { text-align:center; margin: 15px 0; }
  #paySection img { width: 200px; border:1px solid #ccc; border-radius:8px; }
  pre { white-space: pre-wrap; word-break: break-word; background:#eee; padding:10px; border-radius:6px; font-family: monospace; user-select: all; cursor: default; }
</style>
</head>
<body>

<h2>外貌基因型智能推断测评</h2>

<div id="sectionUserInfo">
  <label for="inputAge">年龄</label>
  <input type="number" id="inputAge" value="25" min="0" />
  <label for="selectGender">性别</label>
  <select id="selectGender">
    <option value="male">男性</option>
    <option value="female">女性</option>
    <option value="other">其他</option>
  </select>
  <button id="btnStart">开始答题</button>
</div>

<div id="sectionQuiz" style="display:none;">
  <div id="questionText" style="font-weight:700; margin-bottom:10px;"></div>
  <div id="optionsContainer"></div>
  <button id="btnPrev" disabled>上一题</button>
  <button id="btnNext" disabled>下一题</button>
  <div id="progressText" style="text-align:center; margin-top:5px; color:#555;"></div>
</div>

<div id="sectionResult" style="display:none;">
  <h3>基因型推断结果</h3>
  <pre id="geneResult"></pre>
  <h3>基础发展报告（免费）</h3>
  <pre id="basicReport"></pre>
  <h3>深度发展报告（付费）</h3>
  <pre id="premiumReport" title="点击生成深度报告" style="background:#fff0f0; color:#888; cursor:pointer;">点击生成深度报告</pre>
  <div id="paySection" style="display:none;">
    <p>请扫码付款解锁深度报告</p>
    <img src="https://pplx-res.cloudinary.com/image/upload/v1750588444/user_uploads/19241329/9b7936f2-1eeb-4500-99b9-9ddc57f69890/mm_facetoface_collect_qrcode_1750513571858.jpg" alt="收款码" />
    <br/>
    <button id="btnPaidConfirm" style="background:#10b981; margin-top:10px;">我已付款，解锁报告</button>
  </div>
  <button id="btnRestart" style="margin-top:20px;">重新开始</button>
</div>

<script>
// 洗牌算法
function shuffle(arr) {
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [arr[i], arr[j]] = [arr[j], arr[i]];
  }
  return arr;
}

// 12道外貌相关基因题库（示例，真实应用建议结合专业遗传学文献完善）
const geneQuestions = [
  {
    id: "Q1", gene: "MC1R", text: "你的头发颜色最接近？",
    options: [
      { text: "深黑/黑色", genotype: "MC1R:WT/WT", score: 2 },
      { text: "棕色", genotype: "MC1R:WT/V", score: 1 },
      { text: "深棕/浅棕", genotype: "MC1R:V/V", score: 0 },
      { text: "红色/偏红", genotype: "MC1R:V/V", score: -1 }
    ]
  },
  {
    id: "Q2", gene: "OCA2", text: "你的眼睛颜色？",
    options: [
      { text: "深棕/黑色", genotype: "OCA2:AA", score: 2 },
      { text: "棕色", genotype: "OCA2:AG", score: 1 },
      { text: "浅褐/绿色", genotype: "OCA2:GG", score: 0 },
      { text: "蓝色/灰色", genotype: "OCA2:GG", score: -1 }
    ]
  },
  {
    id: "Q3", gene: "SLC45A2", text: "你的皮肤在夏天暴晒后？",
    options: [
      { text: "很容易晒黑", genotype: "SLC45A2:AA", score: 2 },
      { text: "容易晒黑", genotype: "SLC45A2:AG", score: 1 },
      { text: "不易晒黑", genotype: "SLC45A2:GG", score: 0 },
      { text: "晒不黑，易晒伤", genotype: "SLC45A2:GG", score: -1 }
    ]
  },
  {
    id: "Q4", gene: "EDAR", text: "你的头发质地？",
    options: [
      { text: "粗硬直发", genotype: "EDAR:AA", score: 2 },
      { text: "偏硬直发", genotype: "EDAR:AG", score: 1 },
      { text: "柔软直发", genotype: "EDAR:GG", score: 0 },
      { text: "自然卷/波浪", genotype: "EDAR:GG", score: -1 }
    ]
  },
  {
    id: "Q5", gene: "FGFR2", text: "你鼻梁高度？",
    options: [
      { text: "高鼻梁", genotype: "FGFR2:AA", score: 2 },
      { text: "较高", genotype: "FGFR2:AG", score: 1 },
      { text: "中等", genotype: "FGFR2:GG", score: 0 },
      { text: "低鼻梁", genotype: "FGFR2:GG", score: -1 }
    ]
  },
  {
    id: "Q6", gene: "ABCC11", text: "你的耳垢类型？",
    options: [
      { text: "湿耳垢", genotype: "ABCC11:GG", score: 2 },
      { text: "偏湿", genotype: "ABCC11:GA", score: 1 },
      { text: "偏干", genotype: "ABCC11:AA", score: 0 },
      { text: "干耳垢", genotype: "ABCC11:AA", score: -1 }
    ]
  },
  {
    id: "Q7", gene: "ASIP", text: "你皮肤色素沉着情况？",
    options: [
      { text: "色素较深", genotype: "ASIP:AA", score: 2 },
      { text: "中等", genotype: "ASIP:AG", score: 1 },
      { text: "较浅", genotype: "ASIP:GG", score: 0 },
      { text: "极浅", genotype: "ASIP:GG", score: -1 }
    ]
  },
  {
    id: "Q8", gene: "TYR", text: "你有雀斑吗？",
    options: [
      { text: "明显", genotype: "TYR:AA", score: 2 },
      { text: "有一些", genotype: "TYR:AG", score: 1 },
      { text: "很少", genotype: "TYR:GG", score: 0 },
      { text: "基本没有", genotype: "TYR:GG", score: -1 }
    ]
  },
  {
    id: "Q9", gene: "HERC2", text: "你家族中有蓝眼睛成员吗？",
    options: [
      { text: "有且比例高", genotype: "HERC2:GG", score: 2 },
      { text: "有部分", genotype: "HERC2:AG", score: 1 },
      { text: "极少", genotype: "HERC2:AA", score: 0 },
      { text: "没有", genotype: "HERC2:AA", score: -1 }
    ]
  },
  {
    id: "Q10", gene: "SLC24A5", text: "你的肤色属于？",
    options: [
      { text: "深色", genotype: "SLC24A5:AA", score: 2 },
      { text: "中等", genotype: "SLC24A5:AG", score: 1 },
      { text: "浅色", genotype: "SLC24A5:GG", score: 0 },
      { text: "非常浅", genotype: "SLC24A5:GG", score: -1 }
    ]
  },
  {
    id: "Q11", gene: "KRT71", text: "你的头发卷曲程度？",
    options: [
      { text: "非常卷", genotype: "KRT71:TT", score: 2 },
      { text: "中度卷", genotype: "KRT71:CT", score: 1 },
      { text: "微卷", genotype: "KRT71:CC", score: 0 },
      { text: "直发", genotype: "KRT71:CC", score: -1 }
    ]
  },
  {
    id: "Q12", gene: "TP63", text: "你的唇型属于？",
    options: [
      { text: "厚唇", genotype: "TP63:AA", score: 2 },
      { text: "中厚", genotype: "TP63:AG", score: 1 },
      { text: "中等", genotype: "TP63:GG", score: 0 },
      { text: "薄唇", genotype: "TP63:GG", score: -1 }
    ]
  }
];

let currentIndex = 0;
let selectedOptions = [];
let shuffledQuestions = [];
let shuffledOptionsList = [];
let geneScores = {};
let geneTypeResult = {};
let userHasPaid = false;

const sectionUserInfo = document.getElementById("sectionUserInfo");
const sectionQuiz = document.getElementById("sectionQuiz");
const sectionResult = document.getElementById("sectionResult");
const qTextEl = document.getElementById("questionText");
const optionsEl = document.getElementById("optionsContainer");
const btnNext = document.getElementById("btnNext");
const btnPrev = document.getElementById("btnPrev");
const progressText = document.getElementById("progressText");
const geneResultEl = document.getElementById("geneResult");
const basicReportEl = document.getElementById("basicReport");
const premiumReportEl = document.getElementById("premiumReport");
const btnStart = document.getElementById("btnStart");
const btnRestart = document.getElementById("btnRestart");
const paySection = document.getElementById("paySection");
const btnPaidConfirm = document.getElementById("btnPaidConfirm");

function selectOption(idx) {
  selectedOptions[currentIndex] = idx;
  Array.from(optionsEl.children).forEach((el,i) => {
    el.classList.toggle("selected", i === idx);
  });
  btnNext.disabled = false;
}

function renderQuestion(index) {
  currentIndex = index;
  const q = shuffledQuestions[index];
  qTextEl.textContent = q.text;
  optionsEl.innerHTML = "";
  btnNext.disabled = selectedOptions[index] == null;
  btnPrev.disabled = index === 0;
  let options = shuffledOptionsList[index];
  if (!options) {
    options = shuffle(q.options.map((opt, i) => ({ ...opt, origIdx: i })));
    shuffledOptionsList[index] = options;
  }
  for(let i=0;i<options.length;i++) {
    const opt = document.createElement("div");
    opt.className = "option";
    opt.textContent = options[i].text;
    opt.onclick = () => selectOption(i);
    if(selectedOptions[index] === i) opt.classList.add("selected");
    optionsEl.appendChild(opt);
  }
  progressText.textContent = `第 ${index+1} / ${shuffledQuestions.length} 题`;
}

function calculateGeneScores() {
  geneScores = {};
  shuffledQuestions.forEach((q, qIdx) => {
    const optIdx = selectedOptions[qIdx];
    if(optIdx == null) return;
    const origIdx = shuffledOptionsList[qIdx][optIdx].origIdx;
    const opt = q.options[origIdx];
    geneScores[q.gene] = (geneScores[q.gene] || 0) + opt.score;
  });

  // 联动/交互效应举例：如果Q1和Q6都选高分，MC1R和ABCC11的变异概率提升
  // 这里只做简单示范，实际可用更复杂的贝叶斯/决策树
  if(selectedOptions[0]!=null && selectedOptions[5]!=null){
    const q1Score = shuffledOptionsList[0][selectedOptions[0]].score;
    const q6Score = shuffledOptionsList[5][selectedOptions[5]].score;
    if(q1Score>=1 && q6Score>=1) geneScores["MC1R"] += 1;
  }
}

function inferGeneTypes() {
  geneTypeResult = {};
  // 简化推断规则：得分>=2为变异型，否则为野生型或常见型
  for (let gene in geneScores) {
    if (geneScores[gene] >= 2) {
      geneTypeResult[gene] = "变异型/特殊型";
    } else if (geneScores[gene] >= 1) {
      geneTypeResult[gene] = "杂合型/部分变异";
    } else {
      geneTypeResult[gene] = "常见型/野生型";
    }
  }
}

function generateGeneResultText() {
  let text = "";
  for (let i = 0; i < shuffledQuestions.length; i++) {
    const q = shuffledQuestions[i];
    text += `${q.gene}：${geneTypeResult[q.gene]}\n`;
  }
  return text;
}

function generateBasicReport() {
  let report = "";
  for (let gene in geneTypeResult) {
    if (gene === "MC1R") {
      report += geneTypeResult[gene] === "变异型/特殊型"
        ? "MC1R变异型：红发或浅色发色倾向，皮肤较白，防晒需加强。\n"
        : "MC1R常见型：深色发色，紫外线抵抗力较好。\n";
    }
    if (gene === "OCA2") {
      report += geneTypeResult[gene] === "变异型/特殊型"
        ? "OCA2变异型：浅色（蓝/灰/绿）眼睛概率高，注意眼部防护。\n"
        : "OCA2常见型：深色眼睛，黑色素较多。\n";
    }
    if (gene === "SLC45A2") {
      report += geneTypeResult[gene] === "变异型/特殊型"
        ? "SLC45A2变异型：皮肤较白，易晒伤，注意防晒。\n"
        : "SLC45A2常见型：皮肤偏深，耐晒能力强。\n";
    }
    if (gene === "EDAR") {
      report += geneTypeResult[gene] === "变异型/特殊型"
        ? "EDAR变异型：头发粗硬直，汗腺发达，牙齿形态特殊。\n"
        : "EDAR常见型：头发柔软，汗腺发育一般。\n";
    }
    if (gene === "FGFR2") {
      report += geneTypeResult[gene] === "变异型/特殊型"
        ? "FGFR2变异型：高鼻梁特征明显，面部立体感强。\n"
        : "FGFR2常见型：鼻梁中等或偏低，五官柔和。\n";
    }
    if (gene === "ABCC11") {
      report += geneTypeResult[gene] === "变异型/特殊型"
        ? "ABCC11变异型：湿耳垢型，腋臭概率高，注意个人护理。\n"
        : "ABCC11常见型：干耳垢，体味轻。\n";
    }
    if (gene === "ASIP") {
      report += geneTypeResult[gene] === "变异型/特殊型"
        ? "ASIP变异型：肤色较深，色素沉着明显。\n"
        : "ASIP常见型：肤色浅，色素沉着少。\n";
    }
    if (gene === "TYR") {
      report += geneTypeResult[gene] === "变异型/特殊型"
        ? "TYR变异型：雀斑概率高，注意皮肤护理和防晒。\n"
        : "TYR常见型：雀斑少，皮肤均匀。\n";
    }
    if (gene === "HERC2") {
      report += geneTypeResult[gene] === "变异型/特殊型"
        ? "HERC2变异型：蓝眼睛概率高，注意光线保护。\n"
        : "HERC2常见型：棕色或深色眼睛。\n";
    }
    if (gene === "SLC24A5") {
      report += geneTypeResult[gene] === "变异型/特殊型"
        ? "SLC24A5变异型：皮肤极浅，需注意紫外线防护。\n"
        : "SLC24A5常见型：肤色中等或偏深。\n";
    }
    if (gene === "KRT71") {
      report += geneTypeResult[gene] === "变异型/特殊型"
        ? "KRT71变异型：头发卷曲度高，易打理。\n"
        : "KRT71常见型：头发直或微卷。\n";
    }
    if (gene === "TP63") {
      report += geneTypeResult[gene] === "变异型/特殊型"
        ? "TP63变异型：厚唇，面部立体感强。\n"
        : "TP63常见型：薄唇或中等唇型。\n";
    }
  }
  return report;
}

function generatePremiumReport() {
  // 深度报告可结合更多交互、遗传风险、健康建议等
  return "这是基于你答题结果和最新外貌基因科学的深度发展报告，涵盖健康、护理、个性化美学建议等内容。\n（如需定制，请联系专业遗传咨询师）";
}

btnStart.onclick = () => {
  const age = parseInt(document.getElementById("inputAge").value);
  if(isNaN(age) || age < 0) { alert("请输入有效年龄"); return; }
  selectedOptions = [];
  shuffledQuestions = shuffle(geneQuestions.map(q => ({...q, options: q.options.slice()})));
  shuffledOptionsList = [];
  currentIndex = 0;
  sectionUserInfo.style.display = "none";
  sectionQuiz.style.display = "block";
  sectionResult.style.display = "none";
  renderQuestion(currentIndex);
};

btnNext.onclick = () => {
  if(selectedOptions[currentIndex] == null) {
    alert("请选择一个选项");
    return;
  }
  if(currentIndex < shuffledQuestions.length - 1) {
    renderQuestion(currentIndex + 1);
  } else {
    calculateGeneScores();
    inferGeneTypes();
    renderResult();
  }
};

btnPrev.onclick = () => {
  if(currentIndex > 0) renderQuestion(currentIndex - 1);
};

function renderResult() {
  sectionQuiz.style.display = "none";
  sectionResult.style.display = "block";
  userHasPaid = false;
  paySection.style.display = "none";
  geneResultEl.textContent = generateGeneResultText();
  basicReportEl.textContent = generateBasicReport();
  premiumReportEl.textContent = "点击生成深度报告";
}

premiumReportEl.onclick = () => {
  if(userHasPaid) {
    premiumReportEl.textContent = generatePremiumReport();
    paySection.style.display = "none";
  } else {
    paySection.style.display = "block";
    premiumReportEl.textContent = "请扫码付款解锁深度报告";
  }
};

btnPaidConfirm.onclick = () => {
  userHasPaid = true;
  alert("付款确认成功，深度报告已解锁！");
  premiumReportEl.textContent = generatePremiumReport();
  paySection.style.display = "none";
};

btnRestart.onclick = () => {
  sectionUserInfo.style.display = "block";
  sectionQuiz.style.display = "none";
  sectionResult.style.display = "none";
  userHasPaid = false;
  paySection.style.display = "none";
};
</script>
</body>
</html>
