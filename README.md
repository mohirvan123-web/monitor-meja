<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Monitor Meja Gacoan</title>
    <script src="https://cdn.tailwindcss.com"></script>
    
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
        import { getDatabase, ref, set, onValue } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-database.js";

        const firebaseConfig = {
          apiKey: "AIzaSyDBQ4Z74pXUHa9sihwG7e_AJLLGTTORcXI",
          authDomain: "gacoanmeja.firebaseapp.com",
          databaseURL: "https://gacoanmeja-default-rtdb.firebaseio.com",
          projectId: "gacoanmeja",
          storageBucket: "gacoanmeja.firebasestorage.app",
          messagingSenderId: "829661149102",
          appId: "1:829661149102:web:937077e9516a7720827a3f",
          measurementId: "G-4Z0QCZCF95"
        };

        const app = initializeApp(firebaseConfig);
        const db = getDatabase(app);

        const tableGrid = document.getElementById('table-grid');
        let currentMode = 'admin1';

        window.toggleMeja = (id, currentStatus) => {
            if (currentMode !== 'admin1') {
                alert("Mode Lihat saja. Silakan pindah ke Admin 1 untuk mengubah status meja.");
                return;
            }
            const newStatus = currentStatus === 'kosong' ? 'terisi' : 'kosong';
            set(ref(db, 'meja/' + id), { status: newStatus });
        };

        onValue(ref(db, 'meja'), (snapshot) => {
            const data = snapshot.val() || {};
            renderLayout(data);
        });

        function renderLayout(data) {
            tableGrid.innerHTML = '';
            for (let i = 1; i <= 47; i++) {
                const id = "M" + String(i).padStart(2, '0');
                const status = (data[id]) ? data[id].status : 'kosong';
                
                const bgColor = status === 'kosong' ? 'bg-green-500' : 'bg-red-500';
                const label = status === 'kosong' ? 'KOSONG' : 'TERISI';
                
                const card = `
                    <div onclick="toggleMeja('${id}', '${status}')" 
                         class="${bgColor} p-2 rounded-lg shadow text-white text-center cursor-pointer transition transform active:scale-90 select-none">
                        <div class="text-lg font-bold">${id}</div>
                        <div class="text-[9px] uppercase">${label}</div>
                    </div>
                `;
                tableGrid.innerHTML += card;
            }
        }

        window.setMode = (mode) => {
            currentMode = mode;
            const label = document.getElementById('admin-label');
            const indicator = document.getElementById('mode-indicator');
            
            if(mode === 'admin1') {
                label.innerText = 'MODE: ADMIN 1 (INPUT)';
                indicator.className = 'p-3 bg-red-100 text-red-700 font-bold rounded-lg border border-red-200 text-center mb-6';
            } else {
                label.innerText = 'MODE: ADMIN 2 (LIHAT)';
                indicator.className = 'p-3 bg-blue-100 text-blue-700 font-bold rounded-lg border border-blue-200 text-center mb-6';
            }
        };
    </script>
</head>
<body class="bg-gray-50 p-4">
    <div class="max-w-4xl mx-auto">
        <h1 class="text-2xl font-black text-red-600 text-center mb-6">GACOAN MEJA MONITOR</h1>
        
        <div class="flex gap-2 justify-center mb-4">
            <button onclick="setMode('admin1')" class="bg-red-600 text-white px-4 py-2 rounded-lg font-bold">Admin 1</button>
            <button onclick="setMode('admin2')" class="bg-blue-600 text-white px-4 py-2 rounded-lg font-bold">Admin 2</button>
        </div>

        <div id="mode-indicator" class="p-3 bg-red-100 text-red-700 font-bold rounded-lg border border-red-200 text-center mb-6">
            <span id="admin-label">MODE: ADMIN 1 (INPUT)</span>
        </div>

        <div id="table-grid" class="grid grid-cols-4 sm:grid-cols-6 md:grid-cols-10 gap-2"></div>
    </div>
</body>
</html>
