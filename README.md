<!DOCTYPE html>
<html lang="zh">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>多功能外貌基因测验</title>
  <style>
    body { font-family: Arial, sans-serif; padding: 20px; max-width: 600px; margin: auto; }
    .question { margin-bottom: 20px; }
    .options label { display: block; margin: 8px 0; }
    #results { display: none; }
    button { padding: 8px 12px; margin: 5px; }
  </style>
</head>
<body>
  <h1>多功能外貌基因测验（12题）</h1>
  <div id="quiz-container"></div>
  <div id="controls">
    <button id="prev-btn" disabled>上一题</button>
    <button id="next-btn">下一题</button>
    <button id="submit-btn" style="display:none;">提交</button>
  </div>
  <div id="results">
    <h2>测验结果</h2>
    <p id="score"></p>
    <ol id="feedback"></ol>
  </div>

  <script>
    const questions = [
      { q: 'MC1R基因主要影响什么外貌特征？',
        options: ['眼睛颜色', '头发颜色与雀斑', '鼻子形状', '皮肤弹性'],
        answer: 1
      },
      { q: 'HERC2/OCA2基因座与下列哪种颜色相关？',
        options: ['唇色', '牙齿亮度', '眼睛颜色', '发根粗细'],
        answer: 2
      },
      { q: 'EDAR基因的突变会增强以下哪种特征？',
        options: ['眉毛弯曲度', '发丝厚度', '皮肤油脂分泌', '耳垂形状'],
        answer: 1
      },
      { q: 'SLC24A5基因主要影响什么？',
        options: ['骨骼密度', '皮肤颜色', '指甲硬度', '嘴唇形状'],
        answer: 1
      },
      { q: 'TYR基因与以下哪种色素合成直接相关？',
        options: ['黑色素', '角质素', '红色素', '黄色素'],
        answer: 0
      },
      { q: 'IRF6基因突变常导致哪种面部发育异常？',
        options: ['唇腭裂', '扁平足', '多指畸形', '先天性心脏病'],
        answer: 0
      },
      { q: 'SOX10基因与下列哪一系统的发育密切相关？',
        options: ['消化系统', '神经嵴、色素细胞', '免疫系统', '泌尿系统'],
        answer: 1
      },
      { q: 'PAX3基因的变异可引起哪种综合征？',
        options: ['Waardenburg综合征', 'Marfan综合征', 'Down综合征', 'Turner综合征'],
        answer: 0
      },
      { q: 'KITLG基因影响以下哪种表现？',
        options: ['头发颜色', '舌头纹理', '皮脂分泌', '面部对称'],
        answer: 0
      },
      { q: 'FGFR2基因突变常与哪种面部特征异常相关？',
        options: ['唇腭裂', '颅缝早闭症', '多指畸形', '腭部扁平'],
        answer: 1
      },
      { q: 'SLC45A2基因影响人类的哪项外貌？',
        options: ['皮肤光泽度', '眼白纯净度', '指甲生长速度', '肤色与发色'],
        answer: 3
      },
      { q: 'FGF5基因与以下哪种特征关系最密切？',
        options: ['头发生长速度', '牙齿排列', '眉毛形状', '鼻梁高度'],
        answer: 0
      }
    ];

    // 随机打乱题目顺序
    function shuffle(array) {
      for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
    }
    shuffle(questions);

    let current = 0;
    const userAnswers = Array(questions.length).fill(null);

    const container = document.getElementById('quiz-container');
    const prevBtn = document.getElementById('prev-btn');
    const nextBtn = document.getElementById('next-btn');
    const submitBtn = document.getElementById('submit-btn');
    const resultsDiv = document.getElementById('results');
    const scoreP = document.getElementById('score');
    const feedbackOl = document.getElementById('feedback');

    function renderQuestion(index) {
      const qObj = questions[index];
      container.innerHTML = `<div class=\"question\"><h2>第 ${index+1} 题：</h2><p>${qObj.q}</p></div>`;
      const optsDiv = document.createElement('div'); optsDiv.className = 'options';
      qObj.options.forEach((opt, i) => {
        const lbl = document.createElement('label');
        const rd = document.createElement('input');
        rd.type = 'radio'; rd.name = 'option'; rd.value = i;
        if (userAnswers[index] === i) rd.checked = true;
        lbl.appendChild(rd);
        lbl.append(` ${opt}`);
        optsDiv.appendChild(lbl);
      });
      container.appendChild(optsDiv);

      prevBtn.disabled = index === 0;
      nextBtn.style.display = index === questions.length - 1 ? 'none' : 'inline-block';
      submitBtn.style.display = index === questions.length - 1 ? 'inline-block' : 'none';
    }

    function saveAnswer() {
      const sel = document.querySelector('input[name="option"]:checked');
      if (sel) userAnswers[current] = parseInt(sel.value);
    }

    prevBtn.addEventListener('click', () => {
      saveAnswer();
      current--;
      renderQuestion(current);
    });
    nextBtn.addEventListener('click', () => {
      saveAnswer();
      current++;
      renderQuestion(current);
    });
    submitBtn.addEventListener('click', () => {
      saveAnswer();
      showResults();
    });

    function showResults() {
      let score = 0;
      feedbackOl.innerHTML = '';
      questions.forEach((q, i) => {
        const user = userAnswers[i];
        const correct = q.answer;
        const li = document.createElement('li');
        if (user === correct) { score++; li.textContent = `第${i+1}题：回答正确`; }
        else { li.textContent = `第${i+1}题：正确答案——${q.options[correct]}`; }
        feedbackOl.appendChild(li);
      });
      scoreP.textContent = `您答对了 ${score} / ${questions.length} 道题。`;
      document.getElementById('controls').style.display = 'none';
      resultsDiv.style.display = 'block';
    }

    // 初始渲染
    renderQuestion(current);
  </script>
</body>
</html>
