<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Fun Explosion Game</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #ffefd5;
            transition: background-color 0.5s ease;
            font-family: Arial, sans-serif;
            position: relative;
        }
        .gender-selection {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: rgba(255, 255, 255, 0.7);
            padding: 20px;
            border-radius: 10px;
            z-index: 10;
            opacity: 1;
            animation: fadeIn 1s ease-out;
        }
        @keyframes fadeIn {
            0% { opacity: 0; }
            100% { opacity: 1; }
        }
        .gender-selection button {
            padding: 20px 40px;
            margin: 10px;
            font-size: 30px;
            background-color: transparent;
            border: 2px solid #000;
            border-radius: 50%;
            color: #000;
            cursor: pointer;
            transition: transform 0.2s ease, background-color 0.3s;
        }
        .gender-selection button:hover {
            transform: scale(1.1);
            background-color: rgba(0, 0, 0, 0.1);
        }
        canvas {
            display: block;
            border: 10px solid rgba(255, 255, 255, 0.3);
            box-shadow: 0 0 30px rgba(255, 255, 255, 0.5);
        }
        .score-board {
            position: absolute;
            top: 10px;
            left: 10px;
            font-size: 24px;
            color: #ff69b4;
            background: rgba(0, 0, 0, 0.6);
            border: 2px solid #ff69b4;
            padding: 10px 20px;
            border-radius: 8px;
            display: flex;
            align-items: center;
            z-index: 1;
        }
        .score {
            font-weight: bold;
        }
        .timer {
            position: absolute;
            top: 10px;
            right: 10px;
            font-size: 24px;
            color: #ff69b4;
            background: rgba(0, 0, 0, 0.6);
            border: 2px solid #ff69b4;
            padding: 10px 20px;
            border-radius: 8px;
        }
    </style>
