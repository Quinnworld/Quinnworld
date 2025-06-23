<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>基因型推测问卷</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f7f7f7;
            color: #333;
            overflow: hidden; /* 禁止页面滚动 */
            height: 100vh; /* 设置为视口高度 */
        }

        .container {
            width: 100%;
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
            background-color: white;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
            height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
        }

        h1 {
            text-align: center;
            color: #4CAF50;
            margin: 0;
        }

        .question {
            margin-bottom: 20px;
            display: none; /* 默认隐藏所有问题 */
        }

        .question label {
            display: block;
            font-size: 16px;
            margin-bottom: 8px;
        }

        .question input[type="radio"] {
            margin-right: 10px;
        }

        .question .options {
            margin-bottom: 10px;
        }

        .btn-submit {
            display: block;
            width: 100%;
            padding: 12px;
            background-color: #4CAF50;
            color: white;
            border: none;
            font-size: 16px;
            cursor: pointer;
            border-radius: 5px;
        }

        .result {
            margin-top: 20px;
            padding: 20px;
            background-color: #fff;
            border: 1px solid #ddd;
            display: none;
        }

        .result h3 {
            text-align: center;
            color: #4CAF50;
        }

        .question.active {
            display: block; /* 显示当前问题 */
        }
    </style>
</head>
<body>

    <div class="container">
        <h1>基因型推测问卷</h1>

        <form id="surveyForm">
            <!-- 问题 1 -->
            <div class="question" id="question1">
                <label>你多久进行一次运动？</label>
                <div class="options">
                    <input type="radio" name="exercise" value="rarely"> 从不运动
                    <input type="radio" name="exercise" value="occasionally"> 偶尔（每周1-2次）
                    <input type="radio" name="exercise" value="frequently"> 经常（每周3-4次）
                    <input type="radio" name="exercise" value="daily"> 每天都运动
                </div>
                <button type="button" class="btn-submit" onclick="nextQuestion(2)">下一题</button>
            </div>

            <!-- 问题 2 -->
            <div class="question" id="question2">
                <label>你觉得自己容易增重吗？</label>
                <div class="options">
                    <input type="radio" name="weight" value="easily_gain"> 很容易
                    <input type="radio" name="weight" value="slightly_easily"> 稍微容易
                    <input type="radio" name="weight" value="hardly_gain"> 不容易
                    <input type="radio" name="weight" value="never_gain"> 一点都不容易
                </div>
                <button type="button" class="btn-submit" onclick="nextQuestion(3)">下一题</button>
            </div>

            <!-- 问题 3 -->
            <div class="question" id="question3">
                <label>你喜欢吃哪些食物？</label>
                <div class="options">
                    <input type="radio" name="diet" value="high_fat"> 高脂肪的食物（如油炸食品、薯片等）
                    <input type="radio" name="diet" value="high_protein"> 偏重蛋白质食物（如肉类、鱼类）
                    <input type="radio" name="diet" value="vegetables_fruits"> 多吃蔬菜和水果
                    <input type="radio" name="diet" value="sweet_foods"> 偏甜食（如蛋糕、糖果）
                </div>
                <button type="button" class="btn-submit" onclick="nextQuestion(4)">下一题</button>
            </div>

            <!-- 问题 4 -->
            <div class="question" id="question4">
                <label>你平时是否有皮肤过敏的情况？</label>
                <div class="options">
                    <input type="radio" name="skin_allergy" value="often"> 经常过敏
                    <input type="radio" name="skin_allergy" value="occasionally"> 偶尔过敏
                    <input type="radio" name="skin_allergy" value="rarely"> 很少过敏
                    <input type="radio" name="skin_allergy" value="never"> 从不过敏
                </div>
                <button type="button" class="btn-submit" onclick="nextQuestion(5)">下一题</button>
            </div>

            <!-- 问题 5 -->
            <div class="question" id="question5">
                <label>你在阳光下待的时间通常有多久？</label>
                <div class="options">
                    <input type="radio" name="sun_exposure" value="rarely"> 很少在阳光下
                    <input type="radio" name="sun_exposure" value="less_1_hour"> 1小时以内
                    <input type="radio" name="sun_exposure" value="1_2_hours"> 1-2小时
                    <input type="radio" name="sun_exposure" value="more_2_hours"> 超过2小时
                </div>
                <button type="button" class="btn-submit" onclick="generatePrompt()">生成你的外貌描述</button>
            </div>
        </form>

        <div class="result" id="result">
            <h3>根据你的回答，生成的外貌描述是：</h3>
            <p id="description"></p>
        </div>
    </div>

    <script>
        let currentQuestion = 1;

        // 显示当前问题
        function showQuestion(questionNumber) {
            // 隐藏所有问题
            let questions = document.querySelectorAll('.question');
            questions.forEach((question) => {
                question.classList.remove('active');
            });
            
            // 显示当前问题
            document.getElementById('question' + questionNumber).classList.add('active');
        }

        // 显示下一个问题
        function nextQuestion(nextQuestionNumber) {
            currentQuestion = nextQuestionNumber;
            showQuestion(currentQuestion);
        }

        // 生成外貌描述
        function generatePrompt() {
            let exercise = document.querySelector('input[name="exercise"]:checked');
            let weight = document.querySelector('input[name="weight"]:checked');
            let diet = document.querySelector('input[name="diet"]:checked');
            let skin_allergy = document.querySelector('input[name="skin_allergy"]:checked');
            let sun_exposure = document.querySelector('input[name="sun_exposure"]:checked');

            if (!exercise || !weight || !diet || !skin_allergy || !sun_exposure) {
                alert('请回答所有问题');
                return;
            }

            let prompt = "基于你的回答，生成的外貌描述如下：";

            if (exercise.value === 'rarely' || exercise.value === 'occasionally') {
                prompt += "你似乎没有很多运动，体型可能较为圆润。";
            } else if (exercise.value === 'frequently' || exercise.value === 'daily') {
                prompt += "你经常运动，体型可能较为结实，肌肉线条清晰。";
            }

            if (weight.value === 'easily_gain') {
                prompt += "你容易增重，体型可能偏圆润。";
            } else if (weight.value === 'hardly_gain') {
                prompt += "你较难增重，体型可能偏瘦。";
            }

            if (diet.value === 'high_fat') {
                prompt += "你喜欢吃高脂肪食物，可能体型偏圆润。";
            } else if (diet.value === 'vegetables_fruits') {
                prompt += "你偏爱蔬菜和水果，体型较为健康。";
            }

            if (sun_exposure.value === 'more_2_hours') {
                prompt += "你常在阳光下，皮肤可能较为健康的小麦色。";
            } else if (sun_exposure.value === 'less_1_hour') {
                prompt += "你很少在阳光下，皮肤可能偏白。";
            }

            if (skin_allergy.value === 'often') {
                prompt += "你的皮肤较为敏感，可能容易出现过敏反应。";
            }

            document.getElementById('description').textContent = prompt;
            document.getElementById('result').style.display = 'block';
        }

        // 初始化显示第一个问题
        showQuestion(currentQuestion);
    </script>
</body>
</html>
