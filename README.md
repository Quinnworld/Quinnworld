<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>趣味个性发展报告</title>
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

// 12题外貌基因题库（每选项可对应多个基因型和权重）
const geneQuestionBank = [
  {
    question: "你的头发颜色？",
    options: [
      { text: "黑色", genotypes: { MC1R: { "WT/WT": 2, "WT/V": 1 }, OCA2: { "AA": 1 } } },
      { text: "棕色", genotypes: { MC1R: { "WT/V": 2, "V/V": 1 }, OCA2: { "AG": 1 } } },
      { text: "金色", genotypes: { MC1R: { "V/V": 2 }, OCA2: { "GG": 2 } } },
      { text: "红色", genotypes: { MC1R: { "V/V": 3 }, OCA2: { "GG": 1 } } }
    ]
  },
  {
    question: "你的眼睛颜色？",
    options: [
      { text: "黑色", genotypes: { OCA2: { "AA": 2 }, HERC2: { "AA": 2 } } },
      { text: "棕色", genotypes: { OCA2: { "AG": 2 }, HERC2: { "AG": 2 } } },
      { text: "绿色", genotypes: { OCA2: { "GG": 2 }, HERC2: { "GG": 1 } } },
      { text: "蓝色", genotypes: { OCA2: { "GG": 3 }, HERC2: { "GG": 2 } } }
    ]
  },
  {
    question: "你的皮肤在夏天暴晒后？",
    options: [
      { text: "很容易晒黑", genotypes: { SLC45A2: { "AA": 2 }, SLC24A5: { "AA": 1 } } },
      { text: "容易晒黑", genotypes: { SLC45A2: { "AG": 2 }, SLC24A5: { "AG": 1 } } },
      { text: "不易晒黑", genotypes: { SLC45A2: { "GG": 2 }, SLC24A5: { "GG": 1 } } },
      { text: "晒不黑，易晒伤", genotypes: { SLC45A2: { "GG": 2 }, SLC24A5: { "GG": 2 } } }
    ]
  },
  {
    question: "你的头发质地？",
    options: [
      { text: "粗硬直发", genotypes: { EDAR: { "AA": 2 }, KRT71: { "CC": 1 } } },
      { text: "偏硬直发", genotypes: { EDAR: { "AG": 2 }, KRT71: { "CT": 1 } } },
      { text: "柔软直发", genotypes: { EDAR: { "GG": 2 }, KRT71: { "TT": 1 } } },
      { text: "自然卷/波浪", genotypes: { EDAR: { "GG": 2 }, KRT71: { "TT": 2 } } }
    ]
  },
  {
    question: "你的鼻梁高度？",
    options: [
      { text: "高鼻梁", genotypes: { FGFR2: { "AA": 2 }, PAX3: { "AA": 1 } } },
      { text: "较高", genotypes: { FGFR2: { "AG": 2 }, PAX3: { "AG": 1 } } },
      { text: "中等", genotypes: { FGFR2: { "GG": 2 }, PAX3: { "GG": 1 } } },
      { text: "低鼻梁", genotypes: { FGFR2: { "GG": 2 }, PAX3: { "GG": 2 } } }
    ]
  },
  {
    question: "你的耳垢类型？",
    options: [
      { text: "湿耳垢", genotypes: { ABCC11: { "GG": 2 } } },
      { text: "偏湿", genotypes: { ABCC11: { "GA": 2 } } },
      { text: "偏干", genotypes: { ABCC11: { "AA": 1 } } },
      { text: "干耳垢", genotypes: { ABCC11: { "AA": 2 } } }
    ]
  },
  {
    question: "你皮肤色素沉着情况？",
    options: [
      { text: "色素较深", genotypes: { ASIP: { "AA": 2 } } },
      { text: "中等", genotypes: { ASIP: { "AG": 2 } } },
      { text: "较浅", genotypes: { ASIP: { "GG": 1 } } },
      { text: "极浅", genotypes: { ASIP: { "GG": 2 } } }
    ]
  },
  {
    question: "你有雀斑吗？",
    options: [
      { text: "明显", genotypes: { TYR: { "AA": 2 }, IRF4: { "TT": 1 } } },
      { text: "有一些", genotypes: { TYR: { "AG": 2 }, IRF4: { "CT": 1 } } },
      { text: "很少", genotypes: { TYR: { "GG": 1 }, IRF4: { "CC": 1 } } },
      { text: "基本没有", genotypes: { TYR: { "GG": 2 }, IRF4: { "CC": 2 } } }
    ]
  },
  {
    question: "你家族中有蓝眼睛成员吗？",
    options: [
      { text: "有且比例高", genotypes: { HERC2: { "GG": 2 } } },
      { text: "有部分", genotypes: { HERC2: { "AG": 2 } } },
      { text: "极少", genotypes: { HERC2: { "AA": 1 } } },
      { text: "没有", genotypes: { HERC2: { "AA": 2 } } }
    ]
  },
  {
    question: "你的肤色属于？",
    options: [
      { text: "深色", genotypes: { SLC24A5: { "AA": 2 } } },
      { text: "中等", genotypes: { SLC24A5: { "AG": 2 } } },
      { text: "浅色", genotypes: { SLC24A5: { "GG": 1 } } },
      { text: "非常浅", genotypes: { SLC24A5: { "GG": 2 } } }
    ]
  },
  {
    question: "你的头发卷曲程度？",
    options: [
      { text: "非常卷", genotypes: { KRT71: { "TT": 2 } } },
      { text: "中度卷", genotypes: { KRT71: { "CT": 2 } } },
      { text: "微卷", genotypes: { KRT71: { "CC": 1 } } },
      { text: "直发", genotypes: { KRT71: { "CC": 2 } } }
    ]
  },
  {
    question: "你的唇型属于？",
    options: [
      { text: "厚唇", genotypes: { TP63: { "AA": 2 } } },
      { text: "中厚", genotypes: { TP63: { "AG": 2 } } },
      { text: "中等", genotypes: { TP63: { "GG": 1 } } },
      { text: "薄唇", genotypes: { TP63: { "GG": 2 } } }
    ]
  }
];

