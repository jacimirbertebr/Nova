<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DropMaster OS Ultra - Gestão & Venda</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    
    <!-- Firebase SDKs -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getFirestore, doc, setDoc, getDoc, collection, onSnapshot, addDoc, deleteDoc, updateDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";

        // Configuração Firebase (Provida pelo ambiente)
        const firebaseConfig = JSON.parse(__firebase_config);
        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'dropmaster-ultra-001';

        // Estado Global Reativo
        window.state = {
            user: null,
            products: [],
            settings: {
                columns: 'grid-cols-4',
                rounded: 'rounded-3xl',
                themeColor: 'blue',
                shadow: 'shadow-sm'
            },
            cart: []
        };

        // Autenticação Inicial
        const initAuth = async () => {
            try {
                await signInAnonymously(auth);
            } catch (e) { console.error("Erro Auth:", e); }
        };

        onAuthStateChanged(auth, (user) => {
            if (user) {
                window.state.user = user;
                syncData();
            }
        });

        // Sincronização em Tempo Real com Firestore
        const syncData = () => {
            // Escutar Produtos
            const prodRef = collection(db, 'artifacts', appId, 'public', 'data', 'products');
            onSnapshot(prodRef, (snapshot) => {
                window.state.products = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                renderStore();
                renderInventory();
            }, (err) => console.error(err));

            // Escutar Definições de Visualização
            const settingsRef = doc(db, 'artifacts', appId, 'public', 'data', 'settings');
            onSnapshot(settingsRef, (docSnap) => {
                if (docSnap.exists()) {
                    window.state.settings = docSnap.data();
                    applyViewSettings();
                }
            });
        };

        // Funções de Escrita
        window.saveProduct = async (product) => {
            const colRef = collection(db, 'artifacts', appId, 'public', 'data', 'products');
            await addDoc(colRef, product);
        };

        window.removeProduct = async (id) => {
            const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'products', id);
            await deleteDoc(docRef);
        };

        window.updateSettings = async (newSettings) => {
            const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'settings');
            await setDoc(docRef, { ...window.state.settings, ...newSettings });
        };

        initAuth();
    </script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;600;700&display=swap');
        body { font-family: 'Plus Jakarta Sans', sans-serif; transition: all 0.3s ease; }
        .tab-content { display: none; }
        .tab-content.active { display: block; animation: fadeIn 0.3s ease; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(5px); } to { opacity: 1; transform: translateY(0); } }
        .sidebar-item.active { background: #f1f5f9; border-right: 4px solid currentColor; }
    </style>
</head>
<body class="bg-slate-50 text-slate-900">

    <!-- View Switcher -->
    <div class="fixed bottom-6 right-6 z-[100] flex gap-2">
        <button onclick="changeContext('store')" class="bg-white border border-slate-200 px-4 py-3 rounded-2xl shadow-xl hover:bg-slate-50 font-bold flex items-center gap-2">
            <i data-lucide="shopping-bag" size="18"></i> Loja
        </button>
        <button onclick="changeContext('admin')" class="bg-slate-900 text-white px-4 py-3 rounded-2xl shadow-xl hover:scale-105 transition font-bold flex items-center gap-2">
            <i data-lucide="layout-dashboard" size="18"></i> Admin
        </button>
    </div>

    <!-- NOTIFICAÇÃO -->
    <div id="toast" class="fixed top-6 right-6 z-[200] transform translate-x-full transition-transform duration-300 bg-white border border-slate-200 p-4 rounded-2xl shadow-2xl flex items-center gap-3">
        <div id="toast-icon" class="p-2 rounded-lg bg-green-100 text-green-600"></div>
        <p id="toast-msg" class="text-sm font-bold"></p>
    </div>

    <!-- ================= LOJA ================= -->
    <div id="view-store" class="min-h-screen">
        <header class="bg-white/80 backdrop-blur-md sticky top-0 z-50 border-b border-slate-100">
            <div class="container mx-auto px-6 py-4 flex justify-between items-center">
                <div class="flex items-center gap-2">
                    <div id="brand-color" class="w-8 h-8 rounded-lg flex items-center justify-center text-white bg-blue-600">
                        <i data-lucide="zap" size="18"></i>
                    </div>
                    <span class="text-xl font-bold">DropMaster<span class="text-slate-400">UI</span></span>
                </div>
                <div class="flex gap-4">
                    <button onclick="openCart()" class="relative p-2 bg-slate-100 rounded-xl hover:bg-slate-200 transition">
                        <i data-lucide="shopping-cart" size="20"></i>
                        <span id="cart-badge" class="absolute -top-1 -right-1 bg-red-500 text-white text-[10px] w-4 h-4 rounded-full flex items-center justify-center font-bold">0</span>
                    </button>
                </div>
            </div>
        </header>

        <section class="container mx-auto px-6 py-12">
            <div id="store-grid" class="grid gap-8">
                <!-- Produtos renderizados dinamicamente -->
            </div>
        </section>
    </div>

    <!-- ================= ADMIN ================= -->
    <div id="view-admin" class="hidden min-h-screen flex">
        <aside class="w-64 border-r border-slate-200 flex flex-col fixed h-full bg-white">
            <div class="p-8 border-b">
                <h2 class="font-black text-lg tracking-tighter">DROPMASTER <span class="text-blue-600">OS</span></h2>
            </div>
            <nav class="flex-1 p-4 space-y-1">
                <button onclick="switchTab('dash')" id="btn-dash" class="sidebar-item active w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-bold transition-all">
                    <i data-lucide="pie-chart" size="18"></i> Métricas
                </button>
                <button onclick="switchTab('products')" id="btn-products" class="sidebar-item w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-bold text-slate-500">
                    <i data-lucide="box" size="18"></i> Inventário
                </button>
                <button onclick="switchTab('design')" id="btn-design" class="sidebar-item w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-bold text-slate-500">
                    <i data-lucide="palette" size="18"></i> Aparência
                </button>
                <button onclick="switchTab('hub')" id="btn-hub" class="sidebar-item w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-bold text-slate-500">
                    <i data-lucide="share-2" size="18"></i> Integrações
                </button>
            </nav>
        </aside>

        <main class="flex-1 ml-64 p-10 bg-slate-50">
            <!-- DASHBOARD -->
            <div id="tab-dash" class="tab-content active space-y-8">
                <h3 class="text-2xl font-bold">Bem-vindo, Administrador</h3>
                <div class="grid grid-cols-3 gap-6">
                    <div class="bg-white p-6 rounded-3xl border border-slate-200">
                        <p class="text-slate-400 text-[10px] font-bold uppercase mb-1">Vendas Totais</p>
                        <h4 class="text-3xl font-black">R$ 124.900</h4>
                    </div>
                    <div class="bg-white p-6 rounded-3xl border border-slate-200">
                        <p class="text-slate-400 text-[10px] font-bold uppercase mb-1">Visitantes</p>
                        <h4 class="text-3xl font-black">12.402</h4>
                    </div>
                    <div class="bg-white p-6 rounded-3xl border border-slate-200">
                        <p class="text-slate-400 text-[10px] font-bold uppercase mb-1">Conversão</p>
                        <h4 class="text-3xl font-black">3.2%</h4>
                    </div>
                </div>
                <div class="bg-white p-8 rounded-3xl border border-slate-200">
                    <canvas id="chart-admin" height="100"></canvas>
                </div>
            </div>

            <!-- INVENTÁRIO -->
            <div id="tab-products" class="tab-content space-y-6">
                <div class="flex justify-between items-center">
                    <h3 class="text-2xl font-bold">Gestão de Stock</h3>
                    <button onclick="openProductModal()" class="bg-blue-600 text-white px-6 py-2 rounded-xl font-bold text-sm">Novo Produto</button>
                </div>
                <div class="bg-white rounded-3xl border border-slate-200 overflow-hidden">
                    <table class="w-full text-left">
                        <thead class="bg-slate-50 text-[10px] font-bold text-slate-400 uppercase">
                            <tr>
                                <th class="px-6 py-4">Item</th>
                                <th class="px-6 py-4">Preço</th>
                                <th class="px-6 py-4">Ação</th>
                            </tr>
                        </thead>
                        <tbody id="inventory-list" class="divide-y divide-slate-100"></tbody>
                    </table>
                </div>
            </div>

            <!-- DESIGN CUSTOMIZER -->
            <div id="tab-design" class="tab-content space-y-8">
                <h3 class="text-2xl font-bold">Personalizar Aparência</h3>
                <div class="grid grid-cols-2 gap-8">
                    <div class="bg-white p-6 rounded-3xl border border-slate-200 space-y-6">
                        <div>
                            <label class="text-sm font-bold block mb-3">Colunas da Grelha</label>
                            <div class="flex gap-2">
                                <button onclick="window.updateSettings({columns: 'grid-cols-2'})" class="p-3 bg-slate-100 rounded-xl hover:bg-slate-200"><i data-lucide="layout-grid" size="18"></i></button>
                                <button onclick="window.updateSettings({columns: 'grid-cols-3'})" class="p-3 bg-slate-100 rounded-xl hover:bg-slate-200"><i data-lucide="grid-3x3" size="18"></i></button>
                                <button onclick="window.updateSettings({columns: 'grid-cols-4'})" class="p-3 bg-slate-100 rounded-xl hover:bg-slate-200"><i data-lucide="grid" size="18"></i></button>
                            </div>
                        </div>
                        <div>
                            <label class="text-sm font-bold block mb-3">Arredondamento</label>
                            <select onchange="window.updateSettings({rounded: this.value})" class="w-full bg-slate-100 p-3 rounded-xl border-none outline-none text-sm">
                                <option value="rounded-none">Quadrado</option>
                                <option value="rounded-xl">Suave</option>
                                <option value="rounded-3xl" selected>Muito Arredondado</option>
                            </select>
                        </div>
                        <div>
                            <label class="text-sm font-bold block mb-3">Cor de Destaque</label>
                            <div class="flex gap-4">
                                <div onclick="window.updateSettings({themeColor: 'blue'})" class="w-8 h-8 rounded-full bg-blue-600 cursor-pointer border-2 border-white shadow-lg"></div>
                                <div onclick="window.updateSettings({themeColor: 'emerald'})" class="w-8 h-8 rounded-full bg-emerald-600 cursor-pointer border-2 border-white shadow-lg"></div>
                                <div onclick="window.updateSettings({themeColor: 'violet'})" class="w-8 h-8 rounded-full bg-violet-600 cursor-pointer border-2 border-white shadow-lg"></div>
                                <div onclick="window.updateSettings({themeColor: 'rose'})" class="w-8 h-8 rounded-full bg-rose-600 cursor-pointer border-2 border-white shadow-lg"></div>
                            </div>
                        </div>
                    </div>
                    <div class="bg-slate-200 rounded-3xl flex items-center justify-center p-10 relative overflow-hidden">
                        <div class="absolute inset-0 bg-white/30 backdrop-blur-sm flex items-center justify-center font-bold text-slate-400">PRÉ-VISUALIZAÇÃO</div>
                        <div id="preview-card" class="bg-white p-4 w-48 shadow-xl">
                            <div class="h-20 bg-slate-100 rounded-lg mb-2"></div>
                            <div class="h-4 w-full bg-slate-100 mb-1"></div>
                            <div class="h-3 w-1/2 bg-slate-100"></div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- HUB DE INTEGRAÇÕES -->
            <div id="tab-hub" class="tab-content space-y-6">
                <h3 class="text-2xl font-bold">Conectar Marketplaces</h3>
                <div class="grid grid-cols-2 gap-4">
                    <div class="bg-white p-6 rounded-3xl border border-slate-200 flex justify-between items-center">
                        <div class="flex items-center gap-4">
                            <div class="w-12 h-12 bg-yellow-400 rounded-xl flex items-center justify-center text-white font-bold">ML</div>
                            <div>
                                <p class="font-bold">Mercado Livre</p>
                                <p class="text-[10px] text-slate-400 uppercase">Sincronização Ativa</p>
                            </div>
                        </div>
                        <input type="checkbox" checked class="w-5 h-5 accent-blue-600">
                    </div>
                    <div class="bg-white p-6 rounded-3xl border border-slate-200 flex justify-between items-center opacity-50">
                        <div class="flex items-center gap-4">
                            <div class="w-12 h-12 bg-orange-500 rounded-xl flex items-center justify-center text-white font-bold">SH</div>
                            <div>
                                <p class="font-bold">Shopee</p>
                                <p class="text-[10px] text-slate-400 uppercase">Desconectado</p>
                            </div>
                        </div>
                        <input type="checkbox" class="w-5 h-5 accent-blue-600">
                    </div>
                </div>
            </div>
        </main>
    </div>

    <!-- MODAL NOVO PRODUTO -->
    <div id="modal-product" class="hidden fixed inset-0 bg-slate-900/40 backdrop-blur-sm z-[150] flex items-center justify-center p-4">
        <div class="bg-white w-full max-w-md rounded-3xl p-8 shadow-2xl">
            <h4 class="text-xl font-bold mb-6">Novo Produto</h4>
            <div class="space-y-4">
                <input type="text" id="new-p-name" placeholder="Nome do Produto" class="w-full bg-slate-100 p-4 rounded-xl border-none outline-none">
                <input type="number" id="new-p-price" placeholder="Preço (R$)" class="w-full bg-slate-100 p-4 rounded-xl border-none outline-none">
                <select id="new-p-icon" class="w-full bg-slate-100 p-4 rounded-xl border-none outline-none">
                    <option value="watch">Relógio/Smartwatch</option>
                    <option value="headphones">Áudio/Fone</option>
                    <option value="camera">Câmara/Foto</option>
                    <option value="smartphone">Telemóvel</option>
                    <option value="cpu">Hardware</option>
                </select>
                <button onclick="handleSaveProduct()" class="w-full bg-blue-600 text-white py-4 rounded-xl font-bold">Confirmar Cadastro</button>
                <button onclick="document.getElementById('modal-product').classList.add('hidden')" class="w-full text-slate-400 font-bold text-sm">Cancelar</button>
            </div>
        </div>
    </div>

    <!-- CARRINHO / CHECKOUT -->
    <div id="cart-drawer" class="fixed inset-y-0 right-0 w-full max-w-md bg-white shadow-2xl z-[160] transform translate-x-full transition-transform duration-300 flex flex-col">
        <div class="p-8 border-b flex justify-between items-center">
            <h4 class="text-xl font-bold">O seu Carrinho</h4>
            <button onclick="closeCart()"><i data-lucide="x"></i></button>
        </div>
        <div id="cart-items" class="flex-1 overflow-y-auto p-8 space-y-4"></div>
        <div class="p-8 border-t space-y-4">
            <div class="flex justify-between font-bold">
                <span>Total</span>
                <span id="cart-total">R$ 0,00</span>
            </div>
            <button onclick="checkout()" class="w-full bg-slate-900 text-white py-4 rounded-2xl font-bold">Finalizar Compra</button>
        </div>
    </div>

    <script>
        // Lógica de Interface (Não-Firebase)
        
        function changeContext(ctx) {
            document.getElementById('view-store').classList.toggle('hidden', ctx === 'admin');
            document.getElementById('view-admin').classList.toggle('hidden', ctx === 'store');
            lucide.createIcons();
        }

        function switchTab(tab) {
            document.querySelectorAll('.tab-content').forEach(t => t.classList.remove('active'));
            document.querySelectorAll('.sidebar-item').forEach(b => b.classList.remove('active', 'text-blue-600'));
            document.getElementById(`tab-${tab}`).classList.add('active');
            document.getElementById(`btn-${tab}`).classList.add('active');
            document.getElementById(`btn-${tab}`).classList.add('text-blue-600');
            lucide.createIcons();
        }

        function showToast(msg, type = 'success') {
            const t = document.getElementById('toast');
            document.getElementById('toast-msg').innerText = msg;
            t.classList.remove('translate-x-full');
            setTimeout(() => t.classList.add('translate-x-full'), 3000);
        }

        // Renderização Loja
        function renderStore() {
            const grid = document.getElementById('store-grid');
            const s = window.state.settings;
            grid.className = `grid ${s.columns} gap-8`;
            
            grid.innerHTML = window.state.products.map(p => `
                <div class="bg-white p-6 border border-slate-100 hover:shadow-2xl transition group ${s.rounded} ${s.shadow}">
                    <div class="h-48 bg-slate-50 rounded-2xl mb-4 flex items-center justify-center text-slate-300">
                        <i data-lucide="${p.icon}" size="48"></i>
                    </div>
                    <h4 class="font-bold text-slate-800 mb-4">${p.name}</h4>
                    <div class="flex justify-between items-center">
                        <span class="text-xl font-black">R$ ${parseFloat(p.price).toFixed(2)}</span>
                        <button onclick="addToCart('${p.id}')" class="bg-theme text-white p-3 rounded-xl shadow-lg transition-transform hover:scale-110">
                            <i data-lucide="plus" size="18"></i>
                        </button>
                    </div>
                </div>
            `).join('');
            lucide.createIcons();
            updateThemeStyles();
        }

        function renderInventory() {
            const list = document.getElementById('inventory-list');
            list.innerHTML = window.state.products.map(p => `
                <tr class="hover:bg-slate-50">
                    <td class="px-6 py-4 font-bold text-sm text-slate-700">${p.name}</td>
                    <td class="px-6 py-4 font-medium text-slate-500 text-sm">R$ ${p.price}</td>
                    <td class="px-6 py-4">
                        <button onclick="window.removeProduct('${p.id}')" class="text-red-400 hover:text-red-600"><i data-lucide="trash-2" size="16"></i></button>
                    </td>
                </tr>
            `).join('');
            lucide.createIcons();
        }

        function applyViewSettings() {
            const s = window.state.settings;
            const preview = document.getElementById('preview-card');
            preview.className = `bg-white p-4 w-48 shadow-xl ${s.rounded}`;
            renderStore();
        }

        function updateThemeStyles() {
            const colorMap = { blue: '#2563eb', emerald: '#059669', violet: '#7c3aed', rose: '#e11d48' };
            const hex = colorMap[window.state.settings.themeColor];
            document.querySelectorAll('.bg-theme').forEach(el => el.style.backgroundColor = hex);
            document.getElementById('brand-color').style.backgroundColor = hex;
        }

        // Funções de Checkout
        function openCart() { document.getElementById('cart-drawer').classList.remove('translate-x-full'); }
        function closeCart() { document.getElementById('cart-drawer').classList.add('translate-x-full'); }

        function addToCart(id) {
            const p = window.state.products.find(x => x.id === id);
            window.state.cart.push(p);
            updateCartUI();
            showToast("Adicionado ao carrinho!");
        }

        function updateCartUI() {
            const container = document.getElementById('cart-items');
            document.getElementById('cart-badge').innerText = window.state.cart.length;
            let total = 0;
            container.innerHTML = window.state.cart.map((item, idx) => {
                total += parseFloat(item.price);
                return `
                    <div class="flex justify-between items-center bg-slate-50 p-3 rounded-xl">
                        <span class="text-xs font-bold">${item.name}</span>
                        <span class="text-xs font-black">R$ ${item.price}</span>
                    </div>
                `;
            }).join('');
            document.getElementById('cart-total').innerText = `R$ ${total.toFixed(2)}`;
        }

        function checkout() {
            if (window.state.cart.length === 0) return;
            showToast("A processar pagamento...");
            setTimeout(() => {
                window.state.cart = [];
                updateCartUI();
                closeCart();
                showToast("Compra finalizada com sucesso!");
            }, 2000);
        }

        // Modais e Salvar
        function openProductModal() { document.getElementById('modal-product').classList.remove('hidden'); }
        async function handleSaveProduct() {
            const name = document.getElementById('new-p-name').value;
            const price = document.getElementById('new-p-price').value;
            const icon = document.getElementById('new-p-icon').value;
            if(!name || !price) return;

            await window.saveProduct({ name, price, icon, date: new Date().toISOString() });
            document.getElementById('modal-product').classList.add('hidden');
            showToast("Produto guardado no banco de dados!");
        }

        // Gráfico Admin
        const ctx = document.getElementById('chart-admin').getContext('2d');
        new Chart(ctx, {
            type: 'bar',
            data: {
                labels: ['Jan', 'Fev', 'Mar', 'Abr', 'Mai'],
                datasets: [{ label: 'Receita Bruta', data: [12000, 19000, 15000, 22000, 28000], backgroundColor: '#2563eb', borderRadius: 10 }]
            },
            options: { responsive: true, scales: { y: { display: false }, x: { grid: { display: false } } }, plugins: { legend: { display: false } } }
        });

        lucide.createIcons();
    </script>
</body>
</html>
