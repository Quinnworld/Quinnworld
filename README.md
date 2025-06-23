<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>虚拟形象生成问卷</title>
  <style>
    body { max-width: 700px; margin: 24px auto; font-family: "微软雅黑", sans-serif; background: #fff; padding: 18px; color: #222; }
    h2, h3 { text-align: center; }
    .option { background: white; border: 1px solid #bbb; border-radius: 6px; padding: 10px; margin: 6px 0; cursor: pointer; user-select:none; }
    .option.selected { background: #4f46e5; color: white; border-color: #4338ca; }
    button { width: 100%; padding: 12px; font-size: 16px; border: none; border-radius: 6px; cursor: pointer; color: white; background: #4338ca; }
    button:disabled { background: #a5b4fc; cursor: not-allowed; }
    #promptOutput { white-space: pre-wrap; overflow-x: auto; background: #eee; padding: 10px; border-radius: 6px; font-family: monospace; user-select: all; }
    #radarCanvas { display: block; margin: 22px auto 12px; background: #fff; border-radius: 8px; }
    #totalScoreText { text-align: center; font-weight: bold; margin: 10px 0; font-size: 22px; }
  </style>
</head>
<body>

<h2>虚拟形象生成问卷</h2>

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
  <div id="totalScoreText"></div>
  <h3>AI绘画Prompt</h3>
  <pre id="promptOutput"></pre>
  <h3>六维特征雷达图</h3>
  <canvas id="radarCanvas" width="400" height="400"></canvas>
  <button id="btnRestart">重新开始</button>
</div>

<script>
// 六维雷达指标
const radarDims = [
  {key:"pigment", name:"色素特征"},
  {key:"bone", name:"骨相立体"},
  {key:"body", name:"体型"},
  {key:"hair", name:"毛发特征"},
  {key:"advent", name:"冒险/外倾"},
  {key:"stable", name:"情绪稳定"}
];

// 12题题库，每题4选项，每个选项映射到六维分数（-2~2）
const questionPool = [
  {text:"你喜欢什么样的户外活动？",options:[
    {text:"海边日光浴",radar:{pigment:2,advent:1}},
    {text:"森林徒步",radar:{bone:1,stable:1}},
    {text:"极限运动",radar:{body:1,advent:2}},
    {text:"更喜欢室内",radar:{pigment:-1,body:-1,stable:1}}
  ]},
  {text:"你喜欢什么类型的饮食？",options:[
    {text:"高蛋白健身餐",radar:{body:-2}},
    {text:"高碳水主食",radar:{body:2}},
    {text:"喜欢甜食",radar:{body:2,stable:-1}},
    {text:"清淡素食",radar:{body:-1,pigment:1}}
  ]},
  {text:"你在社交中最常被夸奖什么？",options:[
    {text:"五官立体",radar:{bone:2}},
    {text:"皮肤健康",radar:{pigment:2}},
    {text:"气质出众",radar:{advent:1,stable:1}},
    {text:"身材匀称",radar:{body:2}}
  ]},
  {text:"你喜欢什么风格的服饰？",options:[
    {text:"欧美时尚",radar:{pigment:1,bone:1}},
    {text:"日韩清新",radar:{hair:2}},
    {text:"运动休闲",radar:{body:1,advent:1}},
    {text:"民族风",radar:{pigment:1,advent:1}}
  ]},
  {text:"你在压力下更倾向？",options:[
    {text:"主动社交",radar:{advent:2}},
    {text:"独自消化",radar:{stable:2}},
    {text:"运动释放",radar:{body:1,advent:1}},
    {text:"吃东西缓解",radar:{body:2,stable:-1}}
  ]},
  {text:"你最喜欢的季节？",options:[
    {text:"夏天",radar:{pigment:2}},
    {text:"冬天",radar:{body:1,stable:1}},
    {text:"春天",radar:{hair:1,pigment:1}},
    {text:"秋天",radar:{bone:1,stable:1}}
  ]},
  {text:"你对自己的头发最满意哪些方面？",options:[
    {text:"发质柔顺",radar:{hair:2}},
    {text:"自然卷曲",radar:{hair:1,bone:1}},
    {text:"头发颜色独特",radar:{hair:2,pigment:1}},
    {text:"不太满意",radar:{hair:-2}}
  ]},
  {text:"你笑起来时最突出的特点？",options:[
    {text:"酒窝明显",radar:{bone:2}},
    {text:"牙齿整齐",radar:{bone:1,hair:1}},
    {text:"颧骨分明",radar:{bone:2}},
    {text:"脸型圆润",radar:{bone:1,body:1}}
  ]},
  {text:"你更容易被什么吸引？",options:[
    {text:"与众不同的外表",radar:{pigment:2,hair:1}},
    {text:"深邃的气质",radar:{bone:1,advent:1}},
    {text:"有趣的谈吐",radar:{advent:2}},
    {text:"温和的性格",radar:{stable:2}}
  ]},
  {text:"你偏爱的发色？",options:[
    {text:"自然黑/棕",radar:{hair:1}},
    {text:"金色/浅色",radar:{hair:2,pigment:2}},
    {text:"红色/紫色",radar:{hair:2,pigment:2}},
    {text:"其他亮色",radar:{hair:1,pigment:1}}
  ]},
  {text:"你在童年最显著的外貌变化？",options:[
    {text:"开始长雀斑",radar:{pigment:2}},
    {text:"牙齿或颧骨变化",radar:{bone:2,hair:1}},
    {text:"体型明显改变",radar:{body:2}},
    {text:"基本没变化",radar:{bone:-1,body:-1}}
  ]},
  {text:"你面对新环境时？",options:[
    {text:"兴奋好奇",radar:{advent:2}},
    {text:"谨慎观察",radar:{stable:2}},
    {text:"主动融入",radar:{advent:1,stable:1}},
    {text:"不适应",radar:{advent:-2,stable:-2}}
  ]}
];

// 自然语言描述
function genDesc(radar) {
  let desc = [];
  if (radar.pigment > 2) desc.push("肤色白皙或色素对比分明，发色瞳色醒目");
  if (radar.pigment < -1) desc.push("肤色偏深或皮肤色素低，自然色调");
  if (radar.bone > 2) desc.push("五官立体，轮廓分明，骨相突出");
  if (radar.bone < -1) desc.push("面部线条柔和，骨相平缓");
  if (radar.body > 2) desc.push("身材偏壮或容易胖，体态丰盈");
  if (radar.body < -1) desc.push("身型修长或偏瘦，代谢较快");
  if (radar.hair > 2) desc.push("发量丰富，发质特别，发色亮眼");
  if (radar.hair < -1) desc.push("发量偏少或发质细软");
  if (desc.length < 2) desc.push("外貌特征均衡");
  return desc.join("，");
}
function genPsy(radar) {
  let arr = [];
  if (radar.advent > 2) arr.push("性格外向好奇，喜欢冒险与新鲜体验");
  if (radar.advent < -1) arr.push("性格内向，趋于保守");
  if (radar.stable > 2) arr.push("情绪稳定，自控力强，适应力高");
  if (radar.stable < -1) arr.push("情绪敏感，易波动");
  if (arr.length < 1) arr.push("心理行为中性，适应性好");
  return arr.join("，");
}
// AI绘画Prompt
function genPrompt(radar, gender) {
  let parts = [];
  if (gender === "female") parts.push("young woman,");
  else if (gender === "male") parts.push("young man,");
  else parts.push("person,");
  // 色素
  if (radar.pigment > 2) parts.push("fair skin, vivid eyes, striking hair color,");
  else if (radar.pigment < -1) parts.push("natural tan skin, dark eyes,");
  else parts.push("natural skin,");
  // 骨相
  if (radar.bone > 2) parts.push("high cheekbones, defined jaw, 3D facial features,");
  else if (radar.bone < -1) parts.push("soft gentle facial lines, round face,");
  // 体型
  if (radar.body > 2) parts.push("full figure, healthy body,");
  else if (radar.body < -1) parts.push("slim, slender figure,");
  // 毛发
  if (radar.hair > 2) parts.push("voluminous hair, unique hairstyle,");
  else if (radar.hair < -1) parts.push("thin soft hair,");
  // 风格
  if (radar.advent > 2) parts.push("confident, adventurous, lively vibe,");
  else if (radar.advent < -1) parts.push("reserved, introverted, calm,");
  if (radar.stable > 2) parts.push("emotionally stable, poised,");
  else if (radar.stable < -1) parts.push("sensitive, gentle temperament,");
  parts.push("beautiful lighting, best quality, ultra detailed");
  return parts.join(" ");
}

// 雷达图绘制
function drawRadar(ctx, width, height, radar, maxAbs=4) {
  ctx.clearRect(0,0,width,height);
  const cx = width/2, cy = height/2, r = Math.min(width, height)/2-40, n = radarDims.length;
  ctx.save();
  ctx.translate(cx,cy);
  // 画网格
  ctx.strokeStyle="#ccc";
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
  // 画轴线
  for(let i=0;i<n;i++){
    let angle = i*2*Math.PI/n - Math.PI/2;
    let x = Math.cos(angle)*r, y = Math.sin(angle)*r;
    ctx.beginPath();ctx.moveTo(0,0);ctx.lineTo(x,y);ctx.stroke();
    // 维度文字
    ctx.font="16px sans-serif";
    ctx.textAlign="center";ctx.textBaseline="middle";
    ctx.fillStyle="#222";
    ctx.fillText(radarDims[i].name, Math.cos(angle)*(r+22), Math.sin(angle)*(r+22));
  }
  // 画数据
  ctx.beginPath();
  for(let i=0;i<n;i++){
    let angle = i*2*Math.PI/n - Math.PI/2;
    let v = Math.max(-maxAbs, Math.min(maxAbs, radar[radarDims[i].key]));
    let rr = r*(v+maxAbs)/(2*maxAbs);
    let x = Math.cos(angle)*rr, y = Math.sin(angle)*rr;
    if(i===0)ctx.moveTo(x,y);else ctx.lineTo(x,y);
  }
  ctx.closePath();
  ctx.strokeStyle="#4f46e5";
  ctx.lineWidth=3;
  ctx.stroke();
  ctx.fillStyle="rgba(79,70,229,0.18)";
  ctx.fill();
  ctx.restore();
}

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
  radarScore={pigment:0,bone:0,body:0,hair:0,advent:0,stable:0};
  for(let i=0;i<currentQuestions.length;i++){
    const sel=currentQuestions[i].selected;
    if(typeof sel!=="number")continue;
    const radar=currentQuestions[i].options[sel].radar;
    for(const k in radar) radarScore[k]=(radarScore[k]||0)+radar[k];
  }
  // 归一到[-4,4]用于雷达（12题最大理论绝对值6~8）
  for(const k in radarScore){
    if(radarScore[k]>4)radarScore[k]=4;
    if(radarScore[k]<-4)radarScore[k]=-4;
  }
  // 总结语
  totalScoreText.textContent = "你的虚拟形象特征已生成";
  // prompt
  promptOutput.textContent = genPrompt(radarScore, gender);
  // 雷达
  drawRadar(radarCanvas.getContext("2d"), 400, 400, radarScore, 4);
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