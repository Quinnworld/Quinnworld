<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>虚拟基因人格画像问卷</title>
  <style>
    html, body {
      height: 100%;
      margin: 0;
      padding: 0;
      overflow: hidden;
      background: #f5f7fa;
    }
    body {
      height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      min-width: 100vw;
    }
    #mainbox {
      width: 480px;
      min-height: 540px;
      max-width: 98vw;
      background: #fff;
      box-shadow: 0 6px 24px 0 #cdd2f1;
      border-radius: 18px;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: flex-start;
      padding: 36px 24px 24px 24px;
      position: relative;
    }
    @media (max-width: 600px) {
      #mainbox {
        width: 98vw;
        min-height: 98vh;
        padding: 16px 2vw 16px 2vw;
      }
    }
    h2, h3 {
      text-align: center;
      font-weight: 600;
      margin: 8px 0 24px;
      letter-spacing: 1.5px;
      color: #2b2f4a;
    }
    label, select, input[type=number] {
      width: 100%;
      margin: 10px 0 15px;
      font-size: 17px;
      font-weight: 500;
    }
    select, input[type=number] {
      padding: 8px;
      border-radius: 7px;
      border: 1px solid #d9dbe7;
      background: #f4f6fb;
    }
    button {
      width: 100%;
      padding: 14px;
      font-size: 17px;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      color: white;
      background: linear-gradient(90deg,#6366f1,#60a5fa 70%);
      margin-top: 10px;
      font-weight: 600;
      transition: background .2s;
      box-shadow: 0 2px 8px #dbeafe;
    }
    button:disabled {
      background: #dbeafe;
      color: #90a4c6;
      cursor: not-allowed;
    }
    .option {
      background: #f3f3fa;
      border: 1.5px solid #c7d2fe;
      border-radius: 8px;
      padding: 13px 11px;
      margin: 10px 0;
      transition: background 0.15s, color 0.15s, border 0.15s;
      cursor: pointer;
      font-size: 16.2px;
      letter-spacing: .2px;
      user-select: none;
    }
    .option.selected {
      background: linear-gradient(90deg,#818cf8 70%,#60a5fa 100%);
      color: white;
      border-color: #6366f1;
      font-weight: bold;
      box-shadow: 0 2px 8px #dbeafe;
    }
    #promptOutput {
      white-space: pre-wrap;
      overflow-x: auto;
      background: #f4f6fb;
      padding: 12px;
      border-radius: 8px;
      font-family: inherit;
      margin-bottom: 12px;
      font-size: 15.6px;
    }
    #radarCanvas {
      display: block;
      margin: 25px auto 8px;
      background: #fff;
      border-radius: 8px;
    }
    #totalScoreText {
      text-align: center;
      font-weight: 600;
      margin: 10px 0 10px;
      font-size: 22px;
      color: #2b2f4a;
    }
    .dim-label {
      font-size: 15px;
      color: #444;
      background: #f1f5fe;
      padding: 3px 8px;
      border-radius: 6px;
      margin: 0 7px 0 0;
      display: inline-block;
    }
  </style>
</head>
<body>
<div id="mainbox">
  <h2>虚拟基因人格画像问卷</h2>

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
    <div id="progressText" style="margin-bottom:8px;font-size:15px;color:#555;"></div>
  </div>

  <div id="sectionResult" style="display:none;">
    <div id="totalScoreText"></div>
    <h3>AI形象prompt (可用于AI绘画、角色生成)</h3>
    <pre id="promptOutput"></pre>
    <h3 style="margin:16px 0 0 0;">多维人格特征雷达图</h3>
    <canvas id="radarCanvas" width="340" height="340"></canvas>
    <div id="radarScoreText" style="width:100%;text-align:center;font-size:15px;color:#3b4252;margin:10px 0 0;"></div>
    <button id="btnRestart" style="margin-top:25px;">重新开始</button>
  </div>
</div>
<script>
// 更多基因人格多维度
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

