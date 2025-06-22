<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>个性发展报告</title>
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
  #paySection img { width: 220px; border:1px solid #ccc; border-radius:8px; }
  pre { white-space: pre-wrap; word-break: break-word; background:#eee; padding:10px; border-radius:6px; font-family: monospace; user-select: all; cursor: default; }
</style>
</head>
<body>

<h2>基因型智能推断测评</h2>

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
    <img src="https://pplx-res.cloudinary.com/image/upload/v1750588444/user_uploads/19241329/9b7936f2-1eeb-4500-99b9-9ddc57f69890/mm_facetoface_collect_qrcode_1750513571858.jpg" alt="微信收款码" />
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

// 题库举例，每选项对应不同基因型和权重
const geneQuestionBank = [
  {
    question: "你喝咖啡的感受？",
    options: [
      { text: "喝了容易兴奋", genotypes: { CYP1A2: { "AA": 2 } } },
      { text: "有点提神", genotypes: { CYP1A2: { "AC": 2 } } },
      { text: "没啥感觉", genotypes: { CYP1A2: { "CC": 1 } } },
      { text: "喝多少都能睡", genotypes: { CYP1A2: { "CC": 2 } } }
    ]
  },
  {
    question: "你对压力的反应？",
    options: [
      { text: "容易焦虑", genotypes: { SLC6A4: { "SS": 2 } } },
      { text: "偶尔紧张", genotypes: { SLC6A4: { "SL": 2 } } },
      { text: "一般能扛住", genotypes: { SLC6A4: { "LL": 1 } } },
      { text: "遇事很冷静", genotypes: { SLC6A4: { "LL": 2 } } }
    ]
  },
  {
    question: "你喜欢独处还是热闹？",
    options: [
      { text: "喜欢安静独处", genotypes: { DRD4: { "4R/4R": 2 } } },
      { text: "偶尔独处", genotypes: { DRD4: { "4R/7R": 2 } } },
      { text: "喜欢热闹", genotypes: { DRD4: { "7R/7R": 1 } } },
      { text: "人多越开心", genotypes: { DRD4: { "7R/7R": 2 } } }
    ]
  },
  {
    question: "你对新鲜事物的态度？",
    options: [
      { text: "很保守", genotypes: { DRD4: { "4R/4R": 2 } } },
      { text: "偏保守", genotypes: { DRD4: { "4R/7R": 2 } } },
      { text: "愿意尝试", genotypes: { DRD4: { "7R/7R": 1 } } },
      { text: "喜欢创新", genotypes: { DRD4: { "7R/7R": 2 } } }
    ]
  }
  // 可继续扩展更多题目
];

const geneStrengths = {
  SLC6A4: {
    "LL": "情绪自我调节能力强，心理韧性好，抗压能力出色。",
    "SL": "情绪调节能力适中，能逐步适应压力。",
    "SS": "情感细腻，善于共情，适合需要同理心的领域。"
  },
  CYP1A2: {
    "AA": "对咖啡因敏感，善于觉察身体信号，适合健康管理。",
    "AC": "能平衡兴奋与镇静，适应性强。",
    "CC": "耐受力强，适合高强度工作和学习。"
  },
  DRD4: {
    "4R/4R": "稳定可靠，适合长期规划和深度专注。",
    "4R/7R": "兼具稳定与创新，适合多元发展。",
    "7R/7R": "探索精神强，善于创新和适应变化。"
  }
};

const geneAdvice = {
  SLC6A4: {
    "LL": "建议多参与社交和挑战性活动，持续提升抗压能力。",
    "SL": "建议适度锻炼情绪调节力，面对压力时积极寻求支持。",
    "SS": "建议学习情绪管理技巧，关注心理健康，发挥共情优势。"
  },
  CYP1A2: {
    "AA": "建议减少咖啡因摄入，保持良好作息。",
    "AC": "建议根据身体反应调整饮食和作息。",
    "CC": "建议合理利用高耐受力，注意劳逸结合。"
  },
  DRD4: {
    "4R/4R": "建议专注于长期规划和深度学习。",
    "4R/7R": "建议在稳定中不断创新，开拓多元发展。",
    "7R/7R": "建议多参与探索性项目，发挥创新潜力。"
  }
};

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
  let arr = [];
  for (let gene in geneTypeResult) {
    let gt = geneTypeResult[gene];
    if(geneStrengths[gene] && geneStrengths[gene][gt]) {
      arr.push(`【${gene}】${geneStrengths[gene][gt]}`);
    }
    if(geneAdvice[gene] && geneAdvice[gene][gt]) {
      arr.push(`【${gene}建议】${geneAdvice[gene][gt]}`);
    }
  }
  return arr.length ? arr.join('\n') : "你的潜在优势和建议有待进一步发掘。";
}

function generatePremiumReport() {
  return "你的基因型展现出多方面的潜力和优势。建议结合自身特质，科学安排生活、学习与工作，持续发掘潜能。如需更详细的个性化成长方案，请扫码咨询专业顾问。";
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
