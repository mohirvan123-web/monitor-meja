<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gacoan Meja Monitor - Real Time</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* CSS tambahan untuk transisi yang lebih halus */
        .meja-card {
            transition: background-color 0.3s ease, transform 0.1s ease;
        }
        /* Mencegah highlight sentuhan pada elemen tombol */
        .no-select {
            -webkit-user-select: none;
            -moz-user-select: none;
            -ms-user-select: none;
            user-select: none;
        }
    </style>
</head>
<body class="bg-gray-50 font-sans min-h-screen">

    <div class="container mx-auto p-4 md:p-8">
        <header class="text-center mb-6">
            <h1 class="text-4xl font-extrabold text-red-700 tracking-wider">GACOAN MEJA MONITOR</h1>
            <p class="text-gray-600 mt-1 font-medium">Sistem Status Meja Real-Time</p>
        </header>

        <div class="flex justify-center space-x-4 mb-6 sticky top-0 bg-gray-50 pt-4 z-10 shadow-sm border-b pb-4">
            <button id="mode-admin1" class="px-5 py-2 rounded-xl font-bold shadow-lg transition duration-300 transform no-select bg-red-600 text-white hover:bg-red-700">
                Admin 1 (Petugas Meja)
            </button>
            <button id="mode-admin2" class="px-5 py-2 rounded-xl font-bold shadow-lg transition duration-300 transform no-select bg-gray-300 text-gray-800 hover:bg-blue-600 hover:text-white">
                Admin 2 (Penyambut Tamu)
            </button>
        </div>

        <div class="flex flex-col sm:flex-row justify-between items-center p-3 mb-6 rounded-lg shadow-md bg-white border border-gray-200">
            <p class="text-sm font-medium mb-1 sm:mb-0">Mode Aktif: <span id="current-mode" class="font-bold text-red-600">Admin 1</span></p>
            <p id="connection-status" class="text-sm font-bold text-yellow-600">
                STATUS: Menghubungkan...
            </p>
        </div>
        
        <div id="table-grid" class="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-4 xl:grid-cols-5 gap-5 md:gap-6">
            <div class="col-span-full text-center p-8 text-gray-500 bg-white rounded-lg shadow-inner">Memuat data meja...</div>
        </div>
    </div>

    <script>
        // =========================================================
        // 🚨 PENTING: GANTI IP DI BAWAH INI DENGAN IP LOKAL TABLET ANDA!
        // Contoh: '192.168.0.50'
        const SERVER_IP = '10.208.137.54'; 
        // =========================================================
        const WS_URL = `ws://${SERVER_IP}:8080`;

        const tableGrid = document.getElementById('table-grid');
        const modeAdmin1Btn = document.getElementById('mode-admin1');
        const modeAdmin2Btn = document.getElementById('mode-admin2');
        const currentModeDisplay = document.getElementById('current-mode');
        const connectionStatusDisplay = document.getElementById('connection-status');
        let currentMode = 'admin1'; 
        let tableStatus = {}; // Status diisi oleh data dari server

        // Inisialisasi WebSocket
        let ws;

        function connectWebSocket() {
            connectionStatusDisplay.textContent = 'STATUS: Menghubungkan...';
            ws = new WebSocket(WS_URL);

            ws.onopen = () => {
                console.log("Terhubung ke server WebSocket.");
                connectionStatusDisplay.textContent = 'STATUS: TERHUBUNG';
                connectionStatusDisplay.classList.remove('text-yellow-600', 'text-red-600');
                connectionStatusDisplay.classList.add('text-green-600');
            };

            ws.onmessage = (event) => {
                const message = JSON.parse(event.data);
                if (message.type === 'STATUS_UPDATE') {
                    tableStatus = message.data; // Perbarui status dari server
                    renderTables(); // Render ulang tampilan
                }
            };

            ws.onerror = (error) => {
                console.error("Kesalahan WebSocket:", error);
                connectionStatusDisplay.textContent = 'STATUS: GAGAL (Cek IP & Server)';
                connectionStatusDisplay.classList.remove('text-yellow-600', 'text-green-600');
                connectionStatusDisplay.classList.add('text-red-600');
            };

            ws.onclose = () => {
                console.log("Koneksi terputus. Mencoba menghubungkan kembali dalam 5 detik...");
                connectionStatusDisplay.textContent = 'STATUS: TERPUTUS';
                connectionStatusDisplay.classList.remove('text-yellow-600', 'text-green-600');
                connectionStatusDisplay.classList.add('text-red-600');
                // Otomatis coba sambungkan kembali
                setTimeout(connectWebSocket, 5000); 
            };
        }

        // Fungsi untuk meng-render (menampilkan) meja
        function renderTables() {
            tableGrid.innerHTML = ''; // Kosongkan tampilan lama
            const tableIds = Object.keys(tableStatus).sort(); // Urutkan ID meja

            if (tableIds.length === 0 && ws.readyState === WebSocket.OPEN) {
                 tableGrid.innerHTML = `<div class="col-span-full text-center p-8 text-blue-500 bg-white rounded-lg shadow-inner">Server aktif, tetapi belum ada data meja.</div>`;
                 return;
            } else if (tableIds.length === 0) {
                 tableGrid.innerHTML = `<div class="col-span-full text-center p-8 text-gray-500 bg-white rounded-lg shadow-inner">Menghubungkan ke server...</div>`;
                 return;
            }
            
            tableIds.forEach(tableId => {
                const status = tableStatus[tableId];
                
                let bgColor, statusText, clickableClass;

                if (status === 'kosong') {
                    bgColor = 'bg-green-500 hover:bg-green-600';
                    statusText = 'KOSONG';
                    clickableClass = 'cursor-pointer active:scale-95';
                } else {
                    bgColor = 'bg-red-500 hover:bg-red-600';
                    statusText = 'TERISI';
                    clickableClass = 'cursor-pointer active:scale-95';
                }

                // Jika di mode Admin 2, tombol tidak interaktif
                if (currentMode === 'admin2') {
                    clickableClass = 'cursor-default';
                    // Hilangkan efek hover/active di Admin 2
                    bgColor = status === 'kosong' ? 'bg-green-500' : 'bg-red-500';
                }

                // Buat elemen HTML untuk meja
                const tableCard = document.createElement('div');
                tableCard.className = `meja-card p-6 rounded-xl shadow-xl text-white font-bold text-center ${bgColor} ${clickableClass} no-select`;
                tableCard.innerHTML = `
                    <p class="text-5xl mb-2">${tableId}</p>
                    <p class="text-xl tracking-wider">${statusText}</p>
                `;
                
                // Tambahkan event listener hanya di mode Admin 1 dan saat WS terhubung
                if (currentMode === 'admin1' && ws.readyState === WebSocket.OPEN) {
                    tableCard.addEventListener('click', () => {
                        // Kirim permintaan perubahan status ke server
                        ws.send(JSON.stringify({
                            type: 'TOGGLE_TABLE',
                            tableId: tableId
                        }));
                    });
                }
                
                tableGrid.appendChild(tableCard);
            });
        }

        // Fungsi untuk mengubah Mode Aplikasi
        function setMode(mode) {
            currentMode = mode;
            currentModeDisplay.textContent = mode === 'admin1' ? 'Admin 1 (INPUT)' : 'Admin 2 (VIEW)';
            
            // Perbarui tampilan tombol
            if (mode === 'admin1') {
                modeAdmin1Btn.classList.remove('bg-gray-300', 'text-gray-800', 'hover:bg-blue-600', 'hover:text-white');
                modeAdmin1Btn.classList.add('bg-red-600', 'text-white', 'hover:bg-red-700');
                
                modeAdmin2Btn.classList.add('bg-gray-300', 'text-gray-800', 'hover:bg-blue-600', 'hover:text-white');
                modeAdmin2Btn.classList.remove('bg-blue-600', 'text-white', 'hover:bg-blue-700');
            } else {
                modeAdmin2Btn.classList.remove('bg-gray-300', 'text-gray-800', 'hover:bg-blue-600', 'hover:text-white');
                modeAdmin2Btn.classList.add('bg-blue-600', 'text-white', 'hover:bg-blue-700');
                
                modeAdmin1Btn.classList.add('bg-red-600', 'text-white', 'hover:bg-red-700');
                modeAdmin1Btn.classList.remove('bg-red-700', 'ring-2', 'ring-red-300'); // Hapus ring jika ada
            }
            
            renderTables();
        }

        // Event Listeners untuk tombol Mode
        modeAdmin1Btn.addEventListener('click', () => setMode('admin1'));
        modeAdmin2Btn.addEventListener('click', () => setMode('admin2'));

        // Inisialisasi: Mulai koneksi WebSocket dan atur mode awal
        connectWebSocket();
        setMode('admin1');
    </script>

</body>
</html>