// 12题，每题映射到多维
const questionPool = [
  {text:"你喜欢什么样的户外活动？",options:[
    {text:"海边日光浴",radar:{pigment:2,advent:1,happiness:1}},
    {text:"森林徒步",radar:{bone:1,stable:1,stamina:1}},
    {text:"极限运动",radar:{body:1,advent:2,stamina:2}},
    {text:"更喜欢室内",radar:{pigment:-1,body:-1,stable:1,create:1}}
  ]},
  {text:"你喜欢什么类型的饮食？",options:[
    {text:"高蛋白健身餐",radar:{body:-2,self:1,stamina:1}},
    {text:"高碳水主食",radar:{body:2,happiness:1}},
    {text:"喜欢甜食",radar:{body:2,stable:-1,happiness:1}},
    {text:"清淡素食",radar:{body:-1,pigment:1,empathy:1}}
  ]},
  {text:"你在社交中最常被夸奖什么？",options:[
    {text:"五官立体",radar:{bone:2}},
    {text:"皮肤健康",radar:{pigment:2}},
    {text:"气质出众",radar:{advent:1,stable:1,create:1}},
    {text:"身材匀称",radar:{body:2,self:1}}
  ]},
  {text:"你喜欢什么风格的服饰？",options:[
    {text:"欧美时尚",radar:{pigment:1,bone:1,create:1}},
    {text:"日韩清新",radar:{hair:2,social:1}},
    {text:"运动休闲",radar:{body:1,advent:1,stamina:1}},
    {text:"民族风",radar:{pigment:1,advent:1,empathy:1}}
  ]},
  {text:"你在压力下更倾向？",options:[
    {text:"主动社交",radar:{advent:2,social:2}},
    {text:"独自消化",radar:{stable:2,self:1}},
    {text:"运动释放",radar:{body:1,advent:1,stamina:2}},
    {text:"吃东西缓解",radar:{body:2,stable:-1,happiness:1}}
  ]},
  {text:"你最喜欢的季节？",options:[
    {text:"夏天",radar:{pigment:2,happiness:1}},
    {text:"冬天",radar:{body:1,stable:1,self:1}},
    {text:"春天",radar:{hair:1,pigment:1,empathy:1}},
    {text:"秋天",radar:{bone:1,stable:1,happiness:1}}
  ]},
  {text:"你对自己的头发最满意哪些方面？",options:[
    {text:"发质柔顺",radar:{hair:2}},
    {text:"自然卷曲",radar:{hair:1,bone:1,create:1}},
    {text:"头发颜色独特",radar:{hair:2,pigment:1,create:2}},
    {text:"不太满意",radar:{hair:-2,self:-1}}
  ]},
  {text:"你笑起来时最突出的特点？",options:[
    {text:"酒窝明显",radar:{bone:2,happiness:1}},
    {text:"牙齿整齐",radar:{bone:1,hair:1,self:1}},
    {text:"颧骨分明",radar:{bone:2,create:1}},
    {text:"脸型圆润",radar:{bone:1,body:1,empathy:1}}
  ]},
  {text:"你更容易被什么吸引？",options:[
    {text:"与众不同的外表",radar:{pigment:2,hair:1,create:2}},
    {text:"深邃的气质",radar:{bone:1,advent:1,empathy:1}},
    {text:"有趣的谈吐",radar:{advent:2,create:2}},
    {text:"温和的性格",radar:{stable:2,empathy:2}}
  ]},
  {text:"你偏爱的发色？",options:[
    {text:"自然黑/棕",radar:{hair:1}},
    {text:"金色/浅色",radar:{hair:2,pigment:2,create:1}},
    {text:"红色/紫色",radar:{hair:2,pigment:2,create:2}},
    {text:"其他亮色",radar:{hair:1,pigment:1,create:1}}
  ]},
  {text:"你在童年最显著的外貌变化？",options:[
    {text:"开始长雀斑",radar:{pigment:2,happiness:1}},
    {text:"牙齿或颧骨变化",radar:{bone:2,hair:1,self:1}},
    {text:"体型明显改变",radar:{body:2,stamina:1}},
    {text:"基本没变化",radar:{bone:-1,body:-1,stable:1}}
  ]},
  {text:"你面对新环境时？",options:[
    {text:"兴奋好奇",radar:{advent:2,create:1,social:1}},
    {text:"谨慎观察",radar:{stable:2,self:1}},
    {text:"主动融入",radar:{advent:1,stable:1,social:2}},
    {text:"不适应",radar:{advent:-2,stable:-2,happiness:-1}}
  ]}
];

