<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dynamic Genetic Questionnaire with Gender and Age Effects</title>
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
            width: 50%;
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
    <h1>Dynamic Genetic Questionnaire with Gender and Age Effects</h1>
</header>

<div class="container">
    <h2>Answer the following questions to generate your genetic prompt</h2>
    <div id="questionContainer"></div>
    <button id="submitBtn">Generate Prompt</button>
    
    <div id="promptOutput" class="output"></div>
</div>

<script>
// Initial dataset for genetic associations with probabilities
const GENE_ASSOCIATIONS = {
    'hair_color': {
        'black': { genotype: 'MC1R', probability: 0.4 },
        'brown': { genotype: 'MC1R', probability: 0.5 },
        'blonde': { genotype: 'MC1R', probability: 0.3 },
        'red': { genotype: 'MC1R', probability: 0.2 }
    },
    'eye_color': {
        'brown': { genotype: 'OCA2', probability: 0.7 },
        'blue': { genotype: 'OCA2', probability: 0.5 },
        'green': { genotype: 'OCA2', probability: 0.3 },
        'gray': { genotype: 'OCA2', probability: 0.2 }
    },
    'skin_color': {
        'dark': { genotype: 'SLC24A5', probability: 0.4 },
        'medium': { genotype: 'SLC24A5', probability: 0.6 },
        'light': { genotype: 'SLC24A5', probability: 0.8 }
    }
};

// Dynamic question flow with gender and age effects
let userAnswers = {};
let currentQuestionIndex = 0;
const questionFlow = [
    {
        question: "What is your gender?",
        name: "gender",
        options: ["male", "female", "other"],
        nextQuestion: "age"
    },
    {
        question: "What is your age?",
        name: "age",
        options: ["<20", "20-40", "40-60", ">60"],
        nextQuestion: "hair_color"
    },
    {
        question: "What is your hair color?",
        name: "hair_color",
        options: ["black", "brown", "blonde", "red"],
        nextQuestion: "eye_color"
    },
    {
        question: "What is your eye color?",
        name: "eye_color",
        options: ["brown", "blue", "green", "gray"],
        nextQuestion: "skin_color"
    },
    {
        question: "What is your skin color?",
        name: "skin_color",
        options: ["dark", "medium", "light"],
        nextQuestion: null
    }
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
    // Collect user input
    const currentQuestion = questionFlow[currentQuestionIndex];
    const selectedOption = document.getElementById(currentQuestion.name).value;
    userAnswers[currentQuestion.name] = selectedOption;

    currentQuestionIndex++;

    // Display next question or generate the prompt
    displayQuestion();
});

// Function to compute probabilities with gender and age effects
function calculateProbabilities() {
    let probabilities = {
        'hair_color': GENE_ASSOCIATIONS.hair_color[userAnswers.hair_color].probability,
        'eye_color': GENE_ASSOCIATIONS.eye_color[userAnswers.eye_color].probability,
        'skin_color': GENE_ASSOCIATIONS.skin_color[userAnswers.skin_color].probability
    };

    // Gender effect on hair and skin color
    if (userAnswers.gender === "male") {
        probabilities['hair_color'] *= 1.2;  // Male typically have thicker hair
    } else if (userAnswers.gender === "female") {
        probabilities['skin_color'] *= 1.1;  // Females generally have smoother skin
    }

    // Age effect on skin and hair
    if (userAnswers.age === "<20") {
        probabilities['skin_color'] *= 1.2;  // Younger skin tends to be smoother
    } else if (userAnswers.age === ">60") {
        probabilities['skin_color'] *= 0.8;  // Older skin tends to have wrinkles and spots
        probabilities['hair_color'] *= 0.6;  // Gray hair more common with age
    }

    return probabilities;
}

// Function to generate the final prompt
function generatePrompt() {
    const probabilities = calculateProbabilities();

    const prompt = `Generated prompt for your character: 
        Gender: ${userAnswers.gender}, 
        Age: ${userAnswers.age},
        Hair Color: ${userAnswers.hair_color}, 
        Eye Color: ${userAnswers.eye_color}, 
        Skin Color: ${userAnswers.skin_color}`;

    const predictedGenotype = {
        'hair_color': GENE_ASSOCIATIONS.hair_color[userAnswers.hair_color].genotype,
        'eye_color': GENE_ASSOCIATIONS.eye_color[userAnswers.eye_color].genotype,
        'skin_color': GENE_ASSOCIATIONS.skin_color[userAnswers.skin_color].genotype
    };

    const output = document.getElementById('promptOutput');
    output.innerHTML = `
        <h3>Your Genetic Prompt:</h3>
        <p>${prompt}</p>
        <h4>Predicted Genotype:</h4>
        <p>Hair Color Gene: ${predictedGenotype.hair_color}</p>
        <p>Eye Color Gene: ${predictedGenotype.eye_color}</p>
        <p>Skin Color Gene: ${predictedGenotype.skin_color}</p>
        <h4>Probabilities:</h4>
        <p>Hair Color Probability: ${probabilities.hair_color}</p>
        <p>Eye Color Probability: ${probabilities.eye_color}</p>
        <p>Skin Color Probability: ${probabilities.skin_color}</p>
    `;
}

// Start the questionnaire
displayQuestion();
</script>

</body>
</html>
