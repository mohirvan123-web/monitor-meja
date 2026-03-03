<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Monitor Meja Gacoan</title>
    <script src="https://cdn.tailwindcss.com"></script>
    
    <script type="module">
        // Import Firebase SDK dari CDN
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
        import { getDatabase, ref, set, onValue } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-database.js";

        // DATA FIREBASE ANDA
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
        const db = getDatabase(app);

        const tableGrid = document.getElementById('table-grid');
        let currentMode = 'admin1';

        // Fungsi untuk mengubah status meja di Firebase
        window.toggleMeja = (id, currentStatus) => {
            if (currentMode !== 'admin1') {
                alert("Mode Lihat saja. Silakan pindah ke Admin 1 untuk mengubah.");
                return;
            }
            const newStatus = currentStatus === 'kosong' ? 'terisi' : 'kosong';
            set(ref(db, 'meja/' + id), {
                status: newStatus
            });
        };

        // Mendengarkan perubahan data dari Firebase secara Real-time
        onValue(ref(db, 'meja'), (snapshot) => {
            const data = snapshot.val() || {};
            renderLayout(data);
        });

        function renderLayout(data) {
            tableGrid.innerHTML = '';
            // Loop untuk 47 meja
            for (let i = 1; i <= 47; i++) {
                const id = "M" + String(i).padStart(2, '0');
                const status = (data[id]) ? data[id].status : 'kosong';
                
                const bgColor = status === 'kosong' ? 'bg-green-500 hover:bg-green-600' : 'bg-red-500 hover:bg-red-600';
                const label = status === 'kosong' ? 'KOSONG' : 'TERISI';
                
                const card = `
                    <div onclick="toggleMeja('${id}', '${status}')" 
                         class="${bgColor} p-3 rounded-lg shadow-md text-white text-center cursor-pointer transition transform active:scale-95 no-select">
                        <div class="text-xl font-bold">${id}</div>
                        <div class="text-[10px] font-medium opacity-90">${label}</div>
                    </div>
                `;
                tableGrid.innerHTML += card;
            }
        }

        // Fungsi Switch Mode
        window.setMode = (mode) => {
            currentMode = mode;
            const label = document.getElementById('admin-label');
            const indicator = document.getElementById('mode-indicator');
            
            if(mode === 'admin1') {
                label.innerText = 'MODE: ADMIN 1 (KLIK UNTUK ISI/KOSONG)';
                indicator.className = 'p-3 bg-red-100 text-red-700 font-bold rounded-lg border border-red-200 text-center mb-6';
            } else {
                label.innerText = 'MODE: ADMIN 2 (LIHAT SAJA)';
                indicator.className = 'p-3 bg-blue-100 text-blue-700 font-bold rounded-lg border border-blue-200 text-center mb-6
