# xyz
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Quiz Application</title>

    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: Arial, sans-serif;
        }

        body {
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            background: linear-gradient(135deg, #141e30, #243b55);
            padding: 20px;
        }

        .container {
            background: white;
            width: 100%;
            max-width: 600px;
            padding: 30px;
            border-radius: 18px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.3);
        }

        h1 {
            text-align: center;
            color: #243b55;
            margin-bottom: 10px;
        }

        .subtitle {
            text-align: center;
            color: gray;
            margin-bottom: 25px;
        }

        .top-bar {
            display: flex;
            justify-content: space-between;
            margin-bottom: 20px;
            font-weight: bold;
        }

        .timer {
            color: #e63946;
        }

        .progress {
            width: 100%;
            height: 8px;
            background: #ddd;
            border-radius: 10px;
            margin-bottom: 25px;
            overflow: hidden;
        }

        .progress-bar {
            height: 100%;
            width: 0%;
            background: #4361ee;
            transition: 0.3s;
        }

        #question {
            font-size: 20px;
            margin-bottom: 20px;
            color: #222;
        }

        .option {
            display: block;
            width: 100%;
            padding: 14px;
            margin: 10px 0;
            border: 2px solid #ddd;
            border-radius: 10px;
            background: white;
            text-align: left;
            font-size: 16px;
            cursor: pointer;
            transition: 0.2s;
        }

        .option:hover:not(:disabled) {
            background: #eef2ff;
            border-color: #4361ee;
        }

        .option.selected {
            background: #dbeafe;
            border-color: #4361ee;
        }

        .option:disabled {
            cursor: default;
        }

        .next-btn, .restart-btn {
            width: 100%;
            padding: 14px;
            margin-top: 20px;
            border: none;
            border-radius: 10px;
            background: #4361ee;
            color: white;
            font-size: 17px;
            font-weight: bold;
            cursor: pointer;
        }

        .next-btn:hover, .restart-btn:hover {
            background: #3048c5;
        }

        .result {
            text-align: center;
        }

        .score {
            font-size: 40px;
            color: #4361ee;
            font-weight: bold;
            margin: 20px;
        }

        .review-item {
            padding: 15px;
            margin: 12px 0;
            background: #f1f3f8;
            border-radius: 10px;
            text-align: left;
        }

        .correct {
            color: green;
            font-weight: bold;
        }

        .wrong {
            color: #e63946;
            font-weight: bold;
        }

        .hidden {
            display: none;
        }
    </style>
</head>

<body>

<div class="container">

    <h1>Quiz Application</h1>
    <p class="subtitle">Test Your Knowledge</p>

    <!-- Quiz Section -->
    <div id="quiz-section">

        <div class="top-bar">
            <span id="question-number">Question 1/10</span>
            <span class="timer">
                Time: <span id="time">15</span>s
            </span>
        </div>

        <div class="progress">
            <div class="progress-bar" id="progress-bar"></div>
        </div>

        <h2 id="question">Question appears here</h2>

        <div id="options"></div>

        <button class="next-btn" id="next-btn">
            Next Question
        </button>

    </div>

    <!-- Result Section -->
    <div id="result-section" class="hidden result">

        <h2>Quiz Completed!</h2>

        <div class="score" id="score">0/10</div>

        <h3 id="message"></h3>

        <p id="percentage"></p>

        <h2 style="margin-top:25px;">Answer Review</h2>

        <div id="review"></div>

        <button class="restart-btn" onclick="restartQuiz()">
            Restart Quiz
        </button>

    </div>

</div>

