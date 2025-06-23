<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>基因型问卷 & 虚拟形象生成</title>
  <style>
    body { max-width: 700px; margin: 24px auto; font-family: "微软雅黑", sans-serif; background: #fff; padding: 18px; color: #222; }
    h2, h3 { text-align: center; }
    label, select, input[type=number] { width: 100%; margin: 10px 0 15px; font-size: 16px; }
    select, input[type=number] { padding: 8px; border-radius: 6px; border: 1px solid #ccc; }
    button { width: 100%; padding: 12px; font-size: 16px; border: none; border-radius: 6px; cursor: pointer; color: white; background: #4338ca; }
    button:disabled { background: #a5b4fc; cursor: not-allowed; }
    .option { background: white; border: 1px solid #bbb; border-radius: 6px; padding: 10px; margin: 6px 0; cursor: pointer; user-select:none; }
    .option.selected { background: #4f46e5; color: white; border-color: #4338ca; }
    #promptOutput, #descOutput, #psyOutput { white-space: pre-wrap; overflow-x: auto; background: #eee; padding: 10px; border-radius: 6px; font-family: monospace; user-select: all; }
    #geneTable { width:100%; margin: 18px 0; border-collapse:collapse; background: #fcfcfc; }
    #geneTable th, #geneTable td { border:1px solid #ddd; padding: 5px 7px; font-size: 15px; }
    #geneTable th { background: #f5f5f5; }
    #totalScoreText { text-align: center; font-weight: bold; margin: 10px 0; font-size: 32px; transition: color 0.5s; }
    .sectionSub { margin:18px 0; }
  </style>
</head>
<body>

<h2>基因型行为问卷 & AI虚拟形象生成</h2>

<div id="sectionUserInfo">
  <label for="inputAge">年龄（不限）</label>
  <input type="number" id="inputAge" min="0" value="25" />
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
  <div id="progressText"></div>
</div>

<div id="sectionResult" style="display:none;">
  <h3>您的基因型得分表</h3>
  <table id="geneTable"></table>
  <div id="totalScoreText"></div>
  <div class="sectionSub">
    <h3>外貌特征自动描述</h3>
    <pre id="descOutput"></pre>
  </div>
  <div class="sectionSub">
    <h3>虚拟心理/行为特征</h3>
    <pre id="psyOutput"></pre>
  </div>
  <div class="sectionSub">
    <h3>AI虚拟形象人设摘要</h3>
    <pre id="promptOutput" title="点击可复制"></pre>
  </div>
  <button id="btnRestart">重新开始</button>
</div>

<script>
// 主流多功能基因
const geneList = [
  "MC1R", "OCA2", "SLC24A5", "KITLG", "TYR",
  "EDAR", "FGFR2", "FGFR3", "TBX15", "PAX3", "DCHS2", "RUNX2",
  "FTO", "LEPR", "MC4R", "ADIPOQ", "DRD4", "5HTT", "COMT", "BDNF", "MAOA", "OXTR"
];
const geneGeneDesc = {
  MC1R: "皮肤/头发颜色、色素、晒伤", OCA2: "眼色、头发色、皮肤色",
  SLC24A5: "皮肤色、黑色素、发色", KITLG: "皮肤色、发色、雀斑", TYR: "黑色素、皮肤、眼色",
  EDAR: "毛发粗细、发型、脸型、牙齿", FGFR2: "面部骨相、鼻型、颧骨", FGFR3: "骨发育、脸型、耳廓",
  TBX15: "脸型、颧骨、骨相", PAX3: "脸型、额头、鼻梁", DCHS2: "脸型、酒窝", RUNX2: "颧骨、骨相、体型",
  FTO: "体型、胖瘦、脂肪分布", LEPR: "体型、脂肪分布", MC4R: "体型、肥胖", ADIPOQ: "脂肪代谢、体型",
  DRD4:"冒险/探索倾向", "5HTT":"情绪稳定性", COMT:"压力反应", BDNF:"学习记忆", MAOA:"冲动控制", OXTR:"共情/社交"
};
const geneTypeColors = {
  "高色素型": "#FF5722","骨相立体型": "#1976D2","体型易胖型": "#8E24AA",
  "毛发特殊型": "#1DE9B6","心理冒险型": "#FFB300","心理稳定型": "#009688","普通型": "#888888"
};
const geneTypeDesc = {
  "高色素型": "色素基因突出，肤色发色瞳色有独特亮点。",
  "骨相立体型": "骨相五官突出，轮廓感强。",
  "体型易胖型": "体型基因高分，易积脂肪。",
  "毛发特殊型": "毛发特征优势明显。",
  "心理冒险型": "心理行为基因偏冒险、探索。",
  "心理稳定型": "心理行为基因偏情绪稳定、自控。",
  "普通型": "外貌心理相关基因结构均衡。"
};
// 12题，题目与选项随机
const questionPool = [
  {text:"你喜欢什么样的户外活动？",options:[
    {text:"海边日光浴",tags:{MC1R:2,SLC24A5:1,KITLG:1,DRD4:1}},
    {text:"森林徒步",tags:{FGFR2:1,TYR:1,DCHS2:1,OXTR:1}},
    {text:"极限运动",tags:{FTO:1,MC4R:1,RUNX2:1,DRD4:2}},
    {text:"更喜欢室内",tags:{MC1R:-1,FTO:-1,"5HTT":1}}
  ]},
  {text:"你喜欢什么类型的饮食？",options:[
    {text:"高蛋白健身餐",tags:{FTO:-2,MC4R:-1}},
    {text:"高碳水主食",tags:{FTO:2,LEPR:2}},
    {text:"喜欢甜食",tags:{FTO:2,MC4R:1,TYR:1}},
    {text:"清淡素食",tags:{FTO:-1,SLC24A5:1}}
  ]},
  {text:"你在社交中最常被夸奖什么？",options:[
    {text:"五官立体",tags:{FGFR2:2,PAX3:2,TBX15:1,OXTR:1}},
    {text:"皮肤健康",tags:{MC1R:2,SLC24A5:1,OCA2:1}},
    {text:"气质出众",tags:{RUNX2:1,FTO:1}},
    {text:"身材匀称",tags:{FTO:2,MC4R:2,LEPR:1}}
  ]},
  {text:"你喜欢什么风格的服饰？",options:[
    {text:"欧美时尚",tags:{MC1R:1,OCA2:1,FGFR2:1}},
    {text:"日韩清新",tags:{EDAR:2,SLC24A5:1}},
    {text:"运动休闲",tags:{FTO:1,MC4R:1}},
    {text:"民族风",tags:{KITLG:1,PAX3:1}}
  ]},
  {text:"你在压力下更倾向？",options:[
    {text:"主动社交",tags:{OXTR:2,DRD4:1}},
    {text:"独自消化",tags:{"5HTT":1,COMT:1}},
    {text:"运动释放",tags:{FTO:1,DRD4:1}},
    {text:"吃东西缓解",tags:{FTO:2,MC4R:2}}
  ]},
  {text:"你最喜欢的季节？",options:[
    {text:"夏天",tags:{MC1R:2,KITLG:1}},
    {text:"冬天",tags:{FTO:1,MC4R:1}},
    {text:"春天",tags:{OCA2:1,SLC24A5:1}},
    {text:"秋天",tags:{FGFR2:1,TBX15:1}}
  ]},
  {text:"你对自己的头发最满意哪些方面？",options:[
    {text:"发质柔顺",tags:{EDAR:2,FGFR2:1}},
    {text:"自然卷曲",tags:{EDAR:-2,FGFR3:1,TBX15:1}},
    {text:"头发颜色独特",tags:{MC1R:1,OCA2:1,KITLG:2}},
    {text:"不太满意",tags:{EDAR:-1,MC1R:-1}}
  ]},
  {text:"你笑起来时最突出的特点？",options:[
    {text:"酒窝明显",tags:{DCHS2:2,RUNX2:1}},
    {text:"牙齿整齐",tags:{EDAR:2,FGFR3:1}},
    {text:"颧骨分明",tags:{FGFR2:1,TBX15:2}},
    {text:"脸型圆润",tags:{PAX3:1,FTO:1}}
  ]},
  {text:"你更容易被什么吸引？",options:[
    {text:"与众不同的外表",tags:{MC1R:2,EDAR:1}},
    {text:"深邃的气质",tags:{PAX3:1,BDNF:1}},
    {text:"有趣的谈吐",tags:{OXTR:1,DRD4:1}},
    {text:"温和的性格",tags:{"5HTT":2,OXTR:1}}
  ]},
  {text:"你偏爱的发色？",options:[
    {text:"自然黑/棕",tags:{OCA2:1}},
    {text:"金色/浅色",tags:{MC1R:2,SLC24A5:2}},
    {text:"红色/紫色",tags:{MC1R:2,KITLG:2}},
    {text:"其他亮色",tags:{SLC24A5:1,KITLG:1}}
  ]},
  {text:"你在童年最显著的外貌变化？",options:[
    {text:"开始长雀斑",tags:{MC1R:2,KITLG:2}},
    {text:"牙齿或颧骨变化",tags:{EDAR:2,FGFR2:1,TBX15:1}},
    {text:"体型明显改变",tags:{FTO:2,LEPR:1}},
    {text:"基本没变化",tags:{RUNX2:-1,FTO:-1}}
  ]},
  {text:"你面对新环境时？",options:[
    {text:"兴奋好奇",tags:{DRD4:2,OCA2:1}},
    {text:"谨慎观察",tags:{"5HTT":2,COMT:1}},
    {text:"主动融入",tags:{OXTR:2}},
    {text:"不适应",tags:{"5HTT":-2}}
  ]}
];

// 洗牌
function shuffleArray(arr) {
  for(let i=arr.length-1;i>0;i--){
    const j=Math.floor(Math.random()*(i+1));
    [arr[i],arr[j]]=[arr[j],arr[i]];
  }
}

// 全局变量
let currentQuestions=[],geneScores={},currentQuestion=null,selectedOptionIndex=null,questionCount=0,age=25,gender="male";
const sectionUserInfo=document.getElementById("sectionUserInfo");
const sectionQuiz=document.getElementById("sectionQuiz");
const sectionResult=document.getElementById("sectionResult");
const qTextEl=document.getElementById("questionText");
const optionsEl=document.getElementById("optionsContainer");
const btnNext=document.getElementById("btnNext");
const btnPrev=document.getElementById("btnPrev");
const progressText=document.getElementById("progressText");
const promptOutput=document.getElementById("promptOutput");
const descOutput=document.getElementById("descOutput");
const psyOutput=document.getElementById("psyOutput");
const btnStart=document.getElementById("btnStart");
const btnRestart=document.getElementById("btnRestart");
const inputAge=document.getElementById("inputAge");
const selectGender=document.getElementById("selectGender");
const totalScoreText=document.getElementById("totalScoreText");
const geneTable=document.getElementById("geneTable");

function selectOption(idx){
  selectedOptionIndex=idx;
  Array.from(optionsEl.children).forEach((el,i)=>{
    el.classList.toggle("selected",i===idx);
  });
  btnNext.disabled=false;
}
function renderQuestion(q){
  currentQuestion=q;
  qTextEl.textContent=q.text;
  optionsEl.innerHTML="";
  selectedOptionIndex=typeof q.selected==="number"?q.selected:null;
  btnNext.disabled=selectedOptionIndex===null;
  btnPrev.disabled=questionCount<=0;
  for(let i=0;i<q.options.length;i++){
    const opt=document.createElement("div");
    opt.className="option";
    opt.textContent=q.options[i].text;
    opt.onclick=()=>selectOption(i);
    if(selectedOptionIndex===i)opt.classList.add("selected");
    optionsEl.appendChild(opt);
  }
  progressText.textContent=`第 ${questionCount+1} / ${currentQuestions.length} 题`;
}
function showResult(){
  sectionQuiz.style.display="none";
  sectionResult.style.display="block";
  geneScores={};
  for(const gene of geneList)geneScores[gene]=0;
  for(let i=0;i<currentQuestions.length;i++){
    const sel=currentQuestions[i].selected;
    if(typeof sel!=="number")continue;
    const tags=currentQuestions[i].options[sel].tags;
    for(const g in tags){
      geneScores[g]=(geneScores[g]||0)+tags[g];
    }
  }
  geneTable.innerHTML="<tr><th>基因</th><th>分数</th><th>主要影响</th></tr>"+
    geneList.map(g=>`<tr><td>${g}</td><td>${geneScores[g]}</td><td>${geneGeneDesc[g]||""}</td></tr>`).join("");
  // 型别判定
  let geneType="普通型",color=geneTypeColors["普通型"];
  let pigmentSum=geneScores["MC1R"]+geneScores["OCA2"]+geneScores["SLC24A5"]+geneScores["KITLG"]+geneScores["TYR"];
  let boneSum=geneScores["EDAR"]+geneScores["FGFR2"]+geneScores["FGFR3"]+geneScores["TBX15"]+geneScores["PAX3"]+geneScores["DCHS2"]+geneScores["RUNX2"];
  let bodySum=geneScores["FTO"]+geneScores["LEPR"]+geneScores["MC4R"]+geneScores["ADIPOQ"];
  let hairSum=geneScores["EDAR"]+geneScores["KITLG"]+geneScores["MC1R"]+geneScores["FGFR2"];
  let psychoAd=geneScores["DRD4"]+geneScores["BDNF"];
  let psychoStable=geneScores["5HTT"]+geneScores["COMT"]+geneScores["MAOA"]+geneScores["OXTR"];
  const maxVal=Math.max(pigmentSum,boneSum,bodySum,hairSum,psychoAd,psychoStable);
  if(maxVal===pigmentSum&&pigmentSum>2){geneType="高色素型";color=geneTypeColors[geneType];}
  else if(maxVal===boneSum&&boneSum>2){geneType="骨相立体型";color=geneTypeColors[geneType];}
  else if(maxVal===bodySum&&bodySum>1){geneType="体型易胖型";color=geneTypeColors[geneType];}
  else if(maxVal===hairSum&&hairSum>1){geneType="毛发特殊型";color=geneTypeColors[geneType];}
  else if(maxVal===psychoAd&&psychoAd>0){geneType="心理冒险型";color=geneTypeColors[geneType];}
  else if(maxVal===psychoStable&&psychoStable>0){geneType="心理稳定型";color=geneTypeColors[geneType];}
  totalScoreText.textContent="主导型别："+geneType+"（总分："+maxVal+"）";
  totalScoreText.style.color=color;
  let desc=[];
  if(pigmentSum>1)desc.push("肤色或色素特征突出。");
  if(hairSum>1)desc.push("毛发特征明显。");
  if(boneSum>1)desc.push("五官立体、骨相鲜明。");
  if(bodySum>1)desc.push("体型偏丰腴。");
  if(bodySum<-1)desc.push("体型偏瘦。");
  if(desc.length===0)desc.push("外貌特征均衡。");
  descOutput.textContent=desc.join('\n');
  let psy=[];
  if(psychoAd>0)psy.push("冒险、探索欲较强。");
  if(psychoStable>0)psy.push("情绪稳定，自控力佳。");
  if(geneScores["MAOA"]<0)psy.push("偶尔有冲动。");
  if(geneScores["5HTT"]<0)psy.push("情绪敏感。");
  if(psy.length===0)psy.push("心理行为均衡。");
  psyOutput.textContent=psy.join('\n');
  let prompt="# AI虚拟形象属性\n主导基因型："+geneType+"\n外貌特征："+desc.join(" ")+"\n心理/行为特质："+psy.join(" ")+"\n# 详细基因评分\n"+geneList.map(g=>`${g}:${geneScores[g]}`).join(', ');
  promptOutput.textContent=prompt;
}
btnStart.onclick=()=>{
  age=parseInt(inputAge.value);
  if(isNaN(age)||age<0){alert("请输入有效年龄");return;}
  gender=selectGender.value;
  geneScores={};questionCount=0;
  currentQuestions=questionPool.slice();
  shuffleArray(currentQuestions);
  currentQuestions=currentQuestions.slice(0,12);
  currentQuestions.forEach(q=>shuffleArray(q.options));
  sectionUserInfo.style.display="none";
  sectionResult.style.display="none";
  sectionQuiz.style.display="block";
  renderQuestion(currentQuestions[questionCount]);
};
btnNext.onclick=()=>{
  if(selectedOptionIndex===null)return;
  currentQuestions[questionCount].selected=selectedOptionIndex;
  questionCount++;
  const nextQ=currentQuestions[questionCount]||null;
  if(!nextQ){showResult();}else{renderQuestion(nextQ);}
};
btnPrev.onclick=()=>{
  if(questionCount<=0)return;
  currentQuestions[questionCount].selected=selectedOptionIndex;
  questionCount--;
  renderQuestion(currentQuestions[questionCount]);
  if(typeof currentQuestions[questionCount].selected==="number"){
    selectOption(currentQuestions[questionCount].selected);
  }
};
btnRestart.onclick=()=>{
  geneScores={};questionCount=0;
  sectionResult.style.display="none";
  sectionQuiz.style.display="none";
  sectionUserInfo.style.display="block";
};
</script>
</body>
</html>