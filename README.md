<!DOCTYPE html>
<html lang="id" style="scroll-behavior: smooth;">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Reservasi Online Gacoan</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .form-input {
            width: 100%;
            border: 1px solid #d1d5db;
            border-radius: 0.5rem;
            padding: 0.75rem;
            transition-property: border-color;
            transition-duration: 300ms;
        }

        .form-input:focus {
            outline: none;
            border-color: #3b82f6;
        }
        
        @keyframes popIn {
          0% { transform: scale(0); opacity: 0; }
          80% { transform: scale(1.1); opacity: 1; }
          100% { transform: scale(1); }
        }

        .qty-btn {
          animation: popIn 0.2s ease-out;
        }
        
        /* General Modal Styles */
        .modal {
            display: none;
            position: fixed;
            z-index: 100;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            overflow: auto;
            background-color: rgba(0,0,0,0.6);
            backdrop-filter: blur(5px);
            -webkit-backdrop-filter: blur(5px);
        }

        .modal-content {
            background-color: #fefefe;
            margin: 10vh auto;
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.2);
            width: 90%;
            max-width: 500px;
            position: relative;
            transform: translateY(-50px);
            opacity: 0;
            animation: fadeInTop 0.5s forwards;
        }

        @keyframes fadeInTop {
            to {
                transform: translateY(0);
                opacity: 1;
            }
        }
        
        .close-btn {
            color: #aaa;
            float: right;
            font-size: 28px;
            font-weight: bold;
            transition: color 0.2s;
        }

        .close-btn:hover,
        .close-btn:focus {
            color: #000;
            text-decoration: none;
            cursor: pointer;
        }
        
        .sticky-buttons {
            position: sticky;
            bottom: 0;
            z-index: 50;
            padding: 1rem;
            background-color: white;
            box-shadow: 0 -2px 10px rgba(0,0,0,0.1);
            display: flex;
            justify-content: center;
        }

        /* Responsive grid for menu */
        .menu-grid {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 1.5rem;
        }
        
        @media (min-width: 768px) {
            .menu-grid {
                grid-template-columns: repeat(3, 1fr);
            }
            .modal-content {
                max-width: 600px; /* Lebarkan modal seat */
            }
        }
        
        /* Seat Selection Styles */
        .seat-grid {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 10px;
            padding: 1rem;
            border: 1px dashed #ccc;
            border-radius: 8px;
            background-color: #f7f7f7;
        }
        
        .seat-item {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            width: 70px;
            height: 70px;
            border-radius: 8px;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.2s ease;
            text-align: center;
            line-height: 1.2;
            padding: 5px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        
        .seat-item span.capacity {
            font-size: 0.7rem;
            font-weight: normal;
        }
        
        .seat-item.available {
            background-color: #4CAF50; /* Green */
            color: white;
            border: 2px solid #388E3C;
        }
        
        .seat-item.reserved {
            background-color: #e57373; /* Light Red */
            color: white;
            cursor: not-allowed;
            opacity: 0.6;
            border: 2px solid #D32F2F;
        }

        .seat-item.reserved:hover {
            transform: none;
        }
        
        .seat-item.selected {
            background-color: #2196F3; /* Blue */
            color: white;
            border: 3px solid #1976D2;
            transform: scale(1.05);
        }

        .seat-item.available:hover {
            background-color: #388E3C;
            transform: scale(1.05);
        }

        .screen-line {
            width: 100%;
            text-align: center;
            margin: 20px 0;
            padding: 5px;
            background-color: #a0a0a0;
            color: white;
            font-weight: bold;
            border-radius: 4px;
        }
    </style>
</head>
<body class="bg-gray-100 min-h-screen flex flex-col font-sans">
    
    <header class="bg-blue-600 rounded-xl text-white p-1 shadow-lg sticky top-0 z-50">
        <div class="container mx-auto flex justify-between items-center">
            <h1 class="text-xl font-bold tracking-wide">Gacoan Jakarta</h1>
        </div>
    </header>

    <nav class="bg-pink-300 sticky top-10 z-40 shadow-sm md:shadow-md p-2">
        <div class="container mx-auto">
            <ul class="flex justify-around items-center space-x-4 overflow-x-auto whitespace-nowrap md:justify-center">
                <li><a href="#noodle-section" class="text-gray-800 font-bold px-4 py-2 rounded-full hover:bg-pink-200 transition-colors duration-200">🍜 Noodle</a></li>
                <li><a href="#dimsum-section" class="text-gray-800 font-bold px-4 py-2 rounded-full hover:bg-pink-200 transition-colors duration-200">🥟 Dimsum</a></li>
                <li><a href="#beverage-section" class="text-gray-800 font-bold px-4 py-2 rounded-full hover:bg-pink-200 transition-colors duration-200">🥤 Beverage</a></li>
            </ul>
        </div>
    </nav>

    <main class="flex-1 container mx-auto p-4 flex flex-col gap-8">
        <section class="flex-1">
            <h2 class="text-3xl font-bold mb-3 text-gray-800 border-b-2 border-pink-500 pb-2">FORMULIR RESERVASI ONLINE</h2>

            <div class="bg-white rounded-xl shadow-md p-6 mb-8 border-t-4 border-blue-600">
                <form id="reservation-form" class="space-y-4">
                    <div>
                        <label for="customerName" class="block text-sm font-semibold mb-2">Nama Pelanggan:</label>
                        <input type="text" id="customerName" placeholder="Masukkan nama Anda" class="form-input focus:border-blue-500 transition-colors" required />
                    </div>
                    <div>
                        <label for="customerPhone" class="block text-sm font-semibold mb-2">Nomor HP:</label>
                        <input type="tel" id="customerPhone" placeholder="Contoh: 081234567890" class="form-input focus:border-blue-500 transition-colors" required />
                    </div>
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <div>
                            <label for="reservationDate" class="block text-sm font-semibold mb-2">Tanggal Reservasi:</label>
                            <input type="date" id="reservationDate" class="form-input focus:border-blue-500 transition-colors" required />
                        </div>
                        <div>
                            <label for="reservationTime" class="block text-sm font-semibold mb-2">Jam Reservasi:</label>
                            <input type="time" id="reservationTime" class="form-input focus:border-blue-500 transition-colors" required />
                        </div>
                    </div>
                    <div>
                        <label for="numberOfPeople" class="block text-sm font-semibold mb-2">Jumlah Orang (Kapasitas Meja):</label>
                        <input type="number" id="numberOfPeople" placeholder="Min 1, Max 10" min="1" max="10" class="form-input focus:border-blue-500 transition-colors" required />
                    </div>
                    <div>
                        <button type="button" id="selectSeatBtn" class="w-full bg-pink-500 text-white rounded-lg py-2 font-semibold hover:bg-pink-600 transition-colors duration-300 transform active:scale-95">
                            PILIH MEJA DINE IN
                        </button>
                    </div>
                    <p id="selectedSeatDisplay" class="text-sm font-semibold text-gray-600">Meja yang dipilih: -</p>
                    <div>
                        <label class="block text-sm font-semibold mb-2">Metode Pembayaran:</label>
                        <select id="paymentMethod" class="form-input focus:border-blue-500 transition-colors" required>
                            <option value="Langsung di Kasir">Langsung di Kasir</option>
                            <option value="Transfer ke Rekening 13579">Transfer ke Rekening 13579</option>
                        </select>
                    </div>
                    <input type="hidden" id="orderType" value="Dine In" /> </form>
            </div>

            <h3 id="noodle-section" class="text-2xl font-bold mb-4 mt-8 text-gray-700 border-b-2 border-pink-500 pb-0">🍜 Noodle</h3>
            <div id="noodle-list" class="menu-grid">
                </div>

            <h3 id="dimsum-section" class="text-2xl font-bold mb-4 mt-8 text-gray-700 border-b-2 border-pink-500 pb-0">🥟 Dimsum</h3>
            <div id="dimsum-list" class="menu-grid">
                </div>

            <h3 id="beverage-section" class="text-2xl font-bold mb-4 mt-8 text-gray-700 border-b-2 border-pink-500 pb-0">🥤 Beverage</h3>
            <div id="beverage-list" class="menu-grid">
                </div>

        </section>
    </main>
    
    <div class="sticky-buttons">
        <div class="container mx-auto flex justify-center">
            <button id="viewOrderList" class="w-full max-w-sm bg-blue-600 text-white rounded-lg py-1 px-6 font-semibold hover:bg-blue-700 transition-colors duration-300 shadow-md">Lihat Pesanan & Submit</button>
        </div>
    </div>

    <div id="orderModal" class="modal">
        <div class="modal-content">
            <span class="close-btn order-close-btn">&times;</span>
            <h3 class="text-xl font-bold mb-4 text-gray-800">Daftar Reservasi Anda</h3>
            <div id="modalReservationInfo" class="mb-4 p-3 bg-blue-50 rounded-lg border-l-4 border-blue-500">
                </div>
            <div id="modalOrderList" class="space-y-4 max-h-60 overflow-y-auto">
                </div>
            <div class="mt-6 pt-4 border-t border-gray-300 flex justify-between items-center">
                <span id="modalTotal" class="text-lg font-bold text-gray-800">Total: Rp 0</span>
                <button id="modalSubmitBtn" class="bg-blue-600 text-white rounded-lg py-2 px-4 font-semibold hover:bg-blue-700 transition-colors duration-300 transform active:scale-95">Submit Reservasi via WhatsApp</button>
            </div>
        </div>
    </div>
    
    <div id="seatSelectionModal" class="modal">
        <div class="modal-content !max-w-xl">
            <span class="close-btn seat-close-btn">&times;</span>
            <h3 class="text-xl font-bold mb-4 text-gray-800">Pilih Meja Anda</h3>
            <p class="text-sm mb-4 text-center">Pilih **1** meja yang sesuai dengan jumlah **<span id="seatCapacityRequired" class="font-bold text-pink-600">0</span>** orang.</p>
            
            <div class="screen-line">AREA KASIR/DAPUR</div>

            <div id="seatLayout" class="seat-grid">
                </div>
            
            <div class="flex justify-center gap-4 mt-6 text-sm">
                <div class="flex items-center gap-1">
                    <div class="w-4 h-4 rounded-full bg-blue-500"></div><span>Terpilih</span>
                </div>
                <div class="flex items-center gap-1">
                    <div class="w-4 h-4 rounded-full bg-green-500"></div><span>Tersedia</span>
                </div>
                <div class="flex items-center gap-1">
                    <div class="w-4 h-4 rounded-full bg-red-400"></div><span>Terisi/Reserved</span>
                </div>
            </div>

            <div class="mt-6 pt-4 border-t border-gray-300 flex justify-end">
                <button id="confirmSeatBtn" class="bg-pink-500 text-white rounded-lg py-2 px-4 font-semibold hover:bg-pink-600 transition-colors duration-300 transform active:scale-95 disabled:opacity-50" disabled>Konfirmasi Meja</button>
            </div>
        </div>
    </div>

    <footer class="bg-gray-200 text-center p-4 text-sm text-gray-600 mt-8">
        © 2025 Reservasi Online Gacoan
    </footer>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const menuData = [
                { name: "Mie Suit", category: "NOODLE", price: 11000, type: "pink" },
                { name: "Mie Gacoan Lv.0", category: "NOODLE", price: 11000, type: "yellow" },
                { name: "Mie gacoan Lv. 1", category: "NOODLE", price: 11000, type: "yellow" },
                { name: "Mie Gacoan Lv. 2", category: "NOODLE", price: 11000, type: "yellow" },
                { name: "Mie Gacoan Lv. 3", category: "NOODLE", price: 11000, type: "yellow" },
                { name: "Mie Gacoan Lv. 4", category: "NOODLE", price: 11000, type: "yellow" },
                { name: "Mie Gacoan Lv. 6", category: "NOODLE", price: 12000, type: "yellow" },
                { name: "Mie Gacoan Lv. 8", category: "NOODLE", price: 12000, type: "yellow" },
                { name: "Mie Hompimpa Lv.1", category: "NOODLE", price: 11000, type: "blue" },
                { name: "Mie Hompimpa Lv.2", category: "NOODLE", price: 11000, type: "blue" },
                { name: "Mie Hompimpa Lv. 3", category: "NOODLE", price: 11000, type: "blue" },
                { name: "Mie Hompimpa Lv. 4", category: "NOODLE", price: 11000, type: "blue" },
                { name: "Mie Hompimpa Lv. 6", category: "NOODLE", price: 12000, type: "blue" },
                { name: "Mie Hompimpa Lv. 8", category: "NOODLE", price: 12000, type: "blue" },

                { name: "Udang Keju", category: "DIMSUM", price: 10000, type: "pink" },
                { name: "Udang Rambutan", category: "DIMSUM", price: 10000, type: "pink" },
                { name: "Lumpia Udang", category: "DIMSUM", price: 10000, type: "pink" },
                { name: "Siomay Ayam", category: "DIMSUM", price: 10000, type: "pink" },
                { name: "Pangsit Goreng", category: "DIMSUM", price: 11000, type: "pink" },

                { name: "Es Gobak Sodor", category: "BEVERAGE", price: 10000, type: "blue" },
                { name: "Es Petak Umpet", category: "BEVERAGE", price: 10000, type: "blue" },
                { name: "Es Teklek", category: "BEVERAGE", price: 10000, type: "blue" },
                { name: "Es Sluku Bathok", category: "BEVERAGE", price: 10000, type: "blue" },
                { name: "Green Thai Tea", category: "BEVERAGE", price: 10000, type: "blue" },
                { name: "Thai Tea", category: "BEVERAGE", price: 10000, type: "blue" },
                { name: "Lemon Tea", category: "BEVERAGE", price: 10000, type: "blue" },
                { name: "Tea", category: "BEVERAGE", price: 10000, type: "blue" },
                { name: "Coklat", category: "BEVERAGE", price: 10000, type: "blue" },
                { name: "Vanilla Late", category: "BEVERAGE", price: 10000, type: "blue" },
                { name: "Teh Tarik", category: "BEVERAGE", price: 10000, type: "blue" },
                { name: "Air Mineral", category: "BEVERAGE", price: 5000, type: "blue" },
            
                { name: "Tea Hot", category: "BEVERAGE", price: 5000, type: "blue" },
            ];

            // Data Meja (Simulasi Layout Bioskop)
            const seatLayoutData = [
                { id: "T01", capacity: 2, status: "available" },
                { id: "T02", capacity: 2, status: "available" },
                { id: "T03", capacity: 4, status: "available" },
                { id: "T04", capacity: 4, status: "reserved" }, // Contoh meja terisi
                { id: "T05", capacity: 6, status: "available" },
                { id: "T06", capacity: 8, status: "available" },
                { id: "T07", capacity: 2, status: "available" },
                { id: "T08", capacity: 4, status: "available" },
                { id: "T09", capacity: 4, status: "reserved" },
                { id: "T10", capacity: 2, status: "available" },
                { id: "T11", capacity: 4, status: "available" },
                { id: "T12", capacity: 6, status: "available" },
            ];
            
            let selectedSeatId = null; // Meja yang dipilih pengguna
            
            // DOM Elements
            const noodleListEl = document.getElementById('noodle-list');
            const dimsumListEl = document.getElementById('dimsum-list');
            const beverageListEl = document.getElementById('beverage-list');
            const viewOrderBtn = document.getElementById('viewOrderList');
            
            const orderModal = document.getElementById('orderModal');
            const modalOrderList = document.getElementById('modalOrderList');
            const modalReservationInfo = document.getElementById('modalReservationInfo');
            const modalTotal = document.getElementById('modalTotal');
            const modalSubmitBtn = document.getElementById('modalSubmitBtn');
            const orderCloseBtn = document.querySelector('.order-close-btn');

            const seatSelectionModal = document.getElementById('seatSelectionModal');
            const selectSeatBtn = document.getElementById('selectSeatBtn');
            const confirmSeatBtn = document.getElementById('confirmSeatBtn');
            const seatCloseBtn = document.querySelector('.seat-close-btn');
            const seatLayoutEl = document.getElementById('seatLayout');
            const selectedSeatDisplay = document.getElementById('selectedSeatDisplay');
            const numberOfPeopleInput = document.getElementById('numberOfPeople');
            const seatCapacityRequiredEl = document.getElementById('seatCapacityRequired');


            const selectedMenus = {}; // Menu yang dipilih

            // --- Fungsi Menu ---
            function renderMenu() {
                menuData.forEach(item => {
                    const card = document.createElement('div');
                    card.classList.add('bg-white', 'rounded-2xl', 'shadow-lg', 'p-4', 'flex', 'flex-col', 'transform', 'transition-transform', 'duration-300', 'hover:scale-[1.02]');
                    
                    const categoryColor = item.type === "pink" ? "text-pink-600" : (item.type === "yellow" ? "text-yellow-500" : "text-blue-400");
                    
                    card.innerHTML = `
                        <h4 class="font-bold text-lg text-gray-900 mt-2">${item.name}</h4>
                        <p class="${categoryColor} text-sm font-semibold">${item.category}</p>
                        <p class="font-bold text-lg mt-1 text-pink-600">Rp ${item.price.toLocaleString("id-ID")}</p>
                        <div class="flex items-center gap-2 mt-auto pt-2">
                            <button class="minus-qty bg-gray-200 px-2 rounded transform transition-transform duration-150 active:scale-95">-</button>
                            <span class="qty-display px-2 w-8 text-center">0</span>
                            <button class="plus-qty bg-gray-200 px-2 rounded transform transition-transform duration-150 active:scale-95">+</button>
                        </div>
                    `;

                    const minusBtn = card.querySelector('.minus-qty');
                    const plusBtn = card.querySelector('.plus-qty');
                    const qtyDisplay = card.querySelector('.qty-display');

                    plusBtn.addEventListener('click', () => {
                        let qty = (selectedMenus[item.name] || 0) + 1;
                        selectedMenus[item.name] = qty;
                        qtyDisplay.innerText = qty;
                    });

                    minusBtn.addEventListener('click', () => {
                        let qty = (selectedMenus[item.name] || 0) - 1;
                        if (qty < 0) qty = 0;
                        selectedMenus[item.name] = qty;
                        qtyDisplay.innerText = qty;
                    });

                    if (item.category === "NOODLE") {
                        noodleListEl.appendChild(card);
                    } else if (item.category === "DIMSUM") {
                        dimsumListEl.appendChild(card);
                    } else if (item.category === "BEVERAGE") {
                        beverageListEl.appendChild(card);
                    }
                });
            }

            // --- Fungsi Seat Selection ---
            function renderSeatLayout() {
                seatLayoutEl.innerHTML = '';
                const numberOfPeople = parseInt(numberOfPeopleInput.value) || 1;
                seatCapacityRequiredEl.innerText = numberOfPeople;
                
                seatLayoutData.forEach(table => {
                    const isReserved = table.status === 'reserved';
                    const isSelected = selectedSeatId === table.id;
                    const canFit = table.capacity >= numberOfPeople;
                    const isDisabled = isReserved || !canFit;
                    
                    const seatItem = document.createElement('div');
                    seatItem.id = `seat-${table.id}`;
                    seatItem.classList.add('seat-item', isSelected ? 'selected' : (isReserved ? 'reserved' : 'available'));
                    seatItem.classList.toggle('opacity-50', isDisabled && !isReserved);

                    seatItem.innerHTML = `
                        Meja ${table.id.replace('T', '')}
                        <span class="capacity">(${table.capacity} Pax)</span>
                    `;
                    
                    if (!isDisabled) {
                        seatItem.addEventListener('click', () => {
                            if (selectedSeatId === table.id) {
                                // Deselect
                                selectedSeatId = null;
                                seatItem.classList.remove('selected');
                                seatItem.classList.add('available');
                            } else {
                                // Select new seat
                                const prevSelected = document.querySelector('.seat-item.selected');
                                if (prevSelected) {
                                    prevSelected.classList.remove('selected');
                                    prevSelected.classList.add('available');
                                }
                                selectedSeatId = table.id;
                                seatItem.classList.remove('available');
                                seatItem.classList.add('selected');
                            }
                            // Toggle konfirmasi button
                            confirmSeatBtn.disabled = !selectedSeatId;
                        });
                    }

                    seatLayoutEl.appendChild(seatItem);
                });
            }

            // --- Fungsi Validasi ---
            function validateForm(checkSeat = false) {
                const customerName = document.getElementById('customerName').value.trim();
                const customerPhone = document.getElementById('customerPhone').value.trim();
                const reservationDate = document.getElementById('reservationDate').value;
                const reservationTime = document.getElementById('reservationTime').value;
                const numberOfPeople = document.getElementById('numberOfPeople').value;

                if (!customerName || !customerPhone || !reservationDate || !reservationTime || !numberOfPeople) {
                    alert('Mohon lengkapi semua data reservasi (Nama, HP, Tanggal, Jam, Jumlah Orang).');
                    return false;
                }
                
                if (checkSeat && !selectedSeatId) {
                    alert('Mohon pilih meja sebelum melanjutkan.');
                    return false;
                }

                return true;
            }

            // --- Fungsi Pesan dan Submit ---
            function generateOrderMessage() {
                const customerName = document.getElementById('customerName').value.trim();
                const customerPhone = document.getElementById('customerPhone').value.trim();
                const reservationDate = document.getElementById('reservationDate').value;
                const reservationTime = document.getElementById('reservationTime').value;
                const paymentMethod = document.getElementById('paymentMethod').value;
                const numberOfPeople = document.getElementById('numberOfPeople').value;
                const selectedSeat = selectedSeatId ? selectedSeatId : 'BELUM DIPILIH';

                let totalHarga = 0;
                const selectedItems = [];

                for (const name in selectedMenus) {
                    if (selectedMenus[name] > 0) {
                        const itemData = menuData.find(m => m.name === name);
                        if (itemData) {
                            selectedItems.push({
                                name: name,
                                qty: selectedMenus[name],
                                price: itemData.price,
                            });
                            totalHarga += selectedMenus[name] * itemData.price;
                        }
                    }
                }
                
                let pesan = `🔔 *Reservasi DINE IN Baru* 🔔%0A%0A`;
                pesan += `👤 *Nama:* ${customerName}%0A`;
                pesan += `📞 *Nomor HP:* ${customerPhone}%0A`;
                pesan += `🗓️ *Tanggal:* ${reservationDate}%0A`;
                pesan += `⏰ *Jam:* ${reservationTime}%0A`;
                pesan += `👨‍👩‍👧‍👦 *Jumlah Orang:* ${numberOfPeople} orang%0A`;
                pesan += `🪑 *Nomor Meja:* ${selectedSeat}%0A`;
                pesan += `💳 *Metode Bayar:* ${paymentMethod}%0A%0A`;
                
                pesan += `🍜 *Daftar Menu Pesanan:*%0A`;
                if (selectedItems.length > 0) {
                    selectedItems.forEach(item => {
                        pesan += `- ${item.name}   ➡️   ${item.qty} pcs%0A`;
                    });
                } else {
                    pesan += `- _Tidak ada menu yang dipilih_%0A`;
                }

                pesan += `%0A────────────────────────%0A`;
                pesan += `💸 *Estimasi Total:* Rp ${totalHarga.toLocaleString("id-ID")}%0A`;
                pesan += `────────────────────────%0A%0A`;
                pesan += `*Mohon segera konfirmasi ketersediaan meja ${selectedSeat} dan reservasi ini.* 🙏`;
                
                return { pesan, selectedItems, totalHarga, reservationDate, reservationTime, customerName, numberOfPeople, selectedSeat };
            }

            function handleSubmission() {
                if (validateForm(true)) { // Validasi dengan cek meja
                    const { pesan } = generateOrderMessage();
                    const noWA = "6283187982993"; // Ganti dengan nomor WhatsApp penerima
                    const url = `https://api.whatsapp.com/send?phone=${noWA}&text=${pesan}`;
                    window.open(url, '_blank', 'noopener,noreferrer');
                }
            }

            // --- Event Listeners ---
            
            // 1. Tombol Buka Modal Pilih Meja
            selectSeatBtn.addEventListener('click', () => {
                if (!validateForm(false)) return;
                
                // Pastikan input jumlah orang valid sebelum buka modal
                const people = parseInt(numberOfPeopleInput.value);
                if (people < 1 || people > 10 || isNaN(people)) {
                    alert('Jumlah orang harus di antara 1 sampai 10.');
                    numberOfPeopleInput.focus();
                    return;
                }

                renderSeatLayout();
                seatSelectionModal.style.display = 'block';
                confirmSeatBtn.disabled = !selectedSeatId; // Update button state
            });
            
            // 2. Tombol Konfirmasi Meja
            confirmSeatBtn.addEventListener('click', () => {
                if (selectedSeatId) {
                    selectedSeatDisplay.innerHTML = `Meja yang dipilih: <span class="text-blue-600 font-bold">${selectedSeatId}</span> (Kapasitas ${seatLayoutData.find(t => t.id === selectedSeatId).capacity} Pax)`;
                    seatSelectionModal.style.display = 'none';
                }
            });

            // 3. Tombol Lihat Pesanan
            viewOrderBtn.addEventListener('click', () => {
                if (!validateForm(true)) return; // Validasi wajib meja terpilih
                
                const { selectedItems, totalHarga, reservationDate, reservationTime, customerName, numberOfPeople, selectedSeat } = generateOrderMessage();
                
                // Update info reservasi di modal
                modalReservationInfo.innerHTML = `
                    <p><strong>Nama:</strong> ${customerName}</p>
                    <p><strong>Tanggal/Jam:</strong> ${reservationDate} / ${reservationTime}</p>
                    <p><strong>Jumlah Orang:</strong> ${numberOfPeople}</p>
                    <p><strong>Meja:</strong> <span class="font-bold text-pink-600">${selectedSeat}</span></p>
                `;

                // Update daftar menu
                modalOrderList.innerHTML = '';
                if (selectedItems.length > 0) {
                    selectedItems.forEach(item => {
                        const itemEl = document.createElement('div');
                        itemEl.classList.add('flex', 'justify-between', 'items-center', 'py-2', 'border-b', 'border-gray-200');
                        itemEl.innerHTML = `
                            <span>${item.qty}x ${item.name}</span>
                            <span>Rp ${(item.qty * item.price).toLocaleString("id-ID")}</span>
                        `;
                        modalOrderList.appendChild(itemEl);
                    });
                } else {
                    modalOrderList.innerHTML = '<p class="text-gray-500 text-center">Belum ada menu yang dipilih.</p>';
                }
                modalTotal.innerText = `Total: Rp ${totalHarga.toLocaleString("id-ID")}`;
                orderModal.style.display = 'block';
            });
            
            // 4. Submit Reservasi
            modalSubmitBtn.addEventListener('click', handleSubmission);

            // 5. Close Modals
            orderCloseBtn.addEventListener('click', () => {
                orderModal.style.display = 'none';
            });
            seatCloseBtn.addEventListener('click', () => {
                seatSelectionModal.style.display = 'none';
            });

            window.addEventListener('click', (event) => {
                if (event.target === orderModal) {
                    orderModal.style.display = 'none';
                }
                if (event.target === seatSelectionModal) {
                    seatSelectionModal.style.display = 'none';
                }
            });

            // Initial calls
            renderMenu();
            // Inisialisasi tampilan meja
            if(selectedSeatId) {
                selectedSeatDisplay.innerHTML = `Meja yang dipilih: <span class="text-blue-600 font-bold">${selectedSeatId}</span>`;
            } else {
                 selectedSeatDisplay.innerHTML = `Meja yang dipilih: <span class="text-red-500 font-bold">BELUM DIPILIH</span>`;
            }
        });
    </script>

</body>
</html>
