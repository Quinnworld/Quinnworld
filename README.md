// 题库长度假设为100+
const questionPool = [...]; // 100+题目对象数组
const maxQuestions = 12; // 每次答题12题

function shuffleArray(arr) {
  for (let i = arr.length -1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i+1));
    [arr[i], arr[j]] = [arr[j], arr[i]];
  }
  return arr;
}

function getRandomQuiz() {
  const shuffled = shuffleArray([...questionPool]);
  return shuffled.slice(0, maxQuestions);
}

// 调用示例
const quizQuestions = getRandomQuiz();
renderQuestion(quizQuestions[0]);  // 从这12题开始答题