// prompt生成
function genPrompt(radar, gender) {
  let parts = [];
  if (gender === "female") parts.push("young woman,");
  else if (gender === "male") parts.push("young man,");
  else parts.push("person,");
  // 色素
  if (radar.pigment > 2) parts.push("fair skin, vivid eyes, striking hair color,");
  else if (radar.pigment < -1) parts.push("tan skin, natural skin tone,");
  // 骨相
  if (radar.bone > 2) parts.push("high cheekbones, defined jaw, 3D face,");
  else if (radar.bone < -1) parts.push("round face, soft facial lines,");
  // 体型
  if (radar.body > 2) parts.push("full figure, healthy body,");
  else if (radar.body < -1) parts.push("slim body, slender,");
  // 毛发
  if (radar.hair > 2) parts.push("voluminous hair, unique hairstyle,");
  else if (radar.hair < -1) parts.push("thin soft hair,");
  // 行为与气质
  if (radar.advent > 2) parts.push("adventurous, confident, lively,");
  else if (radar.advent < -1) parts.push("reserved, introverted, calm,");
  if (radar.stable > 2) parts.push("emotionally stable, poised,");
  else if (radar.stable < -1) parts.push("sensitive, gentle temperament,");
  if (radar.social > 2) parts.push("very social, charismatic,");
  else if (radar.social < -1) parts.push("private, prefers solitude,");
  if (radar.create > 2) parts.push("creative, imaginative, unique style,");
  if (radar.self > 2) parts.push("disciplined, persistent,");
  if (radar.stamina > 2) parts.push("high stamina, energetic,");
  if (radar.empathy > 2) parts.push("empathetic, warm-hearted,");
  if (radar.happiness > 2) parts.push("bright, sunny disposition,");
  parts.push("beautiful lighting, best quality, ultra detailed");
  return parts.join(" ");
}

// 雷达分数格式化
function radarScoreText(radar){
  return radarDims.map(dim=>`<span class="dim-label">${dim.name}：${radar[dim.key]}</span>`).join('');
}

// 雷达图
function drawRadar(ctx, width, height, radar, maxAbs=4) {
  ctx.clearRect(0,0,width,height);
  const n = radarDims.length, cx=width/2, cy=height/2, r = Math.min(width, height)/2-35;
  ctx.save(); ctx.translate(cx,cy);
  // 网格
  ctx.strokeStyle="#e0e7ff";
  ctx.lineWidth=1;
  for(let k=1;k<=maxAbs;k++){
    ctx.beginPath();
    for(let i=0;i<n;i++){
      let angle = i*2*Math.PI/n - Math.PI/2;
      let rr = r*k/maxAbs;
      let x = Math.cos(angle)*rr, y = Math.sin(angle)*rr;
      if(i===0)ctx.moveTo(x,y);else ctx.lineTo(x,y);
    }
    ctx.closePath();ctx.stroke();
  }
  // 轴线与标签
  for(let i=0;i<n;i++){
    let angle = i*2*Math.PI/n - Math.PI/2;
    let x = Math.cos(angle)*r, y = Math.sin(angle)*r;
    ctx.beginPath();ctx.moveTo(0,0);ctx.lineTo(x,y);ctx.stroke();
    ctx.font="14px sans-serif";
    ctx.textAlign="center";ctx.textBaseline="middle";
    ctx.fillStyle="#374151";
    ctx.fillText(radarDims[i].name, Math.cos(angle)*(r+22), Math.sin(angle)*(r+22));
  }
  // 数据
  ctx.beginPath();
  for(let i=0;i<n;i++){
    let angle = i*2*Math.PI/n - Math.PI/2;
    let v = Math.max(-maxAbs, Math.min(maxAbs, radar[radarDims[i].key]));
    let rr = r*(v+maxAbs)/(2*maxAbs);
    let x = Math.cos(angle)*rr, y = Math.sin(angle)*rr;
    if(i===0)ctx.moveTo(x,y);else ctx.lineTo(x,y);
  }
  ctx.closePath();
  ctx.strokeStyle="#6366f1";
  ctx.lineWidth=3;
  ctx.stroke();
  ctx.fillStyle="rgba(99,102,241,0.15)";
  ctx.fill();
  ctx.restore();
}

