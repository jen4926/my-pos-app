<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple POS System</title>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100 font-sans">

    <div class="flex h-screen overflow-hidden">
        <!-- Sidebar -->
        <div class="w-64 bg-blue-900 text-white flex flex-col">
            <div class="p-5 text-2xl font-bold tracking-wider border-b border-blue-800">RMVillas POS</div>
            <nav class="flex-1 p-4 space-y-2">
                <button onclick="switchTab('pos')" id="btn-pos" class="w-full text-left px-4 py-2 rounded bg-blue-700 font-medium">🛒 POS Entry</button>
                <button onclick="switchTab('inventory')" id="btn-inventory" class="w-full text-left px-4 py-2 rounded hover:bg-blue-800 font-medium">📦 Inventory</button>
                <button onclick="switchTab('reports')" id="btn-reports" class="w-full text-left px-4 py-2 rounded hover:bg-blue-800 font-medium">📊 Daily Reports</button>
            </nav>
        </div>

        <!-- Main Content -->
        <div class="flex-1 flex flex-col overflow-y-auto">
            <header class="bg-white shadow px-6 py-4 flex justify-between items-center">
                <h1 id="page-title" class="text-xl font-semibold text-gray-800">POS Entry</h1>
                <span class="text-sm text-gray-500">Admin Mode</span>
            </header>

            <main class="p-6">
                <!-- POS TAB -->
                <div id="tab-pos" class="grid grid-cols-1 md:grid-cols-3 gap-6">
                    <div class="md:col-span-2 bg-white p-4 rounded shadow">
                        <h2 class="text-lg font-bold mb-4">Mga Produkto</h2>
                        <div id="product-grid" class="grid grid-cols-2 sm:grid-cols-3 gap-4">
                            <!-- Dynamic Products -->
                        </div>
                    </div>
                    <div class="bg-white p-4 rounded shadow flex flex-col justify-between">
                        <div>
                            <h2 class="text-lg font-bold mb-4">Current Order</h2>
                            <div id="cart-list" class="divide-y max-h-64 overflow-y-auto mb-4">
                                <p class="text-gray-400 text-sm">Walang laman ang cart.</p>
                            </div>
                        </div>
                        <div class="border-t pt-4">
                            <div class="flex justify-between text-lg font-bold mb-4">
                                <span>Total:</span>
                                <span id="cart-total">₱0.00</span>
                            </div>
                            <button onclick="checkout()" class="w-full bg-green-600 text-white py-2 rounded font-bold hover:bg-green-700">Bayaran / Checkout</button>
                        </div>
                    </div>
                </div>

                <!-- INVENTORY TAB -->
                <div id="tab-inventory" class="hidden bg-white p-6 rounded shadow">
                    <h2 class="text-lg font-bold mb-4">Magdagdag ng Produkto</h2>
                    <div class="flex gap-4 mb-6">
                        <input type="text" id="new-name" placeholder="Pangalan ng Produkto" class="border p-2 rounded flex-1">
                        <input type="number" id="new-price" placeholder="Presyo (₱)" class="border p-2 rounded w-32">
                        <input type="number" id="new-stock" placeholder="Stock" class="border p-2 rounded w-24">
                        <button onclick="addProduct()" class="bg-blue-600 text-white px-4 py-2 rounded">Idagdag</button>
                    </div>
                    <h2 class="text-lg font-bold mb-2">Listahan ng Imbentaryo</h2>
                    <table class="w-full text-left border-collapse">
                        <thead>
                            <tr class="border-b bg-gray-50">
                                <th class="p-2">Produkto</th>
                                <th class="p-2">Presyo</th>
                                <th class="p-2">Stock</th>
                                <th class="p-2">Aksyon</th>
                            </tr>
                        </thead>
                        <tbody id="inventory-table">
                            <!-- Dynamic Inventory -->
                        </tbody>
                    </table>
                </div>

                <!-- REPORTS TAB -->
                <div id="tab-reports" class="hidden bg-white p-6 rounded shadow">
                    <h2 class="text-lg font-bold mb-4">Ulat ng mga Benta</h2>
                    <div class="flex justify-between items-center mb-4">
                        <span class="text-gray-600 font-medium">Kabuuang Benta Ngayon:</span>
                        <span id="total-sales" class="text-xl font-bold text-green-600">₱0.00</span>
                    </div>
                    <div id="sales-list" class="divide-y">
                        <p class="text-gray-400 text-sm">Wala pang transaksyon ngayong araw.</p>
                    </div>
                </div>
            </main>
        </div>
    </div>

    <script>
        let products = JSON.parse(localStorage.getItem('pos_products')) || [
            { id: 1, name: 'Kape (Hot)', price: 45, stock: 50 },
            { id: 2, name: 'Bottled Water', price: 20, stock: 100 },
            { id: 3, name: 'Tinapay', price: 15, stock: 30 }
        ];
        let cart = [];
        let sales = JSON.parse(localStorage.getItem('pos_sales')) || [];

        function switchTab(tab) {
            document.getElementById('tab-pos').classList.add('hidden');
            document.getElementById('tab-inventory').classList.add('hidden');
            document.getElementById('tab-reports').classList.add('hidden');
            
            document.getElementById('btn-pos').className = "w-full text-left px-4 py-2 rounded hover:bg-blue-800 font-medium";
            document.getElementById('btn-inventory').className = "w-full text-left px-4 py-2 rounded hover:bg-blue-800 font-medium";
            document.getElementById('btn-reports').className = "w-full text-left px-4 py-2 rounded hover:bg-blue-800 font-medium";

            document.getElementById('tab-' + tab).classList.remove('hidden');
            document.getElementById('btn-' + tab).className = "w-full text-left px-4 py-2 rounded bg-blue-700 font-medium";

            if(tab === 'pos') renderPOS();
            if(tab === 'inventory') renderInventory();
            if(tab === 'reports') renderReports();
        }

        function renderPOS() {
            const grid = document.getElementById('product-grid');
            grid.innerHTML = '';
            products.forEach(p => {
                grid.innerHTML += `
                    <div onclick="addToCart(${p.id})" class="border p-4 rounded cursor-pointer hover:bg-gray-50 flex flex-col justify-between">
                        <div>
                            <h3 class="font-bold text-gray-800">${p.name}</h3>
                            <p class="text-sm text-gray-500">Stock: ${p.stock}</p>
                        </div>
                        <span class="text-blue-600 font-semibold mt-4">₱${p.price.toFixed(2)}</span>
                    </div>
                `;
            });
        }

        function addToCart(id) {
            let product = products.find(p => p.id === id);
            if (product.stock <= 0) {
                alert('Ubos na ang stock nito!');
                return;
            }
            let item = cart.find(i => i.id === id);
            if (item) {
                item.qty++;
            } else {
                cart.push({ id: product.id, name: product.name, price: product.price, qty: 1 });
            }
            product.stock--;
            renderCart();
            renderPOS();
        }

        function renderCart() {
            const list = document.getElementById('cart-list');
            let total = 0;
            if (cart.length === 0) {
                list.innerHTML = `<p class="text-gray-400 text-sm">Walang laman ang cart.</p>`;
                document.getElementById('cart-total').innerText = '₱0.00';
                return;
            }
            list.innerHTML = '';
            cart.forEach(item => {
                total += item.price * item.qty;
                list.innerHTML += `
                    <div class="py-2 flex justify-between items-center text-sm">
                        <div>
                            <p class="font-medium">${item.name}</p>
                            <p class="text-gray-500">₱${item.price} x ${item.qty}</p>
                        </div>
                        <span class="font-bold">₱${(item.price * item.qty).toFixed(2)}</span>
                    </div>
                `;
            });
            document.getElementById('cart-total').innerText = `₱${total.toFixed(2)}`;
        }

        function checkout() {
            if (cart.length === 0) return alert('Walang laman ang cart!');
            let total = cart.reduce((sum, item) => sum + (item.price * item.qty), 0);
            sales.push({ date: new Date().toLocaleString(), items: [...cart], total });
            localStorage.setItem('pos_sales', JSON.stringify(sales));
            localStorage.setItem('pos_products', JSON.stringify(products));
            cart = [];
            renderCart();
            renderPOS();
            alert('Matagumpay na na-checkout ang order!');
        }

        function renderInventory() {
            const table = document.getElementById('inventory-table');
            table.innerHTML = '';
            products.forEach(p => {
                table.innerHTML += `
                    <tr class="border-b">
                        <td class="p-2">${p.name}</td>
                        <td class="p-2">₱${p.price.toFixed(2)}</td>
                        <td class="p-2">${p.stock}</td>
                        <td class="p-2"><button onclick="deleteProduct(${p.id})" class="text-red-600 text-sm font-medium">Burahin</button></td>
                    </tr>
                `;
            });
        }

        function addProduct() {
            let name = document.getElementById('new-name').value;
            let price = parseFloat(document.getElementById('new-price').value);
            let stock = parseInt(document.getElementById('new-stock').value);

            if (!name || isNaN(price) || isNaN(stock)) {
                alert('Paki-kumpleto ang mga impormasyon.');
                return;
            }

            products.push({ id: Date.now(), name, price, stock });
            localStorage.setItem('pos_products', JSON.stringify(products));
            document.getElementById('new-name').value = '';
            document.getElementById('new-price').value = '';
            document.getElementById('new-stock').value = '';
            renderInventory();
            renderPOS();
        }

        function deleteProduct(id) {
            products = products.filter(p => p.id !== id);
            localStorage.setItem('pos_products', JSON.stringify(products));
            renderInventory();
            renderPOS();
        }

        function renderReports() {
            const list = document.getElementById('sales-list');
            const totalSalesEl = document.getElementById('total-sales');
            let grandTotal = 0;

            if (sales.length === 0) {
                list.innerHTML = `<p class="text-gray-400 text-sm">Wala pang transaksyon na naitala.</p>`;
                totalSalesEl.innerText = '₱0.00';
                return;
            }

            list.innerHTML = '';
            sales.forEach(s => {
                grandTotal += s.total;
                list.innerHTML += `
                    <div class="py-3 flex justify-between items-center text-sm border-b">
                        <div>
                            <p class="text-gray-500 text-xs">${s.date}</p>
                            <p class="font-medium">${s.items.map(i => `${i.qty}x${i.name}`).join(', ')}</p>
                        </div>
                        <span class="font-bold text-green-600">₱${s.total.toFixed(2)}</span>
                    </div>
                `;
            });
            totalSalesEl.innerText = `₱${grandTotal.toFixed(2)}`;
        }

        // Initialize first view
        renderPOS();
    </script>
</body>
</html>