</head>
<body>
    <div class="gender-selection" id="genderSelection">
        <h2>Cinsiyet Seçin</h2>
        <button onclick="startGame('male')" style="color: #1e90ff; border-color: #1e90ff;">♂</button>
        <button onclick="startGame('female')" style="color: #ff69b4; border-color: #ff69b4;">♀</button>
    </div>

    <canvas id="explosionCanvas" style="display: none;"></canvas>
    <div class="score-board" style="display: none;">
        Skor: <span class="score">0</span>
    </div>
    <div class="timer" style="display: none;">
        Kalan Süre: <span id="timeLeft">60</span>s
    </div>

    <script>
        const canvas = document.getElementById('explosionCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.querySelector('.score');
        const timerElement = document.getElementById('timeLeft');
        const genderSelection = document.getElementById('genderSelection');
        let particles = [];
        let score = 0;
        let gender = '';
        let explosionCount = 0;
        let timeLimit = 60;
        let timeLeft = timeLimit;
        let difficulty = 'normal'; // "easy", "hard"
        const malePalette = ['#4d90fe', '#00bfff', '#228b22', '#1e90ff'];
        const femalePalette = ['#ff6347', '#ff69b4', '#c71585', '#ff1493'];
        const colorPalettes = { "male": malePalette, "female": femalePalette };
        let selectedPalette = malePalette;

        // Timer variable
        let timerInterval = null;

        function resizeCanvas() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }

        window.addEventListener('resize', resizeCanvas);

        function startGame(selectedGender) {
            gender = selectedGender;
            genderSelection.style.display = 'none';
            canvas.style.display = 'block';
            document.querySelector('.score-board').style.display = 'flex';
            document.querySelector('.timer').style.display = 'block';
            selectedPalette = colorPalettes[gender];
            resetGame();
            startTimer();
            update();
        }

        function resetGame() {
            score = 0;
            explosionCount = 0;
            scoreElement.textContent = score;
            timeLeft = timeLimit;
            timerElement.textContent = `Kalan Süre: ${timeLeft}s`;
            particles = [];
        }

        function startTimer() {
            if (timerInterval) {
                clearInterval(timerInterval);
            }
            timerInterval = setInterval(updateTimer, 1000);
        }

        function updateTimer() {
            timeLeft--;
            timerElement.textContent = `Kalan Süre: ${timeLeft}s`;
            if (timeLeft <= 0) {
                clearInterval(timerInterval);
                alert("Zaman doldu! Skorunuz: " + score);
                resetGame();
                genderSelection.style.display = 'flex'; // Show gender selection again
            }
        }

        function incrementScore(amount) {
            score += amount;
            scoreElement.textContent = score;
            checkAchievements();
        }

        function checkAchievements() {
            if (explosionCount >= 100 && !achievements["100_explosions"]) {
                achievements["100_explosions"] = true;
                alert("Başarı kazandınız! 'Patlayıcı Usta' Rozeti!");
            }
            if (explosionCount >= 500 && !achievements["500_explosions"]) {
                achievements["500_explosions"] = true;
                alert("Başarı kazandınız! 'Patlama Krallığı' Rozeti!");
            }
        }

        function changeBackgroundColor() {
            const currentColorIndex = Math.floor(Math.random() * selectedPalette.length);
            document.body.style.backgroundColor = selectedPalette[currentColorIndex];
        }

        function increaseDifficulty() {
            if (explosionCount % 10 === 0 && explosionCount > 0) {
                if (difficulty === "normal") {
                    difficulty = "hard";
                    alert("Zorluk seviyesi arttı! Yeni zorluk: Hard");
                }
            }
        }

        canvas.addEventListener('click', (event) => {
            createExplosion(event.clientX, event.clientY);
            changeBackgroundColor();
            incrementScore(1);
        });

        function createExplosion(x, y) {
            increaseDifficulty();
            const numParticles = 100 + explosionCount * (difficulty === "hard" ? 20 : 10);
            const explosionPower = 10 + explosionCount + (difficulty === "hard" ? 5 : 0);
            for (let i = 0; i < numParticles; i++) {
                const angle = Math.random() * 2 * Math.PI;
                const speed = Math.random() * explosionPower;
                const shape = ['ring', 'square', 'triangle', 'x', 'star'][Math.floor(Math.random() * 5)];
                particles.push({
                    x: x,
                    y: y,
                    speedX: Math.cos(angle) * speed,
                    speedY: Math.sin(angle) * speed,
                    size: Math.random() * 5 + 10,
                    alpha: 1,
                    color: `hsl(${Math.random() * 60 + 30}, 100%, 50%)`,
                    shape: shape,
                    lifetime: Math.random() * 5 + 1
                });
            }
            explosionCount++;
        }

        function update() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            particles = particles.filter(p => p.alpha > 0.05);

            const gradient = ctx.createRadialGradient(canvas.width / 2, canvas.height / 2, canvas.width / 4, canvas.width / 2, canvas.height / 2, canvas.width);
            gradient.addColorStop(0, "rgba(0, 0, 0, 0.2)");
            gradient.addColorStop(1, "rgba(0, 0, 0, 0.8)");
            ctx.fillStyle = gradient;
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            for (const particle of particles) {
                particle.x += particle.speedX;
                particle.y += particle.speedY;
                particle.speedX *= 0.99;
                particle.speedY *= 0.99;
                particle.alpha *= 0.98;

                particle.lifetime -= 0.05;
                if (particle.lifetime <= 0) {
                    particle.alpha = 0;
                }

                ctx.globalAlpha = particle.alpha;
                ctx.strokeStyle = particle.color;
                ctx.lineWidth = 2;

                switch (particle.shape) {
                    case 'ring':
                        ctx.beginPath();
                        ctx.arc(particle.x, particle.y, particle.size / 2, 0, 2 * Math.PI);
                        ctx.stroke();
                        break;
                    case 'square':
                        ctx.strokeRect(particle.x - particle.size / 2, particle.y - particle.size / 2, particle.size, particle.size);
                        break;
                    case 'triangle':
                        ctx.beginPath();
                        ctx.moveTo(particle.x, particle.y - particle.size / 2);
                        ctx.lineTo(particle.x - particle.size / 2, particle.y + particle.size / 2);
                        ctx.lineTo(particle.x + particle.size / 2, particle.y + particle.size / 2);
                        ctx.closePath();
                        ctx.stroke();
                        break;
                    case 'x':
                        ctx.beginPath();
                        ctx.moveTo(particle.x - particle.size / 2, particle.y - particle.size / 2);
                        ctx.lineTo(particle.x + particle.size / 2, particle.y + particle.size / 2);
                        ctx.moveTo(particle.x + particle.size / 2, particle.y - particle.size / 2);
                        ctx.lineTo(particle.x - particle.size / 2, particle.y + particle.size / 2);
                        ctx.stroke();
                        break;
                    case 'star':
                        ctx.beginPath();
                        ctx.moveTo(particle.x, particle.y - particle.size / 2);
                        ctx.lineTo(particle.x - particle.size / 2, particle.y + particle.size / 2);
                        ctx.lineTo(particle.x + particle.size / 2, particle.y + particle.size / 2);
                        ctx.lineTo(particle.x, particle.y - particle.size / 2); 
                        ctx.closePath();
                        ctx.stroke();
                        break;
                }
            }

            requestAnimationFrame(update);
        }
    </script>
</body>
</html>