// 逻辑与变量
let currentQuestions=[], radarScore={}, questionCount=0, selectedOptionIndex=null, age=25, gender="male";
const sectionUserInfo=document.getElementById("sectionUserInfo");
const sectionQuiz=document.getElementById("sectionQuiz");
const sectionResult=document.getElementById("sectionResult");
const qTextEl=document.getElementById("questionText");
const optionsEl=document.getElementById("optionsContainer");
const btnNext=document.getElementById("btnNext"), btnPrev=document.getElementById("btnPrev");
const progressText=document.getElementById("progressText");
const promptOutput=document.getElementById("promptOutput");
const btnStart=document.getElementById("btnStart");
const btnRestart=document.getElementById("btnRestart");
const inputAge=document.getElementById("inputAge");
const selectGender=document.getElementById("selectGender");
const totalScoreText=document.getElementById("totalScoreText");
const radarCanvas=document.getElementById("radarCanvas");
const radarScoreTextDiv=document.getElementById("radarScoreText");

function shuffleArray(arr){for(let i=arr.length-1;i>0;i--){const j=Math.floor(Math.random()*(i+1));[arr[i],arr[j]]=[arr[j],arr[i]];}}
function selectOption(idx){
  selectedOptionIndex=idx;
  Array.from(optionsEl.children).forEach((el,i)=>el.classList.toggle("selected",i===idx));
  btnNext.disabled=false;
}
function renderQuestion(q){
  qTextEl.textContent=q.text; optionsEl.innerHTML="";
  selectedOptionIndex = typeof q.selected === "number" ? q.selected : null;
  btnNext.disabled = selectedOptionIndex === null;
  btnPrev.disabled = questionCount<=0;
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
  radarScore={};
  radarDims.forEach(dim=>radarScore[dim.key]=0);
  for(let i=0;i<currentQuestions.length;i++){
    const sel=currentQuestions[i].selected;
    if(typeof sel!=="number")continue;
    const radar=currentQuestions[i].options[sel].radar;
    for(const k in radar) radarScore[k]=(radarScore[k]||0)+radar[k];
  }
  // 归一 [-4,4]，并四舍五入
  for(const k in radarScore){
    if(radarScore[k]>4)radarScore[k]=4;
    if(radarScore[k]<-4)radarScore[k]=-4;
    radarScore[k]=Math.round(radarScore[k]);
  }
  totalScoreText.textContent = "你的虚拟人格画像已生成";
  promptOutput.textContent = genPrompt(radarScore, gender);
  drawRadar(radarCanvas.getContext("2d"), 340, 340, radarScore, 4);
  radarScoreTextDiv.innerHTML = radarScoreText(radarScore);
}
btnStart.onclick=()=>{
  age=parseInt(inputAge.value)||25;
  gender=selectGender.value;
  questionCount=0;
  currentQuestions=questionPool.slice();
  shuffleArray(currentQuestions);
  currentQuestions=currentQuestions.slice(0,12);
  currentQuestions.forEach(q=>{shuffleArray(q.options);q.selected=undefined;});
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
  questionCount=0;
  sectionResult.style.display="none";
  sectionQuiz.style.display="none";
  sectionUserInfo.style.display="block";
};
</script>
</body>
</html>