let currentIndex = 0;
let selectedOptions = [];
let shuffledQuestions = [];
let shuffledOptionsList = [];
let genotypeScores = {};
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
  qTextEl.textContent = q.question;
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

function calculateGenotypeScores() {
  genotypeScores = {};
  shuffledQuestions.forEach((q, qIdx) => {
    const optIdx = selectedOptions[qIdx];
    if(optIdx == null) return;
    const origIdx = shuffledOptionsList[qIdx][optIdx].origIdx;
    const opt = q.options[origIdx];
    for (let gene in opt.genotypes) {
      for (let gt in opt.genotypes[gene]) {
        const key = `${gene}:${gt}`;
        genotypeScores[key] = (genotypeScores[key] || 0) + opt.genotypes[gene][gt];
      }
    }
  });
}

function getTopGenotypePerGene() {
  // 按每个基因，选分数最高的基因型
  let geneTypeResult = {};
  let geneTypeScore = {};
  for (let key in genotypeScores) {
    const [gene, gt] = key.split(":");
    if (!geneTypeScore[gene] || genotypeScores[key] > geneTypeScore[gene]) {
      geneTypeScore[gene] = genotypeScores[key];
      geneTypeResult[gene] = gt;
    }
  }
  return geneTypeResult;
}

function generateGeneResultText(geneTypeResult) {
  let text = "";
  for (let gene in geneTypeResult) {
    text += `${gene}：${geneTypeResult[gene]}\n`;
  }
  return text;
}

