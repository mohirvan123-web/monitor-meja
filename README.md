<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Denah Meja Gacoan</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/interactjs/dist/interact.min.js"></script>
    
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
        import { getDatabase, ref, set, onValue, update } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-database.js";

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

        const floorPlan = document.getElementById('floor-plan');
        let currentMode = 'admin1';
        let isLayoutLocked = true;

        // --- LOGIKA DRAG AND DROP ---
        interact('.meja-item').draggable({
            listeners: {
                move(event) {
                    if (isLayoutLocked) return;
                    
                    const target = event.target;
                    // Ambil posisi lama atau 0
                    const x = (parseFloat(target.getAttribute('data-x')) || 0) + event.dx;
                    const y = (parseFloat(target.getAttribute('data-y')) || 0) + event.dy;

                    // Update tampilan sementara (biar lancar/smooth)
                    target.style.transform = `translate(${x}px, ${y}px)`;
                    target.setAttribute('data-x', x);
                    target.setAttribute('data-y', y);
                },
                end(event) {
                    if (isLayoutLocked) return;
                    const target = event.target;
                    const id = target.id;
                    const x = parseFloat(target.getAttribute('data-x'));
                    const y = parseFloat(target.getAttribute('data-y'));

                    // SIMPAN POSISI KE FIREBASE
                    update(ref(db, `meja/${id}`), { x, y });
                }
            }
        });

        // Toggle Status Meja
        window.toggleMeja = (id, currentStatus) => {
            if (!isLayoutLocked) return; // Jangan toggle kalau lagi geser layout
            if (currentMode !== 'admin1') return;

            const newStatus = currentStatus === 'kosong' ? 'terisi' : 'kosong';
            update(ref(db, `meja/${id}`), { status: newStatus });
        };

        // Reset Meja Massal
        window.setSemuaMeja = (statusTarget) => {
            if (confirm(`Set semua meja jadi ${statusTarget}?`)) {
                onValue(ref(db, 'meja'), (snapshot) => {
                    const data = snapshot.val();
                    const updates = {};
                    for (let id in data) {
                        updates[`meja/${id}/status`] = statusTarget;
                    }
                    update(ref(db), updates);
                }, { onlyOnce: true });
            }
        };

        // Render Real-time
        onValue(ref(db, 'meja'), (snapshot) => {
            const data = snapshot.val() || {};
            renderFloorPlan(data);
        });

        function renderFloorPlan(data) {
            // Jika data kosong, buat 82 meja pertama kali
            if (Object.keys(data).length === 0) {
                for(let i=1; i<=82; i++) {
                    const id = "M" + String(i).padStart(2, '0');
                    set(ref(db, `meja/${id}`), { status: 'kosong', x: (i%10)*60, y: Math.floor(i/10)*60 });
                }
                return;
            }

            floorPlan.innerHTML = '';
            for (let id in data) {
                const item = data[id];
                const color = item.status === 'kosong' ? 'bg-green-500' : 'bg-red-500';
                
                const mejaEl = document.createElement('div');
                mejaEl.id = id;
                mejaEl.className = `meja-item absolute ${color} w-12 h-12 rounded shadow-lg text-white flex flex-col items-center justify-center cursor-pointer transition-colors duration-300 select-none`;
                mejaEl.style.transform = `translate(${item.x || 0}px, ${item.y || 0}px)`;
                mejaEl.setAttribute('data-x', item.x || 0);
                mejaEl.setAttribute('data-y', item.y || 0);
                
                mejaEl.innerHTML = `
                    <span class="text-[10px] font-bold">${id}</span>
                `;
                
                mejaEl.onclick = () => window.toggleMeja(id, item.status);
                floorPlan.appendChild(mejaEl);
            }
        }

        // Kontrol Mode & Lock
        window.setMode = (mode) => {
            currentMode = mode;
            document.getElementById('admin-label').innerText = mode === 'admin1' ? 'ADMIN 1' : 'ADMIN 2';
            document.getElementById('edit-controls').style.display = mode === 'admin1' ? 'flex' : 'none';
        };

        window.toggleLock = () => {
            isLayoutLocked = !isLayoutLocked;
            const btn = document.getElementById('lock-btn');
            btn.innerText = isLayoutLocked ? '🔓 Buka Kunci Layout' : '🔒 Kunci Layout (Simpan)';
            btn.className = isLayoutLocked ? 'bg-yellow-500 text-white px-4 py-2 rounded font-bold' : 'bg-green-600 text-white px-4 py-2 rounded font-bold';
            
            const items = document.querySelectorAll('.meja-item');
            items.forEach(el => {
                el.style.border = isLayoutLocked ? 'none' : '2px dashed white';
            });
        };
    </script>
    <style>
        #floor-plan {
            width: 1000px; /* Luas denah, bisa disesuaikan */
            height: 1000px;
            background-image: radial-gradient(#cbd5e1 1px, transparent 1px);
            background-size: 20px 20px; /* Titik-titik bantu untuk merapikan meja */
        }
        .meja-item { touch-action: none; user-select: none; }
    </style>
</head>
<body class="bg-slate-100 overflow-hidden flex flex-col h-screen">

    <div class="bg-white p-4 shadow-md z-10">
        <div class="flex justify-between items-center max-w-5xl mx-auto">
            <h1 class="font-black text-red-600">GACOAN MAP <span id="admin-label" class="text-xs bg-red-100 px-2 py-1 rounded ml-2">ADMIN 1</span></h1>
            <div class="flex gap-2">
                <button onclick="setMode('admin1')" class="bg-red-600 text-white text-xs px-3 py-1 rounded font-bold">Admin 1</button>
                <button onclick="setMode('admin2')" class="bg-blue-600 text-white text-xs px-3 py-1 rounded font-bold">Admin 2</button>
            </div>
        </div>
    </div>

    <div id="edit-controls" class="p-3 bg-white border-b flex flex-wrap gap-2 justify-center shadow-inner">
        <button id="lock-btn" onclick="toggleLock()" class="bg-yellow-500 text-white px-4 py-2 rounded text-xs font-bold">🔓 Buka Kunci Layout</button>
        <button onclick="setSemuaMeja('kosong')" class="bg-emerald-600 text-white px-4 py-2 rounded text-xs font-bold italic">Reset Kosong</button>
    </div>

    <div class="flex-grow overflow-auto p-10 relative">
        <div id="floor-plan" class="relative bg-white shadow-2xl rounded-xl border border-slate-200">
            </div>
    </div>

    <div class="p-2 bg-white text-[10px] text-center text-slate-400">
        *Admin 1: Buka kunci untuk geser posisi. Klik meja untuk ubah status saat terkunci.
    </div>

</body>
</html>
