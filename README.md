# UJIAN-BO-12025<!DOCTYPE html>
<html lang="ms">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Portal Ujian Teori IKBN Kinarut</title>
    <!-- Muat turun Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Konfigurasi Tailwind untuk menggunakan fon Inter -->
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f7f9fb;
        }
        .shadow-custom {
            box-shadow: 0 10px 30px -5px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
        }
    </style>
</head>
<body class="min-h-screen flex items-center justify-center p-4">

    <!-- Container Utama -->
    <div class="w-full max-w-2xl bg-white p-8 md:p-10 rounded-xl shadow-custom border border-gray-100">

        <!-- Header -->
        <header class="text-center mb-8">
            <h1 class="text-4xl font-extrabold text-blue-800 mb-2">IKBN KINARUT</h1>
            <h2 class="text-2xl font-semibold text-gray-700">PORTAL RASMI UJIAN TEORI</h2>
            <p class="text-xl text-red-600 font-bold mt-4 border-t pt-4 border-gray-200">BO 11 SESI 1/2025</p>
        </header>

        <!-- Paparan Pemasa (Tersembunyi pada mulanya) -->
        <div id="timer-container" class="hidden text-center mb-6">
            <div class="bg-blue-100 p-4 rounded-lg shadow-inner">
                <p class="text-lg font-medium text-blue-700">Masa Tinggal:</p>
                <div id="countdown-display" class="text-7xl font-mono font-black text-blue-900 mt-1">60:00</div>
            </div>
            
            <!-- Paparan Amaran -->
            <div id="warning-message" class="mt-4 p-3 rounded-lg font-semibold transition-all duration-300 min-h-[40px] text-sm" role="alert">
                <!-- Amaran akan muncul di sini -->
            </div>
        </div>

        <!-- Borang Kod Akses (Dipaparkan pada mulanya) -->
        <div id="access-form" class="space-y-4">
            <p class="text-center text-lg text-gray-600 font-medium">Sila masukkan Kod Akses untuk memulakan ujian.</p>
            <input type="text" id="access-code-input" placeholder="Kod Akses (Contoh: BO12025U2)"
                   class="w-full p-4 border border-gray-300 rounded-lg focus:ring-blue-500 focus:border-blue-500 text-lg">
            
            <button id="submit-code-btn" onclick="checkAccessCode()"
                    class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-4 rounded-lg transition duration-200 ease-in-out shadow-lg transform hover:scale-[1.01] active:scale-[0.99]">
                Sahkan Kod & Mulakan Ujian
            </button>

            <p id="error-message" class="text-red-500 text-center font-medium hidden">Kod akses salah. Sila cuba lagi.</p>
        </div>

        <!-- Pautan Ujian (Tersembunyi pada mulanya) -->
        <div id="test-link-container" class="hidden mt-8 text-center">
            <p class="text-xl font-medium text-green-700 mb-4">Kod Akses Sah! Sila klik pautan di bawah untuk memulakan ujian anda:</p>
            
            <!-- Link ke Google Form -->
            <a id="test-link" href="https://forms.gle/qjHCwsH85uEXGDKy8" target="_blank" rel="noopener noreferrer"
               class="inline-block w-full max-w-sm bg-green-500 hover:bg-green-600 text-white font-extrabold text-xl py-4 rounded-lg transition duration-300 ease-in-out shadow-xl transform hover:scale-[1.02] active:scale-[0.98]">
                Buka Soalan Ujian (Google Form)
            </a>
            <p class="text-sm text-red-500 mt-3 font-medium">Perhatian: Pemasa akan terus berjalan walaupun anda menutup tetingkap ini.</p>
        </div>

        <!-- Mesej Tamat Masa -->
        <div id="time-up-message" class="hidden mt-8 text-center p-6 bg-red-100 border-l-4 border-red-500 rounded-lg">
            <svg class="mx-auto h-12 w-12 text-red-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
            </svg>
            <h3 class="text-2xl font-bold text-red-700 mt-2">MASA TAMAT!</h3>
            <p class="text-lg text-red-600 mt-1">Ujian telah tamat. Web ditutup.</p>
            <p class="text-md text-red-800 font-semibold mt-3">Sila hantar jawapan anda di Google Form dengan segera.</p>
        </div>

    </div>

    <!-- Logik JavaScript -->
    <script>
        // Data Ujian
        const CORRECT_CODE = "BO12025U2";
        const TOTAL_TIME_SECONDS = 60 * 60; // 60 minit
        
        // Pilihan Amaran (dalam saat dari permulaan)
        const WARNING_TIMES = {
            900: 'amaran-15min', // 15 minit (3600 - 900 = 2700 saat telah berlalu)
            600: 'amaran-10min', // 10 minit
            300: 'amaran-5min'  // 5 minit
        };

        // Elemen DOM
        const accessForm = document.getElementById('access-form');
        const timerContainer = document.getElementById('timer-container');
        const countdownDisplay = document.getElementById('countdown-display');
        const warningMessage = document.getElementById('warning-message');
        const testLinkContainer = document.getElementById('test-link-container');
        const timeUpMessage = document.getElementById('time-up-message');
        const errorMessage = document.getElementById('error-message');
        const submitCodeBtn = document.getElementById('submit-code-btn');
        const accessCodeInput = document.getElementById('access-code-input');

        let timerInterval;
        let timeLeft;
        let isTimeUp = false;

        // --- FUNGSI UTAMA ---

        // Fungsi untuk memeriksa kod akses
        function checkAccessCode() {
            const enteredCode = accessCodeInput.value.trim();
            errorMessage.classList.add('hidden');

            if (enteredCode === CORRECT_CODE) {
                // Simpan masa mula dalam sessionStorage
                const startTime = Date.now();
                sessionStorage.setItem('testStartTime', startTime);
                sessionStorage.setItem('isTestActive', 'true');
                
                // Mulakan pemasa dan UI kemas kini
                startTestUI();
                startTimer();
            } else {
                errorMessage.classList.remove('hidden');
                accessCodeInput.value = ''; // Kosongkan input
            }
        }

        // Fungsi untuk mengemas kini UI selepas kod disahkan
        function startTestUI() {
            accessForm.classList.add('hidden');
            timerContainer.classList.remove('hidden');
            testLinkContainer.classList.remove('hidden');
        }

        // Fungsi untuk memulakan pemasa
        function startTimer() {
            const startTime = sessionStorage.getItem('testStartTime');
            
            if (!startTime) {
                // Sekiranya tiada masa mula, bermakna ada ralat atau tidak melalui semakan kod
                // Tetapi untuk tujuan pemulihan selepas refresh, kita gunakan logic di bawah
                timeLeft = TOTAL_TIME_SECONDS;
                updateTimerDisplay();
                return;
            }

            const elapsedSeconds = Math.floor((Date.now() - parseInt(startTime)) / 1000);
            timeLeft = TOTAL_TIME_SECONDS - elapsedSeconds;

            if (timeLeft <= 0) {
                handleTimeUp();
                return;
            }

            updateTimerDisplay(); // Paparkan masa awal
            checkWarnings();     // Semak amaran awal

            timerInterval = setInterval(() => {
                if (isTimeUp) {
                    clearInterval(timerInterval);
                    return;
                }

                timeLeft--;
                updateTimerDisplay();
                checkWarnings();

                if (timeLeft <= 0) {
                    clearInterval(timerInterval);
                    handleTimeUp();
                }
            }, 1000);
        }

        // Fungsi untuk mengemas kini paparan pemasa
        function updateTimerDisplay() {
            const minutes = Math.floor(timeLeft / 60);
            const seconds = timeLeft % 60;
            const display = `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
            countdownDisplay.textContent = display;

            // Menukar warna pemasa apabila masa hampir tamat
            if (timeLeft <= 300) { // 5 minit
                countdownDisplay.classList.remove('text-blue-900');
                countdownDisplay.classList.add('text-red-600');
            }
        }

        // Fungsi untuk menguruskan amaran masa
        function checkWarnings() {
            const currentWarningTime = timeLeft; 
            let warningText = '';
            let warningClass = '';
            
            if (currentWarningTime === WARNING_TIMES['900']) {
                warningText = 'PERHATIAN: 15 MINIT tinggal sebelum masa ujian tamat!';
                warningClass = 'bg-yellow-100 text-yellow-700';
            } else if (currentWarningTime === WARNING_TIMES['600']) {
                warningText = 'PERINGATAN KRITIKAL: 10 MINIT tinggal. Sila hantar jawapan anda.';
                warningClass = 'bg-orange-100 text-orange-700';
            } else if (currentWarningTime === WARNING_TIMES['300']) {
                warningText = 'AMARAN MUKTAMAD: 5 MINIT tinggal! Ujian akan tamat tidak lama lagi.';
                warningClass = 'bg-red-100 text-red-700';
            } else if (currentWarningTime < 300 && currentWarningTime > 0) {
                 warningText = `300 saat (5 minit) terakhir!`;
                 warningClass = 'bg-red-200 text-red-800 animate-pulse';
            } else {
                warningMessage.classList.remove(...warningMessage.classList);
                warningMessage.classList.add('mt-4', 'p-3', 'rounded-lg', 'font-semibold', 'transition-all', 'duration-300', 'min-h-[40px]', 'text-sm');
                return; // Jangan kemas kini jika tiada amaran
            }

            // Kemas kini paparan amaran
            warningMessage.textContent = warningText;
            warningMessage.className = `mt-4 p-3 rounded-lg font-semibold text-sm transition-all duration-300 ${warningClass}`;
        }

        // Fungsi apabila masa tamat
        function handleTimeUp() {
            isTimeUp = true;
            sessionStorage.removeItem('isTestActive');
            sessionStorage.removeItem('testStartTime');

            // Sembunyikan dan Paparkan Mesej Tamat
            timerContainer.classList.add('hidden');
            testLinkContainer.classList.add('hidden');
            timeUpMessage.classList.remove('hidden');

            // Tutup/Lumpuhkan borang/butang untuk keselamatan
            if (testLinkContainer.querySelector('#test-link')) {
                const link = testLinkContainer.querySelector('#test-link');
                link.href = "#"; // Lumpuhkan pautan
                link.textContent = "MASA TAMAT - Pautan Ujian Dilumpuhkan";
                link.classList.remove('bg-green-500', 'hover:bg-green-600', 'shadow-xl');
                link.classList.add('bg-gray-400', 'cursor-not-allowed', 'shadow-none');
            }
        }

        // --- INICIALISASI PADA MULA MUATAN ---
        window.onload = function() {
            // Semak jika ujian sedang aktif dalam sesi ini (selepas refresh)
            if (sessionStorage.getItem('isTestActive') === 'true' && sessionStorage.getItem('testStartTime')) {
                startTestUI();
                startTimer();
            }
            
            // Allow submission via Enter key
            accessCodeInput.addEventListener('keypress', function(e) {
                if (e.key === 'Enter') {
                    e.preventDefault();
                    checkAccessCode();
                }
            });
        };
    </script>

</body>
</html>
