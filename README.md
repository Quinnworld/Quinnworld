// 12个问题，4个选项，每个选项映射一个基因位点
const questions = [
  {
    question: "你更喜欢以下哪种活动？",
    options: ["读书", "运动", "社交", "艺术创作"],
    geneMapping: [0, 1, 2, 3] // 每个选项映射到基因型（例如，0=智力，1=创造力，2=情绪，3=社交）
  },
  {
    question: "你通常如何解决问题？",
    options: ["逻辑推理", "直觉判断", "团队协作", "独立思考"],
    geneMapping: [0, 1, 2, 3]
  },
  {
    question: "你喜欢什么样的工作环境？",
    options: ["安静的", "充满挑战的", "合作的", "自由的"],
    geneMapping: [0, 1, 2, 3]
  },
  {
    question: "在社交场合中，你的表现如何？",
    options: ["外向", "内向", "随机", "随和"],
    geneMapping: [0, 1, 2, 3]
  },
  // 这里简化问题，可以继续添加更多问题...
];

// 用户答案存储
let userAnswers = new Array(questions.length);

// 生成问卷
function generateQuestions() {
  const questionContainer = document.getElementById('questionContainer');
  questions.forEach((q, index) => {
    const questionElement = document.createElement('div');
    questionElement.classList.add('question');
    questionElement.innerHTML = `<p>${q.question}</p>`;

    q.options.forEach((option, i) => {
      const answerElement = document.createElement('div');
      answerElement.classList.add('answer');
      answerElement.innerHTML = `
        <input type="radio" name="q${index}" value="${i}" onclick="storeAnswer(${index}, ${i})">
        <label>${option}</label>
      `;
      questionElement.appendChild(answerElement);
    });

    questionContainer.appendChild(questionElement);
  });
}

// 存储答案
function storeAnswer(index, optionIndex) {
  userAnswers[index] = optionIndex;
}

// 提交问卷并生成报告
function submitAnswers() {
  if (userAnswers.includes(undefined)) {
    alert("请完成所有问题!");
    return;
  }

  const reportContent = generateReport(userAnswers);
  const bookRecommendations = generateBookRecommendations(userAnswers);

  // 显示报告内容
  document.getElementById('resultContainer').style.display = 'block';
  document.getElementById('reportContent').innerHTML = reportContent;
  
  // 显示推荐书籍
  displayBookRecommendations(bookRecommendations);
}

// 生成报告内容
function generateReport(answers) {
  const scores = calculateScores(answers);
  let report = `<p>您的个性评分如下：</p><ul>`;
  
  report += `
    <li>智力：${scores.intelligence}</li>
    <li>情绪管理：${scores.emotion}</li>
    <li>创造力：${scores.creativity}</li>
    <li>社交能力：${scores.social}</li>
  `;
  report += `</ul>`;

  return report;
}

// 计算得分（可以根据基因型映射和答案计算得分）
function calculateScores(answers) {
  let scores = {
    intelligence: 0,
    emotion: 0,
    creativity: 0,
    social: 0
  };

  answers.forEach((answerIndex, questionIndex) => {
    const mapping = questions[questionIndex].geneMapping[answerIndex];
    // 假设每个答案都给一个固定分数，或者可以根据实际需要进一步修改
    if (mapping === 0) scores.intelligence++;
    if (mapping === 1) scores.creativity++;
    if (mapping === 2) scores.emotion++;
    if (mapping === 3) scores.social++;
  });

  return scores;
}

// 生成书籍推荐
function generateBookRecommendations(answers) {
  // 示例书籍推荐（这里可以扩展为更多书籍）
  const bookLibrary = [
    { title: "批判性思维", author: "理查德·保罗", genre: "思维训练", category: "智力" },
    { title: "情绪的智慧", author: "丹尼尔·戈尔曼", genre: "情绪管理", category: "情绪" },
    { title: "创新者的解答", author: "克莱顿·M·克里斯坦森", genre: "创新思维", category: "创造力" },
    { title: "高效能人士的七个习惯", author: "史蒂芬·柯维", genre: "职业发展", category: "社交" }
  ];

  let recommendations = [];

  // 基于分数生成推荐
  const scores = calculateScores(answers);
  if (scores.intelligence > 1) recommendations.push(bookLibrary[0]);
  if (scores.emotion > 1) recommendations.push(bookLibrary[1]);
  if (scores.creativity > 1) recommendations.push(bookLibrary[2]);
  if (scores.social > 1) recommendations.push(bookLibrary[3]);

  return recommendations;
}

// 显示推荐书籍
function displayBookRecommendations(books) {
  const bookListElement = document.getElementById('bookList');
  books.forEach(book => {
    const bookItem = document.createElement('div');
    bookItem.className = 'book-item';
    bookItem.innerHTML = `
      <h3>${book.title}</h3>
      <p>作者: ${book.author}</p>
      <p>类别: ${book.genre}</p>
    `;
    bookListElement.appendChild(bookItem);
  });
}

// 初始化问卷
generateQuestions();
