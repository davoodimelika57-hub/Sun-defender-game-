# Sun-defender-game-
<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sun Defender - داروخانه دکتر دارابی</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Lalezar&display=swap');
        
        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            font-family: 'Lalezar', cursive;
            background: linear-gradient(to bottom, #87CEEB, #1E90FF);
            user-select: none;
            touch-action: manipulation;
            transition: background 1s ease;
        }
        
        #gameCanvas {
            display: block;
            width: 100%;
            height: 100vh;
        }
        
        #scorePanel {
            position: absolute;
            top: 10px;
            left: 10px;
            background: rgba(255,255,255,0.7);
            padding: 10px;
            border-radius: 10px;
            font-size: 18px;
        }
        
        #levelPanel {
            position: absolute;
            top: 10px;
            right: 10px;
            background: rgba(255,255,255,0.7);
            padding: 10px;
            border-radius: 10px;
            font-size: 18px;
        }
        
        #brand {
            position: absolute;
            top: 10px;
            right: 50%;
            transform: translateX(50%);
            background: rgba(255,215,0,0.8);
            padding: 10px 20px;
            border-radius: 20px;
            font-size: 20px;
            box-shadow: 0 4px 8px rgba(0,0,0,0.2);
        }
        
        #startScreen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.7);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
            z-index: 100;
            text-align: center;
        }
        
        #levelComplete {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.8);
            display: none;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
            z-index: 200;
            text-align: center;
            padding: 20px;
        }
        
        #levelComplete h2 {
            color: #FFD700;
            font-size: 2.5em;
            margin-bottom: 20px;
        }
        
        #levelComplete p {
            font-size: 1.3em;
            max-width: 80%;
            line-height: 1.6;
            margin-bottom: 30px;
        }
        
        #levelComplete a {
            color: #FFD700;
            text-decoration: none;
            font-weight: bold;
        }
        
        #nextLevelButton {
            margin-top: 20px;
            padding: 15px 30px;
            font-size: 20px;
            background: #4CAF50;
            color: white;
            border: none;
            border-radius: 10px;
            cursor: pointer;
            box-shadow: 0 4px 8px rgba(0,0,0,0.2);
        }
        
        #startButton {
            margin-top: 20px;
            padding: 15px 30px;
            font-size: 20px;
            background: #FF8C00;
            color: white;
            border: none;
            border-radius: 10px;
            cursor: pointer;
            box-shadow: 0 4px 8px rgba(0,0,0,0.2);
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas"></canvas>
    
    <div id="scorePanel">امتیاز: <span id="score">0</span></div>
    <div id="levelPanel">مرحله: <span id="level">1</span></div>
    <div id="brand">داروخانه دکتر دارابی</div>
    
    <div id="startScreen">
        <h1 style="font-size: 3em; color: #FFD700;">Sun Defender</h1>
        <p style="font-size: 1.5em; max-width: 80%;">
            از ضد آفتاب استیکی <span style="color: #FFD700;">Vitalayer</span> استفاده کنید!<br>
            اشعه‌های مضر خورشید را جمع کنید!
        </p>
        <button id="startButton">شروع بازی</button>
    </div>

    <div id="levelComplete">
        <h2>تبریک! مرحله تکمیل شد!</h2>
        <p id="scienceFact"></p>
        <p id="sourceLink" style="font-size: 0.9em; color: #aaa;"></p>
        <button id="nextLevelButton">مرحله بعدی</button>
    </div>

    <audio id="bgMusic" loop>
        <source src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3" type="audio/mpeg">
    </audio>
    <audio id="catchSound">
        <source src="https://assets.mixkit.co/sfx/preview/mixkit-positive-interface-beep-221.mp3" type="audio/mpeg">
    </audio>
    <audio id="levelUpSound">
        <source src="https://assets.mixkit.co/sfx/preview/mixkit-achievement-bell-600.mp3" type="audio/mpeg">
    </audio>

    <script>
        // Game setup
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const levelElement = document.getElementById('level');
        const startScreen = document.getElementById('startScreen');
        const startButton = document.getElementById('startButton');
        const levelComplete = document.getElementById('levelComplete');
        const nextLevelButton = document.getElementById('nextLevelButton');
        const scienceFact = document.getElementById('scienceFact');
        const sourceLink = document.getElementById('sourceLink');
        const bgMusic = document.getElementById('bgMusic');
        const catchSound = document.getElementById('catchSound');
        const levelUpSound = document.getElementById('levelUpSound');
        
        // Set canvas size
        function resizeCanvas() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        resizeCanvas();
        window.addEventListener('resize', resizeCanvas);
        
        // Game variables
        let score = 0;
        let level = 1;
        let gameRunning = false;
        let raysRequired = 20; // تغییر: از 8 به 20 افزایش یافت
        let raysCaught = 0;
        
        // Background colors for each level
        const levelBackgrounds = [
            'linear-gradient(to bottom, #87CEEB, #1E90FF)', // Level 1
            'linear-gradient(to bottom, #FF9966, #FF5E62)', // Level 2
            'linear-gradient(to bottom, #A8FF78, #78FFD6)', // Level 3
            'linear-gradient(to bottom, #DA22FF, #9733EE)', // Level 4
            'linear-gradient(to bottom, #FF512F, #DD2476)', // Level 5
            'linear-gradient(to bottom, #00c6ff, #0072ff)', // Level 6
            'linear-gradient(to bottom, #f46b45, #eea849)', // Level 7
            'linear-gradient(to bottom, #7b4397, #dc2430)'  // Level 8
        ];
        
        // Scientific facts about sunscreen with sources
        const sunscreenFacts = [
            {
                text: "ضد آفتاب Vitalayer با SPF 50+ و PA+++ از پوست در برابر اشعه‌های UVA و UVB محافظت می‌کند. این محصول به صورت استیکی فرموله شده و برای استفاده روی صورت و بدن مناسب است.",
                source: "صفحه رسمی اینستاگرام Vitalayer",
                link: "https://www.instagram.com/vitalayer/"
            },
            {
                text: "اشعه UVA باعث پیری زودرس پوست و چین و چروک می‌شود در حالی که UVB باعث آفتاب سوختگی می‌شود. ضد آفتاب‌های طیف گسترده مانند Vitalayer از هر دو نوع محافظت می‌کنند.",
                source: "دیجی‌کالا مگ",
                link: "https://www.digikala.com/mag/"
            },
            {
                text: "برای اثر بخشی کامل، ضد آفتاب Vitalayer باید 15 دقیقه قبل از قرارگیری در معرض آفتاب استفاده شود و هر 2 ساعت یا بعد از شنا/تعریق شدید تجدید شود.",
                source: "وبسایت رسمی Vitalayer",
                link: "https://vitalayer.com"
            },
            {
                text: "ضد آفتاب استیکی Vitalayer به دلیل فرمولاسیون ویژه، برای پوست‌های حساس و کودکان نیز مناسب است و بدون ایجاد چربی یا سفیدی روی پوست باقی می‌ماند.",
                source: "صفحه رسمی اینستاگرام Vitalayer",
                link: "https://www.instagram.com/vitalayer/"
            },
            {
                text: "استفاده منظم از ضد آفتاب می‌تواند تا 80% از پیری زودرس پوست جلوگیری کند. حتی در روزهای ابری هم 80% اشعه UV به پوست می‌رسد!",
                source: "دیجی‌کالا مگ",
                link: "https://www.digikala.com/mag/"
            },
            {
                text: "ضد آفتاب Vitalayer به دلیل بافت استیکی برای مناطق حساس مانند دور چشم و لب‌ها ایده‌آل است و با تعریق یا شستشو به راحتی پاک نمی‌شود.",
                source: "وبسایت رسمی Vitalayer",
                link: "https://vitalayer.com"
            },
            {
                text: "برای محافظت کامل، به اندازه 2 بند انگشت از ضد آفتاب Vitalayer برای صورت و گردن استفاده کنید. این مقدار معادل حدود 1/4 قاشق چایخوری است.",
                source: "صفحه رسمی اینستاگرام Vitalayer",
                link: "https://www.instagram.com/vitalayer/"
            },
            {
                text: "ضد آفتاب‌های با SPF بالا مانند Vitalayer SPF 50+ برای افرادی که ساعات طولانی در معرض آفتاب هستند یا پوست حساسی دارند توصیه می‌شود.",
                source: "دیجی‌کالا مگ",
                link: "https://www.digikala.com/mag/"
            }
        ];
        
        // Sun properties
        const sun = {
            x: canvas.width / 2,
            y: 100,
            radius: 60
        };
        
        // Player (sunscreen) properties
        const player = {
            x: canvas.width / 2 - 60,
            y: canvas.height - 120,
            width: 120,
            height: 80,
            speed: 10,
            color: '#FF5E00'
        };
        
        // Rays array
        let rays = [];
        
        // Start game
        startButton.addEventListener('click', function() {
            startScreen.style.display = 'none';
            score = 0;
            raysCaught = 0;
            level = 1;
            scoreElement.textContent = score;
            levelElement.textContent = level;
            document.body.style.background = levelBackgrounds[level-1];
            gameRunning = true;
            
            // Play music with medium volume
            bgMusic.volume = 0.5;
            bgMusic.play().catch(e => console.log("Auto-play prevented:", e));
            
            gameLoop();
        });
        
        // Next level button
        nextLevelButton.addEventListener('click', function() {
            levelComplete.style.display = 'none';
            gameRunning = true;
            raysCaught = 0;
            document.body.style.background = levelBackgrounds[level-1];
            gameLoop();
        });
        
        // Mouse/Touch controls
        canvas.addEventListener('mousemove', movePlayer);
        canvas.addEventListener('touchmove', function(e) {
            e.preventDefault();
            movePlayer({clientX: e.touches[0].clientX});
        }, { passive: false });
        
        function movePlayer(e) {
            if (!gameRunning) return;
            player.x = e.clientX - player.width/2;
            
            // Boundary check
            if (player.x < 0) player.x = 0;
            if (player.x > canvas.width - player.width) {
                player.x = canvas.width - player.width;
            }
        }
        
        // Create new ray
        function createRay() {
            const types = ['UVA', 'UVB', 'IR'];
            const colors = ['#FF3366', '#33FF66', '#3366FF'];
            
            // Rays can come from anywhere at the top
            rays.push({
                x: Math.random() * canvas.width,
                y: 0,
                width: 12,
                height: 35,
                type: types[Math.floor(Math.random() * 3)],
                color: colors[Math.floor(Math.random() * 3)],
                speed: 1 + (level * 0.3) // Slower speed
            });
        }
        
        // Draw sun with face
        function drawSun() {
            // Sun body
            ctx.beginPath();
            ctx.arc(sun.x, sun.y, sun.radius, 0, Math.PI * 2);
            ctx.fillStyle = '#FFD700';
            ctx.fill();
            
            // Sun face
            ctx.fillStyle = '#000';
            ctx.beginPath();
            ctx.arc(sun.x - 20, sun.y - 15, 10, 0, Math.PI * 2);
            ctx.arc(sun.x + 20, sun.y - 15, 10, 0, Math.PI * 2);
            ctx.fill();
            
            // Smile
            ctx.beginPath();
            ctx.arc(sun.x, sun.y + 10, 25, 0.1 * Math.PI, 0.9 * Math.PI);
            ctx.lineWidth = 5;
            ctx.stroke();
        }
        
        // Draw sunscreen
        function drawSunscreen() {
            // Sunscreen stick
            ctx.fillStyle = player.color;
            ctx.beginPath();
            ctx.roundRect(
                player.x, player.y, 
                player.width, player.height, 
                [0, 0, 15, 15]
            );
            ctx.fill();
            
            // Text
            ctx.fillStyle = '#FFF';
            ctx.font = 'bold 16px Arial';
            ctx.textAlign = 'center';
            ctx.fillText('Vitalayer', player.x + player.width/2, player.y + 35);
            ctx.font = '12px Arial';
            ctx.fillText('SPF 50+', player.x + player.width/2, player.y + 55);
        }
        
        // Check level completion
        function checkLevelCompletion() {
            if (raysCaught >= raysRequired) {
                gameRunning = false;
                levelUpSound.play();
                
                // Show level complete screen with scientific fact
                const fact = sunscreenFacts[(level-1) % sunscreenFacts.length];
                scienceFact.textContent = fact.text;
                sourceLink.innerHTML = `منبع: <a href="${fact.link}" target="_blank">${fact.source}</a>`;
                levelComplete.style.display = 'flex';
                
                // Prepare for next level
                level++;
                levelElement.textContent = level;
                raysRequired = 20 + (level * 2); // تغییر: از 8 به 20 افزایش یافت
                
                // Change background for next level
                if (level <= levelBackgrounds.length) {
                    document.body.style.background = levelBackgrounds[level-1];
                } else {
                    // If all levels completed, loop backgrounds
                    document.body.style.background = levelBackgrounds[(level-1) % levelBackgrounds.length];
                }
            }
        }
        
        // Game loop
        function gameLoop() {
            if (!gameRunning) return;
            
            // Clear canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // Draw background
            ctx.fillStyle = '#87CEEB';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            // Draw sun
            drawSun();
            
            // Create new rays randomly (slower pace)
            if (Math.random() < 0.02 + (level * 0.005)) {
                createRay();
            }
            
            // Draw and update rays
            for (let i = rays.length - 1; i >= 0; i--) {
                const ray = rays[i];
                
                // Draw ray
                ctx.fillStyle = ray.color;
                ctx.fillRect(ray.x, ray.y, ray.width, ray.height);
                
                // Draw ray type
                ctx.fillStyle = '#000';
                ctx.font = 'bold 12px Arial';
                ctx.textAlign = 'center';
                ctx.fillText(ray.type, ray.x + ray.width/2, ray.y + ray.height + 15);
                
                // Move ray (slower)
                ray.y += ray.speed;
                
                // Collision detection with sunscreen
                if (ray.y + ray.height > player.y && 
                    ray.x > player.x && 
                    ray.x < player.x + player.width) {
                    
                    // Play sound
                    catchSound.currentTime = 0;
                    catchSound.play();
                    
                    // Remove ray
                    rays.splice(i, 1);
                    
                    // Increase score and caught count
                    score++;
                    raysCaught++;
                    scoreElement.textContent = score;
                    
                    // Check level completion
                    checkLevelCompletion();
                }
                
                // Remove off-screen rays
                if (ray.y > canvas.height) {
                    rays.splice(i, 1);
                }
            }
            
            // Draw player
            drawSunscreen();
            
            requestAnimationFrame(gameLoop);
        }
        
        // Initial click to enable audio
        document.addEventListener('click', function() {
            bgMusic.play().catch(e => console.log("Auto-play prevented:", e));
        }, { once: true });
    </script>
</body>
</html>
