<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>세 자리수 곱셈 연습</title>
    <!-- Tailwind CSS를 사용해 깔끔하고 예쁜 디자인을 만듭니다. -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8;
        }
        .container {
            max-width: 500px;
            margin: 0 auto;
            padding: 2rem;
            background-color: white;
            border-radius: 1.5rem;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
        }
        .message-box {
            min-height: 4rem;
        }
        /* 모달 스타일 */
        .modal {
            position: fixed;
            z-index: 100;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.4);
            display: none;
            justify-content: center;
            align-items: center;
        }
        .modal-content {
            background-color: white;
            padding: 2rem;
            border-radius: 1rem;
            text-align: center;
            max-width: 90%;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
        }
    </style>
</head>
<body class="flex items-center justify-center min-h-screen">

    <div class="container text-center">
        <h1 class="text-3xl sm:text-4xl font-extrabold text-gray-900 mb-6">세 자리수 곱셈 연습</h1>
        <div class="problem mb-8">
            <div class="text-6xl font-bold text-gray-800" id="num1"></div>
            <div class="flex items-center justify-center text-4xl font-bold text-gray-600 my-2">
                <span>x</span>
                <span id="num2" class="ml-4"></span>
            </div>
        </div>

        <div class="input-area mb-8 flex flex-col items-center">
            <input type="number" id="answer-input" placeholder="정답을 입력하세요" class="w-full sm:w-80 p-4 text-center text-xl border-2 border-gray-300 rounded-xl focus:border-blue-500 focus:outline-none transition-colors">
        </div>

        <div class="message-box mb-8 text-xl font-semibold text-center" id="message"></div>

        <div class="buttons flex flex-col sm:flex-row justify-center gap-4">
            <button id="check-btn" class="w-full sm:w-auto px-8 py-4 bg-blue-500 text-white font-bold rounded-xl shadow-lg hover:bg-blue-600 transition-colors">
                정답 확인
            </button>
            <button id="next-btn" class="w-full sm:w-auto px-8 py-4 bg-gray-400 text-white font-bold rounded-xl shadow-lg cursor-not-allowed transition-colors" disabled>
                다음 문제
            </button>
        </div>

        <div class="flex justify-between items-center mt-8">
            <div class="score text-lg font-medium text-gray-600">
                <span id="score">0</span> / <span id="total-problems">0</span>
            </div>
            <button id="finish-btn" class="px-4 py-2 bg-red-500 text-white font-bold rounded-xl shadow-lg hover:bg-red-600 transition-colors">
                종료
            </button>
        </div>
    </div>

    <!-- 종료 후 결과 모달 -->
    <div id="result-modal" class="modal">
        <div class="modal-content">
            <h2 class="text-2xl font-bold mb-4">결과</h2>
            <p class="text-lg mb-2">총 문제: <span id="modal-total"></span></p>
            <p class="text-lg mb-2">맞힌 문제: <span id="modal-correct"></span></p>
            <p class="text-lg mb-2">틀린 문제: <span id="modal-incorrect"></span></p>
            <p class="text-lg mb-4">소요 시간: <span id="modal-time"></span>초</p>
            <button id="restart-btn" class="px-6 py-3 bg-blue-500 text-white font-bold rounded-xl hover:bg-blue-600 transition-colors">
                다시 시작
            </button>
        </div>
    </div>

    <script>
        // DOM 요소들을 변수에 저장합니다.
        const num1El = document.getElementById('num1');
        const num2El = document.getElementById('num2');
        const answerInput = document.getElementById('answer-input');
        const messageEl = document.getElementById('message');
        const checkBtn = document.getElementById('check-btn');
        const nextBtn = document.getElementById('next-btn');
        const finishBtn = document.getElementById('finish-btn');
        const scoreEl = document.getElementById('score');
        const totalProblemsEl = document.getElementById('total-problems');
        const resultModal = document.getElementById('result-modal');
        const restartBtn = document.getElementById('restart-btn');
        const modalTotalEl = document.getElementById('modal-total');
        const modalCorrectEl = document.getElementById('modal-correct');
        const modalIncorrectEl = document.getElementById('modal-incorrect');
        const modalTimeEl = document.getElementById('modal-time');

        let num1, num2, answer;
        let correctCount = 0;
        let incorrectCount = 0;
        let totalProblems = 0;
        let startTime;

        /**
         * 올림이 여러 번 발생하는 세 자리수 x 한 자리수 문제를 생성하는 함수입니다.
         * 일의 자리와 십의 자리 모두에서 올림이 발생하도록 보장합니다.
         */
        function generateProblem() {
            let hundreds, tens, ones;
            let hasMultipleCarryOver = false;

            // 올림이 여러 번 발생하는 문제만 생성될 때까지 반복합니다.
            while (!hasMultipleCarryOver) {
                // 한 자리수(곱하는 수)를 2부터 9 사이에서 랜덤하게 생성합니다.
                num2 = Math.floor(Math.random() * 8) + 2;
                
                // 각 자리수를 1부터 9 사이에서 랜덤하게 생성합니다.
                hundreds = Math.floor(Math.random() * 9) + 1;
                tens = Math.floor(Math.random() * 9) + 1;
                ones = Math.floor(Math.random() * 9) + 1;

                // 세 자리수를 조합합니다.
                num1 = hundreds * 100 + tens * 10 + ones;

                // 정답을 계산합니다.
                answer = num1 * num2;
                
                // 일의 자리에서 올림이 발생하는지 확인합니다.
                const carryFromOnes = Math.floor((ones * num2) / 10);
                // 십의 자리에서 올림이 발생하는지 확인합니다. (일의 자리 올림을 더한 후 계산)
                const carryFromTens = Math.floor(((tens * num2) + carryFromOnes) / 10);

                // 일의 자리와 십의 자리 모두에서 올림이 발생했는지 확인합니다.
                hasMultipleCarryOver = (carryFromOnes > 0 && carryFromTens > 0);
            }
            
            // 화면에 문제를 표시합니다.
            num1El.textContent = num1;
            num2El.textContent = num2;

            // 입력 필드와 메시지를 초기화합니다.
            answerInput.value = '';
            messageEl.textContent = '';
            answerInput.disabled = false;
            checkBtn.disabled = false;
            checkBtn.classList.remove('bg-gray-400', 'cursor-not-allowed');
            checkBtn.classList.add('bg-blue-500', 'hover:bg-blue-600');
            nextBtn.disabled = true;
            nextBtn.classList.remove('bg-blue-500', 'hover:bg-blue-600');
            nextBtn.classList.add('bg-gray-400', 'cursor-not-allowed');

            // 입력 필드에 자동으로 초점을 맞춥니다.
            answerInput.focus();
        }

        /**
         * 정답을 확인하는 함수입니다.
         */
        function checkAnswer() {
            // 사용자의 입력값을 가져와 숫자로 변환합니다.
            const userAnswer = parseInt(answerInput.value, 10);
            
            // 입력값이 없거나 숫자가 아니면 함수를 종료합니다.
            if (isNaN(userAnswer)) {
                messageEl.textContent = '숫자를 입력해주세요.';
                messageEl.style.color = 'orange';
                return;
            }

            totalProblems++;

            // 정답을 확인하고 메시지를 표시합니다.
            if (userAnswer === answer) {
                messageEl.textContent = '정답입니다! 🎉';
                messageEl.style.color = 'green';
                correctCount++;
            } else {
                messageEl.textContent = `아쉽지만 틀렸어요. 정답은 ${answer}입니다.`;
                messageEl.style.color = 'red';
                incorrectCount++;
            }
            
            // 점수를 업데이트합니다.
            scoreEl.textContent = correctCount;
            totalProblemsEl.textContent = totalProblems;

            // 정답을 확인한 후에는 입력 필드와 확인 버튼을 비활성화하고, 다음 문제 버튼을 활성화합니다.
            answerInput.disabled = true;
            checkBtn.disabled = true;
            checkBtn.classList.remove('bg-blue-500', 'hover:bg-blue-600');
            checkBtn.classList.add('bg-gray-400', 'cursor-not-allowed');
            nextBtn.disabled = false;
            nextBtn.classList.remove('bg-gray-400', 'cursor-not-allowed');
            nextBtn.classList.add('bg-blue-500', 'hover:bg-blue-600');
        }

        /**
         * 게임 종료 및 결과 모달을 보여주는 함수입니다.
         */
        function finishGame() {
            const endTime = Date.now();
            const timeElapsed = Math.floor((endTime - startTime) / 1000);

            modalTotalEl.textContent = totalProblems;
            modalCorrectEl.textContent = correctCount;
            modalIncorrectEl.textContent = incorrectCount;
            modalTimeEl.textContent = timeElapsed;

            resultModal.style.display = 'flex';
        }
        
        /**
         * 게임을 다시 시작하는 함수입니다.
         */
        function restartGame() {
            // 모든 상태를 초기화합니다.
            correctCount = 0;
            incorrectCount = 0;
            totalProblems = 0;
            scoreEl.textContent = correctCount;
            totalProblemsEl.textContent = totalProblems;

            resultModal.style.display = 'none';
            
            // 게임을 다시 시작합니다.
            startGame();
        }

        /**
         * 게임을 시작하는 함수입니다.
         */
        function startGame() {
            startTime = Date.now();
            generateProblem();
        }

        // '정답 확인' 버튼에 클릭 이벤트 리스너를 추가합니다.
        checkBtn.addEventListener('click', checkAnswer);

        // '다음 문제' 버튼에 클릭 이벤트 리스너를 추가합니다.
        nextBtn.addEventListener('click', generateProblem);

        // '종료' 버튼에 클릭 이벤트 리스너를 추가합니다.
        finishBtn.addEventListener('click', finishGame);

        // '다시 시작' 버튼에 클릭 이벤트 리스너를 추가합니다.
        restartBtn.addEventListener('click', restartGame);

        // Enter 키를 눌렀을 때 정답을 확인하도록 합니다.
        answerInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter' && !checkBtn.disabled) {
                checkAnswer();
            }
        });

        // 페이지가 로드되면 게임을 시작합니다.
        window.onload = startGame;
    </script>
</body>
</html>
