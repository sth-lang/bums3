<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>백운중 이차방정식 카드 놀이</title>
    <style>
        body { 
            font-family: 'Pretendard', -apple-system, sans-serif; 
            background-color: #f3f4f6; 
            text-align: center; 
            padding: 20px; 
            margin: 0;
        }
        h1 { color: #1f2937; font-size: 26px; margin-bottom: 20px; font-weight: 800; }
        
        /* === 게임 화면 스타일 === */
        #game-section { display: block; }
        
        .grid-container {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 10px;
            max-width: 600px;
            margin: 0 auto;
        }
        @media (min-width: 768px) {
            .grid-container { grid-template-columns: repeat(6, 1fr); max-width: 900px; }
        }

        .card {
            aspect-ratio: 1 / 1;
            perspective: 1000px;
            cursor: pointer;
        }
        .card-inner {
            width: 100%; height: 100%;
            transition: transform 0.6s;
            transform-style: preserve-3d;
            position: relative;
        }
        .card.flipped .card-inner {
            transform: rotateY(180deg);
        }
        .card-face {
            width: 100%; height: 100%;
            position: absolute;
            backface-visibility: hidden;
            display: flex;
            align-items: center;
            justify-content: center;
            border-radius: 12px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            font-weight: bold;
            padding: 8px;
            box-sizing: border-box;
            word-break: keep-all;
            text-align: center;
            line-height: 1.3;
        }
        
        /* 카드 앞면 (백운중 X) */
        .card-front {
            background-color: #3b82f6;
            color: white;
            font-size: 18px;
        }
        
        /* 공통 카드 뒷면 스타일 */
        .card-back {
            color: #1f2937;
            transform: rotateY(180deg);
            font-size: 20px;
            white-space: pre-line;
        }

        /* 🟦 문제 카드 (하늘색) */
        .card-back.question {
            background-color: #bae6fd; 
            border: 3px solid #38bdf8;
        }

        /* 🟨 정답 카드 (노란색) */
        .card-back.answer {
            background-color: #fef08a; 
            border: 3px solid #eab308;
        }
        
        /* 🟩 짝을 맞췄을 때 (초록색) - 우선순위 적용 */
        .card.matched .card-back.question,
        .card.matched .card-back.answer {
            background-color: #10b981 !important;
            color: white !important;
            border-color: #10b981 !important;
        }

        /* === 문제 수정 화면 스타일 === */
        #edit-section { 
            display: none; 
            max-width: 600px; 
            margin: 0 auto; 
            background: white; 
            padding: 20px; 
            border-radius: 12px; 
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }
        .edit-row {
            display: flex;
            gap: 10px;
            margin-bottom: 10px;
            align-items: center;
        }
        .edit-row span { font-weight: bold; width: 40px; }
        .edit-row input {
            flex: 1;
            padding: 10px;
            font-size: 16px;
            border: 1px solid #d1d5db;
            border-radius: 6px;
        }

        /* 버튼 스타일 */
        .btn-group {
            margin-top: 30px;
            display: flex;
            justify-content: center;
            gap: 15px;
            flex-wrap: wrap;
        }
        button {
            padding: 12px 24px;
            font-size: 16px;
            font-weight: bold;
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            transition: background-color 0.2s;
        }
        .btn-primary { background-color: #4f46e5; }
        .btn-primary:hover { background-color: #4338ca; }
        .btn-secondary { background-color: #f59e0b; }
        .btn-secondary:hover { background-color: #d97706; }
        .btn-cancel { background-color: #6b7280; }
        .btn-cancel:hover { background-color: #4b5563; }
        .btn-save { background-color: #10b981; width: 100%; margin-top: 15px;}
        .btn-save:hover { background-color: #059669; }
        
        .info-text { font-size: 14px; color: #6b7280; margin-bottom: 20px; }

    </style>
</head>
<body>

    <!-- 1. 게임 화면 -->
    <div id="game-section">
        <h1>백운중 이차방정식 카드 게임</h1>
        <div class="grid-container" id="game-board"></div>
        <div class="btn-group">
            <button class="btn-primary" onclick="startGame()">새 게임 시작 (섞기)</button>
            <button class="btn-secondary" onclick="openEditMode()">내 문제 직접 만들기 ✏️</button>
        </div>
    </div>

    <!-- 2. 문제 수정 화면 -->
    <div id="edit-section">
        <h1>문제 직접 만들기 (12쌍)</h1>
        <p class="info-text">식(하늘색)과 정답(노란색)을 입력해보세요! 완료 후 게임이 바로 시작됩니다.</p>
        <div id="edit-list"></div>
        <button class="btn-save" onclick="saveAndStart()">✅ 저장하고 내 게임 시작하기</button>
        <button class="btn-cancel" onclick="closeEditMode()" style="margin-top: 10px; width: 100%;">취소</button>
    </div>

    <script>
        // 기본 12쌍의 이차방정식
        let currentPairs = [
            { q: "x² - 1 = 0", a: "x = 1\n또는\nx = -1" },
            { q: "x² - 4 = 0", a: "x = 2\n또는\nx = -2" },
            { q: "x² - 9 = 0", a: "x = 3\n또는\nx = -3" },
            { q: "x² - 16 = 0", a: "x = 4\n또는\nx = -4" },
            { q: "x² - 25 = 0", a: "x = 5\n또는\nx = -5" },
            { q: "x² - 36 = 0", a: "x = 6\n또는\nx = -6" },
            { q: "x² - 3x + 2 = 0", a: "x = 1\n또는\nx = 2" },
            { q: "x² - 5x + 6 = 0", a: "x = 2\n또는\nx = 3" },
            { q: "x² - x - 6 = 0", a: "x = -2\n또는\nx = 3" },
            { q: "x² - 4x - 5 = 0", a: "x = -1\n또는\nx = 5" },
            { q: "x² - 7x + 10 = 0", a: "x = 2\n또는\nx = 5" },
            { q: "x² + 4x + 3 = 0", a: "x = -1\n또는\nx = -3" }
        ];

        let cards = [];
        let firstCard = null;
        let secondCard = null;
        let lockBoard = false;
        let matchCount = 0;

        // 게임 시작 함수
        function startGame() {
            const board = document.getElementById('game-board');
            board.innerHTML = '';
            cards = [];
            matchCount = 0;
            
            let idCounter = 0;
            currentPairs.forEach((pair, index) => {
                // 문제 카드는 'question' 속성 추가
                cards.push({ id: idCounter++, type: index, text: pair.q, cardType: 'question' });
                // 정답 카드는 'answer' 속성 추가
                cards.push({ id: idCounter++, type: index, text: pair.a, cardType: 'answer' });
            });

            // 무작위로 섞기
            cards.sort(() => 0.5 - Math.random());

            // 카드 HTML 생성
            cards.forEach((card, index) => {
                const cardElement = document.createElement('div');
                cardElement.classList.add('card');
                cardElement.dataset.type = card.type;
                
                // cardType 변수에 따라 하늘색(question) 또는 노란색(answer) 클래스가 들어갑니다
                cardElement.innerHTML = `
                    <div class="card-inner">
                        <div class="card-face card-front">백운중 ${index + 1}</div>
                        <div class="card-face card-back ${card.cardType}">${card.text}</div>
                    </div>
                `;
                
                cardElement.addEventListener('click', flipCard);
                board.appendChild(cardElement);
            });
        }

        // 카드 뒤집기 로직
        function flipCard() {
            if (lockBoard) return;
            if (this === firstCard) return;

            this.classList.add('flipped');

            if (!firstCard) {
                firstCard = this;
                return;
            }

            secondCard = this;
            checkForMatch();
        }

        // 짝 맞는지 확인
        function checkForMatch() {
            let isMatch = firstCard.dataset.type === secondCard.dataset.type;

            if (isMatch) {
                disableCards();
            } else {
                unflipCards();
            }
        }

        // 맞췄을 때
        function disableCards() {
            firstCard.classList.add('matched');
            secondCard.classList.add('matched');
            firstCard.removeEventListener('click', flipCard);
            secondCard.removeEventListener('click', flipCard);
            resetBoard();
            
            matchCount++;
            if (matchCount === 12) {
                setTimeout(() => alert("🎉 축하합니다! 모든 카드를 맞췄습니다!"), 500);
            }
        }

        // 틀렸을 때 (1초 뒤 원상복구)
        function unflipCards() {
            lockBoard = true;
            setTimeout(() => {
                firstCard.classList.remove('flipped');
                secondCard.classList.remove('flipped');
                resetBoard();
            }, 1000);
        }

        function resetBoard() {
            [firstCard, secondCard, lockBoard] = [null, null, false];
        }

        // === 문제 수정 관련 로직 ===
        
        function openEditMode() {
            document.getElementById('game-section').style.display = 'none';
            document.getElementById('edit-section').style.display = 'block';
            
            const editList = document.getElementById('edit-list');
            editList.innerHTML = '';
            
            currentPairs.forEach((pair, i) => {
                const row = document.createElement('div');
                row.classList.add('edit-row');
                let displayA = pair.a.replace(/\n/g, " ");
                
                row.innerHTML = `
                    <span>${i + 1}번</span>
                    <input type="text" id="edit-q-${i}" value="${pair.q}" placeholder="예: x² - 1 = 0 (문제)">
                    <input type="text" id="edit-a-${i}" value="${displayA}" placeholder="예: x = 1 또는 x = -1 (정답)">
                `;
                editList.appendChild(row);
            });
        }

        function closeEditMode() {
            document.getElementById('edit-section').style.display = 'none';
            document.getElementById('game-section').style.display = 'block';
        }

        function saveAndStart() {
            for (let i = 0; i < 12; i++) {
                let qVal = document.getElementById(`edit-q-${i}`).value;
                let aVal = document.getElementById(`edit-a-${i}`).value;
                
                if(aVal.includes('또는') && !aVal.includes('\n')) {
                    aVal = aVal.replace('또는', '\n또는\n');
                }

                currentPairs[i] = { q: qVal, a: aVal };
            }
            
            closeEditMode();
            startGame();
        }

        window.onload = startGame;
    </script>
</body>
</html>
