<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<title>简单问卷测试</title>
<style>
  body { max-width: 600px; margin: 20px auto; font-family: sans-serif; }
  .hidden { display: none; }
  .option { padding: 8px; margin: 4px 0; border: 1px solid #ccc; cursor: pointer; }
  .option.selected { background-color: #4f46e5; color: white; }
  button { margin-top: 12px; padding: 10px; width: 100%; }
</style>
</head>
<body>

<div id="startSection">
  <label>年龄（任意数字）:<input type="number" id="ageInput" /></label><br/>
  <label>性别:
    <select id="genderSelect">
      <option value="male">男</option>
      <option value="female">女</option>
    </select>
  </label><br/>
  <button id="startBtn">开始答题</button>
</div>

<div id="quizSection" class="hidden">
  <div id="questionText"></div>
  <div id="optionsContainer"></div>
  <button id="nextBtn" disabled>下一题</button>
  <div id="progressText"></div>
</div>

<div id="resultSection" class="hidden">
  <h3>结果展示</h3>
  <pre id="resultText"></pre>
  <button id="restartBtn">重新开始</button>
</div>

<script>
  const questions = [
    { text: "你喜欢社交吗？", options: ["不喜欢","有点喜欢","喜欢","非常喜欢"] },
    { text: "你情绪稳定吗？", options: ["不稳定","一般","稳定","非常稳定"] },
    { text: "你觉得自己聪明吗？", options: ["不聪明","一般","聪明","非常聪明"] }
  ];

  let currentIndex = 0;
  let selectedOption = -1;
  let answers = [];

  const startSection = document.getElementById("startSection");
  const quizSection = document.getElementById("quizSection");
  const resultSection = document.getElementById("resultSection");

  const ageInput = document.getElementById("ageInput");
  const genderSelect = document.getElementById("genderSelect");
  const startBtn = document.getElementById("startBtn");

  const questionText = document.getElementById("questionText");
  const optionsContainer = document.getElementById("optionsContainer");
  const nextBtn = document.getElementById("nextBtn");
  const progressText = document.getElementById("progressText");

  const resultText = document.getElementById("resultText");
  const restartBtn = document.getElementById("restartBtn");

  startBtn.onclick = () => {
    if(ageInput.value === "") {
      alert("请输入年龄");
      return;
    }
    startSection.classList.add("hidden");
    quizSection.classList.remove("hidden");
    currentIndex = 0;
    answers = [];
    showQuestion(currentIndex);
  };

  restartBtn.onclick = () => {
    resultSection.classList.add("hidden");
    startSection.classList.remove("hidden");
    ageInput.value = "";
    genderSelect.value = "male";
  };

  function showQuestion(idx) {
    selectedOption = -1;
    nextBtn.disabled = true;
    questionText.textContent = (idx + 1) + ". " + questions[idx].text;
    optionsContainer.innerHTML = "";
    questions[idx].options.forEach((opt, i) => {
      const div = document.createElement("div");
      div.textContent = opt;
      div.className = "option";
      div.onclick = () => {
        selectedOption = i;
        nextBtn.disabled = false;
        Array.from(optionsContainer.children).forEach((c, j) => {
          c.classList.toggle("selected", i === j);
        });
      };
      optionsContainer.appendChild(div);
    });
    progressText.textContent = `题目 ${idx+1} / ${questions.length}`;
  }

  nextBtn.onclick = () => {
    if(selectedOption === -1) return;
    answers.push(selectedOption);
    currentIndex++;
    if(currentIndex >= questions.length){
      quizSection.classList.add("hidden");
      showResult();
      resultSection.classList.remove("hidden");
    } else {
      showQuestion(currentIndex);
    }
  };

  function showResult() {
    const age = ageInput.value;
    const gender = genderSelect.value;
    resultText.textContent = `年龄: ${age}\n性别: ${gender}\n答题结果: ${answers.join(", ")}`;
  }
</script>

</body>
</html>
