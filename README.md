<!DOCTYPE html>
<html>
<head>
    <title>Моя гра LPR Kombat</title>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <style>
        /* Стилі з першого кроку */
        canvas { border: 2px solid black; display: block; margin: 0 auto; background: #f0f0f0; }
        body { text-align: center; font-family: sans-serif; margin: 0; padding: 0; background-color: #1a1a2e; color: white; }

        /* Стилі екрану завантаження */
        #loader-wrapper {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            text-align: center;
            width: 100%;
        }

        #progress-container {
            width: 300px;
            height: 20px;
            background-color: #ddd;
            border-radius: 10px;
            margin: 20px auto;
            overflow: hidden;
            border: 1px solid #333;
        }

        #progress-bar {
            width: 0%;
            height: 100%;
            background-color: #4CAF50;
            transition: width 0.1s;
        }

        #loading-text {
            font-weight: bold;
            font-size: 1.2rem;
            color: white;
        }

        /* Стилі для нижньої навігації */
        .bottom-nav {
            position: fixed;
            bottom: 0;
            left: 0;
            width: 100%;
            background-color: #1a1a2e;
            display: flex;
            justify-content: space-around;
            padding: 10px 0;
            box-shadow: 0 -2px 10px rgba(0, 0, 0, 0.5);
        }

        .nav-btn {
            background: none;
            border: none;
            color: white;
            display: flex;
            flex-direction: column;
            align-items: center;
            cursor: pointer;
            font-size: 0.8rem;
            padding: 5px;
            opacity: 0.7;
        }

        .nav-btn:hover {
            opacity: 1;
        }

        .nav-btn .icon {
            width: 30px;
            height: 30px;
            margin-bottom: 5px;
            /* Приклад іконок: використовуємо символи, якщо картинок немає */
            font-size: 24px; 
            line-height: 30px;
        }

        .airdrop-btn {
            color: orange;
        }

        /* Базовий стиль модального вікна */
        .modal {
            display: none;
            position: fixed;
            z-index: 10;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.6);
        }

        .modal-content {
            background-color: #fefefe;
            margin: 15% auto;
            padding: 20px;
            border: 1px solid #888;
            width: 80%;
            max-width: 400px;
            border-radius: 10px;
            color: #333;
            text-align: center;
        }

        .close-btn {
            color: #aaa;
            float: right;
            font-size: 28px;
            font-weight: bold;
            cursor: pointer;
        }

        .close-btn:hover,
        .close-btn:focus {
            color: black;
            text-decoration: none;
        }
        
        .action-btn {
            padding: 10px 20px;
            margin: 5px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-weight: bold;
            width: 100%;
        }

        .replenish { background-color: #4CAF50; color: white; }
        .withdraw { background-color: #f44336; color: white; }

        /* Стилі для завдань */
        #taskList { list-style-type: none; padding: 0; margin-top: 20px; }
        .task-item {
            background-color: #eee;
            margin-bottom: 10px;
            padding: 15px;
            border-radius: 8px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
            color: #222;
        }
        .task-item.completed { background-color: #d4edda; text-decoration: line-through; opacity: 0.7; }
        .claim-btn { background-color: #4CAF50; color: white; border: none; padding: 8px 12px; border-radius: 5px; cursor: pointer; font-size: 0.8rem; }
        .claim-btn:disabled { background-color: #cccccc; cursor: not-allowed; }
        .task-reward { font-weight: bold; color: #007bff; margin-left: 10px; }

        /* Стилі для друзів */
        .friends-stats { margin: 20px 0; font-size: 1.1rem; color: #222; }
        .referral-link-container { text-align: left; margin-top: 15px; }
        #referralLinkInput { width: calc(100% - 80px); padding: 10px; margin-right: 5px; border: 1px solid #ccc; border-radius: 5px; background-color: #f9f9f9; }
        #copyLinkBtn { padding: 10px 15px; background-color: #007bff; color: white; border: none; border-radius: 5px; cursor: pointer; }

        /* Стилі для Mine/Boost */
        .upgrade-btn { background-color: #ff9800; color: white; border: none; padding: 10px 15px; border-radius: 5px; cursor: pointer; margin-top: 5px; width: 100%; }
        .upgrade-btn:disabled { background-color: #cccccc; cursor: not-allowed; }
        .upgrade-cost { font-size: 0.9rem; color: #555; margin-bottom: 20px; }
    </style>
</head>
<body>

    <!-- Екран завантаження -->
    <div id="loader-wrapper">
        <h2>Завантаження гри...</h2>
        <div id="progress-container">
            <div id="progress-bar"></div>
        </div>
        <div id="loading-text">0%</div>
    </div>

    <!-- Основний вміст гри (спочатку прихований) -->
    <div id="game-content" style="display: none;">
        <h1>LPR Kombat</h1>
        <p>Баланс: <strong id="balanceDisplay">0</strong> LPR | Енергія: <strong id="energyDisplay">0/0</strong> | Прибуток/тап: <strong id="profitPerTapDisplayMain">0</strong></p>
        <canvas id="gameCanvas" width="400" height="400"></canvas>
        <p>Натискайте на хом'яка, щоб майнити монети!</p>
    </div>

    <!-- Нижня навігаційна панель -->
    <div class="bottom-nav">
        <button class="nav-btn" id="exchangeBtn">
            <span class="icon">₿</span>
            Exchange
        </button>
        <button class="nav-btn" id="mineBtn">
            <span class="icon">⛏️</span>
            Mine
        </button>
        <button class="nav-btn" id="friendsBtn">
            <span class="icon">👥</span>
            Friends
        </button>
        <button class="nav-btn" id="earnBtn">
            <span class="icon">💰</span>
            Earn
        </button>
        <button class="nav-btn" id="boostBtn">
            <span class="icon">⚡</span>
            Boost
        </button>
        <button class="nav-btn airdrop-btn" id="airdropBtn">
            <span class="icon">✈️</span>
            Airdrop
        </button>
    </div>

    <!-- Модальне вікно Airdrop -->
    <div id="airdropModal" class="modal">
        <div class="modal-content">
            <span class="close-btn" data-modal="airdrop">&times;</span>
            <h2>LPR Airdrop</h2>
            <p>Курс монети LPR: <strong>$0.000564</strong> за 1 LPR</p>
            <div class="modal-actions">
                <button id="replenishBtn" class="action-btn replenish">Поповнити баланс</button>
                <button id="withdrawBtn" class="action-btn withdraw">Вивести LPR</button>
            </div>
        </div>
    </div>

    <!-- Модальне вікно Earn -->
    <div id="earnModal" class="modal">
        <div class="modal-content">
            <span class="close-btn earn-close" data-modal="earn">&times;</span>
            <h2>Завдання (Earn)</h2>
            <ul id="taskList"></ul>
        </div>
    </div>

    <!-- Модальне вікно Friends -->
    <div id="friendsModal" class="modal">
        <div class="modal-content">
            <span class="close-btn friends-close" data-modal="friends">&times;</span>
            <h2>Мої друзі</h2>
            <p>Запрошуйте друзів та отримуйте бонуси (якщо друг досягне 5 рівня, ви отримаєте 3 млн LPR)!</p>
            <div class="friends-stats">
                <p>Запрошено друзів: <strong id="friendsCountDisplay">0</strong></p>
            </div>
            <div class="referral-link-container">
                <label for="referralLinkInput">Ваше реферальне посилання:</label>
                <input type="text" id="referralLinkInput" readonly>
                <button id="copyLinkBtn">Копіювати</button>
            </div>
        </div>
    </div>

    <!-- Модальне вікно Mine -->
    <div id="mineModal" class="modal">
        <div class="modal-content">
            <span class="close-btn mine-close" data-modal="mine">&times;</span>
            <h2>Прокачування (Mine)</h2>
            <h3>⚡ Енергія</h3>
            <p>Максимальна енергія: <strong id="energyLimitDisplay">...</strong></p>
            <button id="upgradeEnergyBtn" class="upgrade-btn">Прокачати ліміт енергії</button>
            <p class="upgrade-cost" id="energyCostDisplay">Вартість: ... LPR</p>
            <h3>👆 Прибуток за клік</h3>
            <p>Поточний прибуток: <strong id="profitPerTapDisplay">...</strong> LPR/клік</p>
            <button id="upgradeTapBtn" class="upgrade-btn">Прокачати прибуток за клік</button>
            <p class="upgrade-cost" id="tapCostDisplay">Вартість: ... LPR</p>
        </div>
    </div>

    <!-- Модальне вікно Boost -->
    <div id="boostModal" class="modal">
        <div class="modal-content">
            <span class="close-btn boost-close" data-modal="boost">&times;</span>
            <h2>⚡ Прискорення (Boost)</h2>
            <h3>🤖 Автоматичний тап</h3>
            <p>Автоматично тапає 5 разів на секунду протягом 3 годин.</p>
            <p>Статус: <strong id="autoTapStatus">Неактивний</strong></p>
            <p id="autoTapTimer" style="display: none;">Залишилось часу: 00:00:00</p>
            <button id="buyAutoTapBtn" class="upgrade-btn">Придбати Автотап</button>
            <p class="upgrade-cost">Вартість: <strong id="autoTapCostDisplay">800</strong> LPR</p>
        </div>
    </div>


    <script>
        // --- Параметри Гри ---
        let userBalance = 245877075;
        let energyPerClick = 1;
        let currentEnergy = 9500;
        let maxEnergy = 9500;
        let profitPerTap = 1;
        let invitedFriendsCount = 0;

        // --- Параметри Прокачування ---
        let energyUpgradeCost = 10000;
        let tapUpgradeCost = 5000;
        let autoTapActive = false;
        let autoTapCost = 800;
        const autoTapDurationMs = 3 * 60 * 60 * 1000; // 3 hours
        let autoTapIntervalId = null;
        let autoTapTimerIntervalId = null;
        let autoTapEndTime = null;

        // --- Отримання елементів DOM ---
        const balanceDisplay = document.getElementById('balanceDisplay');
        const energyDisplay = document.getElementById('energyDisplay');
        const profitPerTapDisplayMain = document.getElementById('profitPerTapDisplayMain');
        const gameContent = document.getElementById('game-content');
        const airdropModal = document.getElementById('airdropModal');
        const earnModal = document.getElementById('earnModal');
        const friendsModal = document.getElementById('friendsModal');
        const mineModal = document.getElementById('mineModal');
        const boostModal = document.getElementById('boostModal');

        // --- Функції оновлення інтерфейсу (UI) ---
        function updateUI() {
            balanceDisplay.innerText = userBalance.toLocaleString();
            energyDisplay.innerText = `${currentEnergy} / ${maxEnergy}`;
            profitPerTapDisplayMain.innerText = profitPerTap;

            // Оновлення Mine модалки
            const energyLimitEl = document.getElementById('energyLimitDisplay');
            if (energyLimitEl) {
                energyLimitEl.innerText = maxEnergy.toLocaleString();
                document.getElementById('profitPerTapDisplay').innerText = profitPerTap.toLocaleString();
                document.getElementById('energyCostDisplay').innerText = `Вартість: ${energyUpgradeCost.toLocaleString()} LPR`;
                document.getElementById('tapCostDisplay').innerText = `Вартість: ${tapUpgradeCost.toLocaleString()} LPR`;
            }
            updateBoostUI();
        }

        // --- Екран завантаження (з першого кроку) ---
        function startLoading() {
            const progressBar = document.getElementById('progress-bar');
            const loadingText = document.getElementById('loading-text');
            const loaderWrapper = document.getElementById('loader-wrapper');
            let progress = 0;

            const interval = setInterval(() => {
                progress += Math.floor(Math.random() * 10) + 1;
                if (progress >= 100) {
                    progress = 100;
                    clearInterval(interval);
                    setTimeout(() => {
                        loaderWrapper.style.display = 'none';
                        gameContent.style.display = 'block';
                        initGame();
                    }, 500);
                }
                progressBar.style.width = progress + '%';
                loadingText.innerText = progress + '%';
            }, 150);
        }
        window.onload = startLoading;

        // --- Основний цикл гри ---
        function initGame() {
            // Тут ініціалізуємо Canvas гру, якщо потрібно малювати хом'яка
            drawPlaceholderHamster();
            updateUI();
        }

        // --- Простий малюнок хом'яка на canvas (placeholder) ---
        function drawPlaceholderHamster() {
            const canvas = document.getElementById('gameCanvas');
            const ctx = canvas.getContext('2d');
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            // Тіло
            ctx.fillStyle = '#d2691e';
            ctx.beginPath();
            ctx.ellipse(200, 200, 100, 80, 0, 0, Math.PI * 2);
            ctx.fill();
            // Око
            ctx.fillStyle = '#fff';
            ctx.beginPath();
            ctx.arc(230, 180, 16, 0, Math.PI * 2);
            ctx.fill();
            ctx.fillStyle = '#000';
            ctx.beginPath();
            ctx.arc(235, 183, 6, 0, Math.PI * 2);
            ctx.fill();
            // Вусики
            ctx.strokeStyle = '#000';
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.moveTo(140, 200);
            ctx.lineTo(100, 190);
            ctx.moveTo(140, 210);
            ctx.lineTo(95, 210);
            ctx.stroke();
        }

        // --- Логіка клікання по хом'яку (Canvas) ---
        const canvas = document.getElementById('gameCanvas');
        canvas.addEventListener('click', handleTap);

        function handleTap() {
            if (currentEnergy >= energyPerClick) {
                currentEnergy -= energyPerClick;
                userBalance += profitPerTap;
                updateUI();
            } else {
                // Можемо показати повідомлення або анімацію нестачі енергії
                // alert("Недостатньо енергії!");
            }
        }

        // Функція пасивного відновлення енергії 
        setInterval(() => {
            if (currentEnergy < maxEnergy) {
                currentEnergy += 10;
                if (currentEnergy > maxEnergy) currentEnergy = maxEnergy;
                updateUI();
            }
        }, 1000);

        // --- Закриття всіх модалок та глобальний обробник ---
        function closeAllModals() {
            airdropModal.style.display = 'none';
            earnModal.style.display = 'none';
            friendsModal.style.display = 'none';
            mineModal.style.display = 'none';
            boostModal.style.display = 'none';
        }
        window.addEventListener('click', (event) => {
            if (event.target.classList.contains('modal')) {
                event.target.style.display = 'none';
            }
        });

        // Закриття модалок на натискання хрестика
        document.querySelectorAll('.close-btn').forEach(btn => {
            btn.addEventListener('click', () => {
                const modal = btn.closest('.modal');
                if (modal) modal.style.display = 'none';
            });
        });

        // --- Логіка Airdrop (кнопки всередині просто alert'и) ---
        document.getElementById('airdropBtn').addEventListener('click', () => { airdropModal.style.display = 'block'; });
        document.getElementById('replenishBtn').addEventListener('click', () => { alert("Відкрито екран поповнення балансу LPR!"); });
        document.getElementById('withdrawBtn').addEventListener('click', () => { alert("Відкрито екран виводу коштів LPR!"); });

        // --- Логіка Earn (Завдання) ---
        const earnBtn = document.querySelector('#earnBtn');
        const earnCloseBtn = document.querySelector('.earn-close');
        const taskListEl = document.getElementById('taskList');

        const tasks = [
            { id: 1, description: "Здобути 1 млн монет LPR (всього)", goal: 1000000, reward: 1500000, completed: false },
            { id: 2, description: "Запросити 5 друзів", goal: 5, reward: 3000000, completed: false }
        ];

        function renderTasks() {
            taskListEl.innerHTML = '';
            tasks.forEach(task => {
                const li = document.createElement('li');
                li.classList.add('task-item');
                if (task.completed) li.classList.add('completed');
                li.innerHTML = `
                    <span style="flex:1;text-align:left">${task.description}</span>
                    <span class="task-reward">+${task.reward.toLocaleString()} LPR</span>
                    <button class="claim-btn" data-task-id="${task.id}" ${task.completed ? 'disabled' : ''}>
                        ${task.completed ? 'Отримано' : 'Отримати'}
                    </button>`;
                taskListEl.appendChild(li);
            });
            attachTaskListeners();
        }

        function attachTaskListeners() {
            document.querySelectorAll('.claim-btn').forEach(button => {
                button.addEventListener('click', (e) => {
                    const id = parseInt(e.target.dataset.taskId);
                    claimReward(id);
                });
            });
        }

        function claimReward(taskId) {
            const task = tasks.find(t => t.id === taskId);
            if (task && !task.completed) {
                // Умова виконана, якщо баланс достатній або це завдання на друзів
                if (userBalance >= task.goal || (task.id === 2 && invitedFriendsCount >= task.goal)) {
                     userBalance += task.reward;
                     task.completed = true;
                     alert(`Ви успішно отримали ${task.reward.toLocaleString()} LPR!`);
                     renderTasks();
                     updateUI();
                } else {
                    alert(`Умова не виконана: потрібно ${task.goal.toLocaleString()} монет або друзів.`);
                }
            }
        }
        function openEarnModal() { renderTasks(); earnModal.style.display = 'block'; }
        earnBtn.addEventListener('click', openEarnModal);
        earnCloseBtn.addEventListener('click', () => { earnModal.style.display = 'none'; });

        function checkTaskConditions() {
            // Перевіряємо поточний стан і відмічаємо завдання як виконані, якщо умова досягнута
            tasks.forEach(task => {
                if (!task.completed) {
                    if (task.id === 1 && userBalance >= task.goal) {
                        task.completed = true;
                    }
                    if (task.id === 2 && invitedFriendsCount >= task.goal) {
                        task.completed = true;
                    }
                }
            });
            renderTasks();
        }

        // --- Логіка Friends ---
        const friendsBtn = document.querySelector('#friendsBtn');
        const frie
