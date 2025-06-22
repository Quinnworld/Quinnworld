<!DOCTYPE html>
<html lang="zh">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>隐晦行为基因推测测验（含交互效应）</title>
  <style>
    body { font-family: Arial, sans-serif; padding: 20px; max-width: 700px; margin: auto; }
    .question { margin-bottom: 20px; }
    .options label { display: block; margin: 8px 0; cursor: pointer; }
    #results { display: none; margin-top: 30px; }
    button { padding: 8px 12px; margin: 5px; }
    table { border-collapse: collapse; width: 100%; margin-top: 15px; }
    th, td { border: 1px solid #ccc; padding: 8px; text-align: left; }
  </style>
</head>
<body>
  <h1>隐晦行为基因推测测验（含联动与交叉效应）</h1>
  <div id="quiz-container"></div>
  <div id="controls">
    <button id="prev-btn" disabled>上一题</button>
    <button id="next-btn">下一题</button>
    <button id="submit-btn" style="display:none;">提交</button>
  </div>
  <div id="results">
    <h2>测验结果</h2>
    <p id="score"></p>
    <h3>基因型可能性排序</h3>
    <ol id="feedback"></ol>
    <h3>联动与交叉效应推测</h3>
    <table>
      <thead><tr><th>推测基因</th><th>概率</th><th>说明</th></tr></thead>
      <tbody id="interactions"></tbody>
    </table>
  </div>

  <script>
    const questions = [
      { q: '聚会中小酌后，您是否很快会感到面颊发热？', options: ['几乎没感觉','偶尔有点热','经常感到热','总会红晕'], gene:'ALDH2', map:[0,1,2,3] },
      { q: '品尝微苦巧克力时，您能否立刻捕捉到最细微苦味？', options:['几乎察觉不到','稍微察觉','明显苦味','非常敏感'], gene:'TAS2R38', map:[0,1,2,3] },
      { q: '喝完一杯浓咖啡后，您需要多久才能平复心跳？', options:['不到1小时','1-2小时','2-4小时','超过4小时'], gene:'CYP1A2', map:[0,1,2,3] },
      { q: '早餐奶茶或牛奶后，您会感觉肠胃不适吗？', options:['从无不适','偶有轻微','多数次','几乎每次'], gene:'LCT', map:[3,2,1,0] },
      { q: '在甜点面前，您会忍不住多吃几口吗？', options:['一点也不想','偶尔会','大多数情况','经常无法抗拒'], gene:'FGF21', map:[0,1,2,3] },
      { q: '忙碌一天后，您觉得入睡需要多长时间？', options:['很快入眠(<15分钟)','15-30分钟','30-60分钟','>1小时'], gene:'PER3', map:[3,2,1,0] },
      { q: '清晨闹钟响时，您第一反应是？', options:['立即起床','再睡5分钟','反复贪睡','继续沉睡'], gene:'CLOCK', map:[0,1,2,3] },
      { q: '短跑或举重过程，您是否能感受到瞬间爆发力？', options:['极强','较强','一般','较弱'], gene:'ACTN3', map:[3,2,1,0] },
      { q: '长跑中，您通常会感到？', options:['轻松自如','稍感吃力','大部分时间疲惫','难以坚持'], gene:'PPARGC1A', map:[3,2,1,0] },
      { q: '做重大决策时，您是否倾向审慎再三？', options:['绝对谨慎','较为谨慎','偶尔冲动','常常冲动'], gene:'DRD4', map:[0,1,2,3] },
      { q: '喝咖啡或茶后，您的心跳会？', options:['无变化','微增','明显加快','感到不适'], gene:'ADORA2A', map:[0,1,2,3] },
      { q: '轻微划破皮肤时，您会觉得？', options:['几乎无痛','轻微疼痛','较明显疼痛','非常敏感'], gene:'OPRM1', map:[0,1,2,3] }
    ];

    // 预定义联动/交叉效应规则
    const interactionRules = [
      { genes: ['ALDH2','ADORA2A'], infer:'ADH1B', coeff:0.7, desc:'饮酒与咖啡因敏感共同可能指向ADH1B变异' },
      { genes: ['ACTN3','PPARGC1A'], infer:'VEGFA', coeff:0.6, desc:'运动爆发力+耐力高提示血管生长因子增强' },
      { genes: ['CYP1A2','ADORA2A'], infer:'CYP2A6', coeff:0.5, desc:'咖啡因代谢与心跳反应交互影响其他P450酶' },
      { genes: ['TAS2R38','FGF21'], infer:'SLC2A2', coeff:0.4, desc:'味觉敏感与甜食偏好可能关联葡萄糖转运' }
    ];

    function shuffle(arr){ for(let i=arr.length-1;i>0;i--){const j=Math.floor(Math.random()*(i+1));[arr[i],arr[j]]=[arr[j],arr[i]];} }
    shuffle(questions);

    let current=0, answers=Array(questions.length).fill(null);
    const container=document.getElementById('quiz-container'), prev=document.getElementById('prev-btn'), next=document.getElementById('next-btn'), submit=document.getElementById('submit-btn');

    function render(idx){ const q=questions[idx]; container.innerHTML=`<div class="question"><h2>第 ${idx+1} 题：</h2><p>${q.q}</p></div>`;
      const opts=document.createElement('div'); opts.className='options'; q.options.forEach((o,i)=>{const lbl=document.createElement('label');const rd=document.createElement('input');rd.type='radio';rd.name='opt';rd.value=i;if(answers[idx]===i)rd.checked=true;lbl.appendChild(rd);lbl.append(` ${o}`);opts.appendChild(lbl);}); container.appendChild(opts);
      prev.disabled=idx===0; next.style.display=idx===questions.length-1?'none':'inline-block'; submit.style.display=idx===questions.length-1?'inline-block':'none'; }
    prev.onclick=()=>{save();current--;render(current);} ; next.onclick=()=>{save();current++;render(current);} ; submit.onclick=()=>{save();showResults();};
    function save(){ const sel=document.querySelector('input[name="opt"]:checked'); if(sel)answers[current]=parseInt(sel.value); }

    function showResults(){
      // 基因得分
      const scores={}; questions.forEach((q,i)=>{ const v=answers[i]!=null?q.map[answers[i]]:0; scores[q.gene]=(scores[q.gene]||0)+v; });
      const sorted=Object.entries(scores).sort((a,b)=>b[1]-a[1]);
      document.getElementById('score').textContent='基因型评分如下：';
      const fb=document.getElementById('feedback'); fb.innerHTML=''; sorted.forEach(([g,s])=>{ const li=document.createElement('li'); li.textContent=`${g}: ${s}`; fb.appendChild(li); });

      // 交互效应推断
      const tbody=document.getElementById('interactions'); tbody.innerHTML='';
      interactionRules.forEach(rule=>{
        const vals=rule.genes.map(g=>scores[g]||0);
        const prob = ((vals.reduce((a,b)=>a+b,0)/ (rule.genes.length*3)) * rule.coeff).toFixed(2);
        if(prob>0.2){ const tr=document.createElement('tr'); const td1=document.createElement('td'),td2=document.createElement('td'),td3=document.createElement('td');
          td1.textContent=rule.infer; td2.textContent=prob; td3.textContent=rule.desc; tr.append(td1,td2,td3); tbody.appendChild(tr); }
      });

      document.getElementById('controls').style.display='none'; document.getElementById('results').style.display='block';
    }
    render(current);
  </script>
</body>
</html>
