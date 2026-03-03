<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gacoan Meja - Firebase Version</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
        import { getDatabase, ref, set, onValue } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-database.js";

        // 🚨 GANTI DENGAN CONFIG FIREBASE ANDA SENDIRI
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

// Initialize Firebase
const app = initializeApp(firebaseConfig);
const analytics = getAnalytics(app);


        const app = initializeApp(firebaseConfig);
        const db = getDatabase(app);

        // Inisialisasi 47 Meja secara otomatis jika belum ada
        const tableGrid = document.getElementById('table-grid');
        let currentMode = 'admin1';

        // Fungsi Update Status ke Firebase
        window.toggleMeja = (id, currentStatus) => {
            if (currentMode !== 'admin1') return;
            const newStatus = currentStatus === 'kosong' ? 'terisi' : 'kosong';
            set(ref(db, 'meja/' + id), { status: newStatus });
        };

        // Mendengarkan perubahan data secara Real-time
        onValue(ref(db, 'meja'), (snapshot) => {
            const data = snapshot.val();
            renderLayout(data);
        });

        function renderLayout(data) {
            tableGrid.innerHTML = '';
            for (let i = 1; i <= 47; i++) {
                const id = "M" + String(i).padStart(2, '0');
                const status = (data && data[id]) ? data[id].status : 'kosong';
                
                const color = status === 'kosong' ? 'bg-green-500' : 'bg-red-500';
                const label = status === 'kosong' ? 'KOSONG' : 'TERISI';
                
                const card = `
                    <div onclick="toggleMeja('${id}', '${status}')" 
                         class="${color} p-4 rounded-lg shadow text-white text-center cursor-pointer transition active:scale-95">
                        <div class="text-2xl font-bold">${id}</div>
                        <div class="text-xs">${label}</div>
                    </div>
                `;
                tableGrid.innerHTML += card;
            }
        }

        // Switch Mode Admin
        window.setMode = (mode) => {
            currentMode = mode;
            document.getElementById('admin-label').innerText = mode === 'admin1' ? 'MODE: ADMIN 1 (INPUT)' : 'MODE: ADMIN 2 (LIHAT)';
            document.getElementById('mode-indicator').className = mode === 'admin1' ? 'p-2 bg-red-100 text-red-700 font-bold rounded' : 'p-2 bg-blue-100 text-blue-700 font-bold rounded';
        };
    </script>
</head>
<body class="bg-gray-100 p-4">
    <div class="max-w-4xl mx-auto">
        <h1 class="text-3xl font-bold text-center text-red-600 mb-4">GACOAN MONITOR</h1>
        
        <div class="flex gap-2 justify-center mb-6">
            <button onclick="setMode('admin1')" class="bg-red-600 text-white px-4 py-2 rounded shadow">Admin 1</button>
            <button onclick="setMode('admin2')" class="bg-blue-600 text-white px-4 py-2 rounded shadow">Admin 2</button>
        </div>

        <div id="mode-indicator" class="p-2 bg-red-100 text-red-700 font-bold rounded text-center mb-6">
            <span id="admin-label">MODE: ADMIN 1 (INPUT)</span>
        </div>

        <div id="table-grid" class="grid grid-cols-3 sm:grid-cols-5 md:grid-cols-8 gap-3">
            </div>
    </div>
</body>
</html>
