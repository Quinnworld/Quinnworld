<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Genetic Questionnaire with Dynamic Questions</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f4f4f4;
        }
        header {
            background-color: #4CAF50;
            color: white;
            padding: 20px;
            text-align: center;
        }
        .container {
            width: 60%;
            margin: 20px auto;
            padding: 20px;
            background-color: white;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }
        select, button {
            width: 100%;
            padding: 10px;
            margin: 10px 0;
            border: 1px solid #ccc;
            border-radius: 5px;
        }
        button {
            background-color: #4CAF50;
            color: white;
            border: none;
            cursor: pointer;
        }
        button:hover {
            background-color: #45a049;
        }
        .output {
            margin-top: 20px;
            padding: 10px;
            background-color: #f4f4f4;
            border: 1px solid #ccc;
        }
    </style>
</head>
<body>

<header>
    <h1>Genetic Questionnaire</h1>
</header>

<div class="container">
    <h2>Answer the following questions to generate your genetic prompt</h2>
    <div id="questionContainer"></div>
    <button id="submitBtn">Generate Prompt</button>
    
    <div id="promptOutput" class="output"></div>
</div>

<script>
// Define the structure for user answers
let userAnswers = {};
let currentQuestionIndex = 0;
const questionFlow = [
    { question: "What is your age?", name: "age", options: ["<20", "20-40", "40-60", ">60"], nextQuestion: "gender" },
    { question: "What is your gender?", name: "gender", options: ["male", "female", "other"], nextQuestion: "hair_color" },
    { question: "What is your hair color?", name: "hair_color", options: ["black", "brown", "blonde", "red"], nextQuestion: "eye_color" },
    { question: "What is your eye color?", name: "eye_color", options: ["brown", "blue", "green", "gray"], nextQuestion: "skin_color" },
    { question: "What is your skin color?", name: "skin_color", options: ["dark", "medium", "light"], nextQuestion: "height" },
    { question: "What is your height?", name: "height", options: ["short", "medium", "tall"], nextQuestion: null }
];

// Function to display current question
function displayQuestion() {
    if (currentQuestionIndex >= questionFlow.length) {
        generatePrompt();
        return;
    }

    const currentQuestion = questionFlow[currentQuestionIndex];
    const questionContainer = document.getElementById("questionContainer");

    questionContainer.innerHTML = `
        <label for="${currentQuestion.name}">${currentQuestion.question}</label>
        <select id="${currentQuestion.name}" name="${currentQuestion.name}">
            ${currentQuestion.options.map(option => `<option value="${option}">${option}</option>`).join('')}
        </select>
    `;
}

// Function to handle the "Generate Prompt" button click
document.getElementById('submitBtn').addEventListener('click', function() {
    const currentQuestion = questionFlow[currentQuestionIndex];
    const selectedOption = document.getElementById(currentQuestion.name).value;
    userAnswers[currentQuestion.name] = selectedOption;

    currentQuestionIndex++;
    displayQuestion();
});

// Function to generate prompt for painting
function generatePrompt() {
    const { gender, age, hair_color, eye_color, skin_color, height } = userAnswers;

    let prompt = `Anime-style portrait of a ${height} ${gender} character with `;
    prompt += `smooth, ${skin_color} skin and a youthful expression. `;
    prompt += `They have ${hair_color} hair styled neatly and ${eye_color} eyes full of energy. `;
    prompt += `Their clothing is modern and stylish, fitting their personality. `;
    prompt += `The background is an abstract, glowing warm light that gives off a confident and optimistic vibe.`;

    document.getElementById('promptOutput').innerHTML = `
        <h3>Your Generated Painting Prompt:</h3>
        <p>${prompt}</p>
    `;
}

// Start the questionnaire
displayQuestion();
</script>

</body>
</html>