function generateBasicReport(geneTypeResult) {
  let report = "";
  for (let gene in geneTypeResult) {
    const gt = geneTypeResult[gene];
    if (gene === "MC1R") {
      report += gt.includes("V") ? "MC1R变异：浅色/红发概率高，注意防晒。\n" : "MC1R常见型：深色发色，紫外线抵抗力较好。\n";
    }
    if (gene === "OCA2") {
      report += gt === "GG" ? "OCA2变异：浅色眼睛概率高，注意眼部防护。\n" : "OCA2常见型：深色眼睛，黑色素较多。\n";
    }
    if (gene === "SLC45A2") {
      report += gt === "GG" ? "SLC45A2变异：皮肤较白，易晒伤，注意防晒。\n" : "SLC45A2常见型：皮肤偏深，耐晒能力强。\n";
    }
    if (gene === "EDAR") {
      report += gt === "AA" ? "EDAR变异：头发粗硬直，汗腺发达，牙齿形态特殊。\n" : "EDAR常见型：头发柔软，汗腺发育一般。\n";
    }
    if (gene === "FGFR2") {
      report += gt === "AA" ? "FGFR2变异：高鼻梁特征明显，面部立体感强。\n" : "FGFR2常见型：鼻梁中等或偏低，五官柔和。\n";
    }
    if (gene === "ABCC11") {
      report += gt === "GG" ? "ABCC11变异：湿耳垢型，腋臭概率高，注意个人护理。\n" : "ABCC11常见型：干耳垢，体味轻。\n";
    }
    if (gene === "ASIP") {
      report += gt === "AA" ? "ASIP变异：肤色较深，色素沉着明显。\n" : "ASIP常见型：肤色浅，色素沉着少。\n";
    }
    if (gene === "TYR") {
      report += gt === "AA" ? "TYR变异：雀斑概率高，注意皮肤护理和防晒。\n" : "TYR常见型：雀斑少，皮肤均匀。\n";
    }
    if (gene === "HERC2") {
      report += gt === "GG" ? "HERC2变异：蓝眼睛概率高，注意光线保护。\n" : "HERC2常见型：棕色或深色眼睛。\n";
    }
    if (gene === "SLC24A5") {
      report += gt === "GG" ? "SLC24A5变异：皮肤极浅，需注意紫外线防护。\n" : "SLC24A5常见型：肤色中等或偏深。\n";
    }
    if (gene === "KRT71") {
      report += gt === "TT" ? "KRT71变异：头发卷曲度高，易打理。\n" : "KRT71常见型：头发直或微卷。\n";
    }
    if (gene === "TP63") {
      report += gt === "AA" ? "TP63变异：厚唇，面部立体感强。\n" : "TP63常见型：薄唇或中等唇型。\n";
    }
    if (gene === "PAX3") {
      report += gt === "AA" ? "PAX3变异：鼻梁较高，面部立体感强。\n" : "PAX3常见型：鼻梁中等或偏低。\n";
    }
    if (gene === "IRF4") {
      report += gt === "TT" ? "IRF4变异：雀斑概率高，注意皮肤护理。\n" : "IRF4常见型：雀斑少。\n";
    }
  }
  return report;
}

function generatePremiumReport() {
  return "这是基于你答题结果和最新外貌基因科学的深度发展报告，涵盖健康、护理、个性化美学建议等内容。\n（如需定制，请联系专业遗传咨询师）";
}

btnStart.onclick = () => {
  const age = parseInt(document.getElementById("inputAge").value);
  if(isNaN(age) || age < 0) { alert("请输入有效年龄"); return; }
  selectedOptions = [];
  shuffledQuestions = shuffle(geneQuestionBank.map(q => ({...q, options: q.options.slice()})));
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
    calculateGenotypeScores();
    const geneTypeResult = getTopGenotypePerGene();
    renderResult(geneTypeResult);
  }
};

btnPrev.onclick = () => {
  if(currentIndex > 0) renderQuestion(currentIndex - 1);
};

function renderResult(geneTypeResult) {
  sectionQuiz.style.display = "none";
  sectionResult.style.display = "block";
  userHasPaid = false;
  paySection.style.display = "none";
  geneResultEl.textContent = generateGeneResultText(geneTypeResult);
  basicReportEl.textContent = generateBasicReport(geneTypeResult);
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