<script>

    // Quiz Questions
    const questions = [

        {
            question: "Which language is used to structure web pages?",
            options: ["Python", "HTML", "Java", "C++"],
            answer: 1
        },

        {
            question: "What does CPU stand for?",
            options: [
                "Central Processing Unit",
                "Computer Personal Unit",
                "Central Program Utility",
                "Control Processing User"
            ],
            answer: 0
        },

        {
            question: "Which of the following is an input device?",
            options: ["Monitor", "Printer", "Keyboard", "Speaker"],
            answer: 2
        },

        {
            question: "Which symbol is used for comments in Python?",
            options: ["//", "#", "/*", "<!--"],
            answer: 1
        },

        {
            question: "Which memory is temporary?",
            options: ["ROM", "Hard Disk", "RAM", "SSD"],
            answer: 2
        },

        {
            question: "What does CSS stand for?",
            options: [
                "Computer Style Sheets",
                "Creative Style System",
                "Cascading Style Sheets",
                "Colorful Style Sheets"
            ],
            answer: 2
        },

        {
            question: "Which company developed Java?",
            options: ["Microsoft", "Sun Microsystems", "Google", "Apple"],
            answer: 1
        },

        {
            question: "Which number system uses only 0 and 1?",
            options: ["Decimal", "Octal", "Hexadecimal", "Binary"],
            answer: 3
        },

        {
            question: "Which operator is used for equality in Python?",
            options: ["=", "==", "!=", "+"],
            answer: 1
        },

        {
            question: "Which part of a computer performs calculations?",
            options: ["ALU", "RAM", "Monitor", "Keyboard"],
            answer: 0
        }

    ];

    let currentQuestion = 0;
    let score = 0;
    let selectedAnswers = [];
    let timeLeft = 15;
    let timer;

    const questionElement = document.getElementById("question");
    const optionsElement = document.getElementById("options");
    const timeElement = document.getElementById("time");
    const nextButton = document.getElementById("next-btn");

    // Display Question
    function loadQuestion() {

        clearInterval(timer);

        timeLeft = 15;
        timeElement.textContent = timeLeft;

        let q = questions[currentQuestion];

        document.getElementById("question-number").textContent =
            `Question ${currentQuestion + 1}/${questions.length}`;

        document.getElementById("progress-bar").style.width =
            ((currentQuestion + 1) / questions.length * 100) + "%";

        questionElement.textContent = q.question;

        optionsElement.innerHTML = "";

        q.options.forEach((option, index) => {

            let button = document.createElement("button");

            button.textContent = option;
            button.classList.add("option");

            button.onclick = () => selectAnswer(index, button);

            optionsElement.appendChild(button);

        });

        nextButton.textContent =
            currentQuestion === questions.length - 1
            ? "Submit Quiz"
            : "Next Question";

        startTimer();

    }

    // Select Answer
    function selectAnswer(index, button) {

        if (selectedAnswers[currentQuestion] !== undefined) {
            return;
        }

        selectedAnswers[currentQuestion] = index;

        document.querySelectorAll(".option").forEach(btn => {
            btn.disabled = true;
        });

        button.classList.add("selected");

    }

    // Timer Function
    function startTimer() {

        timer = setInterval(() => {

            timeLeft--;

            timeElement.textContent = timeLeft;

            if (timeLeft <= 0) {

                clearInterval(timer);

                if (currentQuestion === questions.length - 1) {
                    showResult();
                } else {
                    currentQuestion++;
                    loadQuestion();
                }

            }

        }, 1000);

    }

    // Next Question
    nextButton.onclick = function() {

        clearInterval(timer);

        if (currentQuestion < questions.length - 1) {

            currentQuestion++;
            loadQuestion();

        } else {

            showResult();

        }

    };

    // Show Final Result
    function showResult() {

        clearInterval(timer);

        score = 0;

        questions.forEach((q, index) => {

            if (selectedAnswers[index] === q.answer) {
                score++;
            }

        });

        document.getElementById("quiz-section").classList.add("hidden");

        document.getElementById("result-section").classList.remove("hidden");

        document.getElementById("score").textContent =
            `${score}/${questions.length}`;

        let percentage = (score / questions.length) * 100;

        document.getElementById("percentage").textContent =
            `Your Score: ${percentage}%`;

        document.getElementById("message").textContent =
            percentage >= 80 ? "Excellent Work!" :
            percentage >= 50 ? "Good Job!" :
            "Keep Practicing!";

        showReview();

    }

    // Answer Review
    function showReview() {

        let review = document.getElementById("review");

        review.innerHTML = "";

        questions.forEach((q, index) => {

            let userAnswer = selectedAnswers[index];

            let isCorrect = userAnswer === q.answer;

            let div = document.createElement("div");

            div.classList.add("review-item");

            let yourAnswer = userAnswer === undefined
                ? "Not Answered"
                : q.options[userAnswer];

            div.innerHTML = `
                <p><b>Q${index + 1}:</b> ${q.question}</p>
                <p>Your Answer:
                    <span class="${isCorrect ? 'correct' : 'wrong'}">
                        ${yourAnswer}
                    </span>
                </p>
                <p>Correct Answer:
                    <span class="correct">
                        ${q.options[q.answer]}
                    </span>
                </p>
            `;

            review.appendChild(div);

        });

    }

    // Restart Quiz
    function restartQuiz() {

        clearInterval(timer);

        currentQuestion = 0;
        score = 0;
        selectedAnswers = [];

        document.getElementById("result-section").classList.add("hidden");

        document.getElementById("quiz-section").classList.remove("hidden");

        loadQuestion();

    }

    // Start Application
    loadQuestion();

</script>

</body>
</html>