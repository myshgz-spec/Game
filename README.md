<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>مسابقة المقارنة بين الأعداد - الرياضيات للصف الثاني</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Cairo:wght@600;800;900&display=swap');

        * {
            box-sizing: border-box;
            font-family: 'Cairo', sans-serif;
            user-select: none;
        }

        body {
            background: linear-gradient(135deg, #a8ff78 0%, #78ffd6 100%);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            margin: 0;
            padding: 20px;
        }

        .game-card {
            background: white;
            border-radius: 30px;
            padding: 30px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.15);
            text-align: center;
            max-width: 600px;
            width: 100%;
            border: 8px solid #FFD700;
            position: relative;
        }

        .header-title {
            color: #ff4757;
            font-size: 26px;
            margin-bottom: 5px;
            text-shadow: 2px 2px #ffeaa7;
        }

        .sub-title {
            color: #2ed573;
            font-size: 18px;
            margin-bottom: 20px;
        }

        .stats-bar {
            display: flex;
            justify-content: space-around;
            background: #f1f2f6;
            padding: 10px;
            border-radius: 20px;
            margin-bottom: 25px;
            font-size: 20px;
            font-weight: bold;
        }

        .score-box { color: #ff6b81; }
        .question-box { color: #1e90ff; }

        .comparison-area {
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 15px;
            margin-bottom: 30px;
        }

        .number-box {
            background: #ff7f50;
            color: white;
            font-size: 55px;
            font-weight: 900;
            width: 110px;
            height: 110px;
            display: flex;
            justify-content: center;
            align-items: center;
            border-radius: 25px;
            box-shadow: 0 8px 0 #e056fd;
            border: 4px solid #fff;
        }

        .symbol-box {
            background: #dfe4ea;
            color: #2f3542;
            font-size: 50px;
            width: 80px;
            height: 80px;
            display: flex;
            justify-content: center;
            align-items: center;
            border-radius: 50%;
            border: 4px dashed #70a1ff;
        }

        .buttons-container {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin-bottom: 20px;
        }

        .btn-op {
            background: #70a1ff;
            color: white;
            font-size: 45px;
            border: none;
            width: 85px;
            height: 85px;
            border-radius: 20px;
            cursor: pointer;
            transition: all 0.2s;
            box-shadow: 0 6px 0 #1e90ff;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .btn-op:hover {
            transform: translateY(-4px);
            background: #1e90ff;
        }

        .btn-op:active {
            transform: translateY(4px);
            box-shadow: none;
        }

        .feedback-message {
            font-size: 24px;
            font-weight: bold;
            height: 40px;
            margin-top: 10px;
        }

        .correct { color: #2ed573; }
        .wrong { color: #ff4757; }

        .btn-restart {
            background: #2ed573;
            color: white;
            font-size: 22px;
            padding: 12px 30px;
            border: none;
            border-radius: 15px;
            cursor: pointer;
            box-shadow: 0 5px 0 #26af5f;
            display: none;
            margin: 20px auto 0;
        }

        .btn-restart:hover {
            background: #26af5f;
        }

        /* حركات احتفالية */
        @keyframes bounce {
            0%, 100% { transform: scale(1); }
            50% { transform: scale(1.15); }
        }

        .bounce {
            animation: bounce 0.4s ease;
        }
    </style>
</head>
<body>

<div class="game-card">
    <h1 class="header-title">🏆 بطل الرياضيات الصغير 🏆</h1>
    <div class="sub-title">المنهاج الفلسطيني - قارن بين الأعداد التالية</div>

    <div class="stats-bar">
        <div class="score-box">النقاط: <span id="score">0</span> ⭐</div>
        <div class="question-box">السؤال: <span id="q-count">1</span> / 10</div>
    </div>

    <div class="comparison-area" id="comp-area">
        <div class="number-box" id="num1">0</div>
        <div class="symbol-box" id="symbol">؟</div>
        <div class="number-box" id="num2">0</div>
    </div>

    <div class="buttons-container" id="controls">
        <button class="btn-op" onclick="checkAnswer('>')">&gt;</button>
        <button class="btn-op" onclick="checkAnswer('=')">=</button>
        <button class="btn-op" onclick="checkAnswer('<')">&lt;</button>
    </div>

    <div class="feedback-message" id="feedback"></div>
    <button class="btn-restart" id="restart-btn" onclick="startGame()">العب من جديد 🔄</button>
</div>

<script>
    let num1, num2;
    let score = 0;
    let currentQuestion = 1;
    const totalQuestions = 10;

    // إعداد المؤثرات الصوتية بواسطة Web Audio API (لا تحتاج ملفات خارجية)
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();

    function playSound(type) {
        if (audioCtx.state === 'suspended') {
            audioCtx.resume();
        }
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.connect(gain);
        gain.connect(audioCtx.destination);

        if (type === 'correct') {
            osc.type = 'sine';
            osc.frequency.setValueAtTime(523.25, audioCtx.currentTime); // C5
            osc.frequency.setValueAtTime(659.25, audioCtx.currentTime + 0.1); // E5
            osc.frequency.setValueAtTime(783.99, audioCtx.currentTime + 0.2); // G5
            gain.gain.setValueAtTime(0.3, audioCtx.currentTime);
            gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.4);
            osc.start();
            osc.stop(audioCtx.currentTime + 0.4);
        } else if (type === 'wrong') {
            osc.type = 'sawtooth';
            osc.frequency.setValueAtTime(200, audioCtx.currentTime);
            osc.frequency.setValueAtTime(150, audioCtx.currentTime + 0.15);
            gain.gain.setValueAtTime(0.3, audioCtx.currentTime);
            gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.3);
            osc.start();
            osc.stop(audioCtx.currentTime + 0.3);
        } else if (type === 'win') {
            osc.type = 'triangle';
            osc.frequency.setValueAtTime(400, audioCtx.currentTime);
            osc.frequency.setValueAtTime(800, audioCtx.currentTime + 0.2);
            gain.gain.setValueAtTime(0.4, audioCtx.currentTime);
            gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.6);
            osc.start();
            osc.stop(audioCtx.currentTime + 0.6);
        }
    }

    function startGame() {
        score = 0;
        currentQuestion = 1;
        document.getElementById('score').textContent = score;
        document.getElementById('controls').style.display = 'flex';
        document.getElementById('restart-btn').style.display = 'none';
        document.getElementById('feedback').textContent = '';
        generateQuestion();
    }

    function generateQuestion() {
        document.getElementById('q-count').textContent = currentQuestion;
        document.getElementById('symbol').textContent = '؟';
        document.getElementById('feedback').textContent = '';

        // توليد أعداد ضمن 99 (المنهاج الفلسطيني)
        num1 = Math.floor(Math.random() * 100);
        
        // إمكانية جعل الرقمين متساويين بنسبة 25%
        if (Math.random() < 0.25) {
            num2 = num1;
        } else {
            num2 = Math.floor(Math.random() * 100);
        }

        document.getElementById('num1').textContent = num1;
        document.getElementById('num2').textContent = num2;
        
        // إضافة حركة تجعل الأرقام تقفز عند التغيير
        const compArea = document.getElementById('comp-area');
        compArea.classList.remove('bounce');
        void compArea.offsetWidth; // Trigger reflow
        compArea.classList.add('bounce');
    }

    function checkAnswer(selectedSymbol) {
        let correctSymbol = '';
        if (num1 > num2) correctSymbol = '>';
        else if (num1 < num2) correctSymbol = '<';
        else correctSymbol = '=';

        const feedbackEl = document.getElementById('feedback');
        document.getElementById('symbol').textContent = selectedSymbol;

        if (selectedSymbol === correctSymbol) {
            playSound('correct');
            score += 10;
            document.getElementById('score').textContent = score;
            feedbackEl.className = 'feedback-message correct';
            feedbackEl.textContent = 'أحسنت يا بطل! إجابة صحيحة 🎉';
        } else {
            playSound('wrong');
            feedbackEl.className = 'feedback-message wrong';
            feedbackEl.textContent = `إجابة خاطئة! الإجابة الصحيحة هي ( ${correctSymbol} ) ❌`;
        }

        // التعطيل المؤقت للأزرار لمنع التكرار السريع
        toggleButtons(false);

        setTimeout(() => {
            currentQuestion++;
            if (currentQuestion <= totalQuestions) {
                toggleButtons(true);
                generateQuestion();
            } else {
                endGame();
            }
        }, 1500);
    }

    function toggleButtons(enable) {
        const buttons = document.querySelectorAll('.btn-op');
        buttons.forEach(btn => btn.disabled = !enable);
    }

    function endGame() {
        playSound('win');
        document.getElementById('controls').style.display = 'none';
        document.getElementById('symbol').textContent = '🏁';
        
        const feedbackEl = document.getElementById('feedback');
        feedbackEl.className = 'feedback-message correct';
        
        if (score >= 80) {
            feedbackEl.innerHTML = `رائع جداً! 🏆 حصلت على ${score} من 100!<br>أنت ممتاز في الرياضيات! ⭐⭐⭐`;
        } else if (score >= 50) {
            feedbackEl.innerHTML = `جيد جداً! 👍 حصلت على ${score} من 100!<br>يمكنك أن تصبح أفضل!`;
        } else {
            feedbackEl.innerHTML = `حاول مرة أخرى! 💪 حصلت على ${score} من 100!<br>تدرب أكثر لتصبح بطلاً!`;
        }

        document.getElementById('restart-btn').style.display = 'block';
    }

    // بدء اللعبة تلقائياً عند الفتح
    window.onload = startGame;
</script>

</body>
</html># Game
