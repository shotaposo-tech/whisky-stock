# whisky-stock
自宅のウイスキーの保存状況
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Whisky Stock</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js" type="module"></script>
    <script src="https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js" type="module"></script>
    <script src="https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js" type="module"></script>
    <style>
        /* iOSライクな慣性スクロールと外観の調整 */
        body {
            -webkit-tap-highlight-color: transparent;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            background-color: #f8fafc;
        }
        .safe-area-bottom {
            padding-bottom: env(safe-area-inset-bottom);
        }
        /* スライダーの見た目カスタマイズ */
        input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            appearance: none;
            width: 24px;
            height: 24px;
            background: #b45309;
            cursor: pointer;
            border-radius: 50%;
            border: 2px solid white;
            box-shadow: 0 2px 4px rgba(0,0,0,0.2);
        }
    </style>
</head>
<body class="text-slate-900 pb-24">
    <div id="app" class="max-w-md mx-auto min-h-screen relative">
        <!-- Header -->
        <header class="sticky top-0 z-30 bg-white/80 backdrop-blur-md border-b border-slate-200 px-5 py-4 flex justify-between items-center">
            <h1 class="text-xl font-bold text-amber-900 flex items-center gap-2">
                🥃 Whisky Stock
            </h1>
            <div id="stats" class="text-xs font-medium text-slate-500 bg-slate-100 px-3 py-1 rounded-full">
                読込中...
            </div>
        </header>

        <!-- Search -->
        <div class="px-4 mt-4">
            <div class="relative">
                <input type="text" id="searchInput" placeholder="銘柄名で検索..." 
                    class="w-full bg-white border border-slate-200 rounded-2xl py-3 pl-11 pr-4 outline-none focus:ring-2 focus:ring-amber-500 shadow-sm text-base">
                <span class="absolute left-4 top-1/2 -translate-y-1/2 opacity-40">🔍</span>
            </div>
        </div>

        <!-- List Content -->
        <div id="whiskyList" class="px-4 mt-6 space-y-3 pb-20">
            <!-- Items will be injected here -->
            <div class="text-center py-10 opacity-50">読み込み中...</div>
        </div>

        <!-- Floating Action Button -->
        <button id="addBtn" class="fixed bottom-8 right-6 w-14 h-14 bg-amber-800 text-white rounded-full shadow-2xl flex items-center justify-center text-3xl z-40 active:scale-95 transition-transform">
            ＋
        </button>

        <!-- Modal Overlay -->
        <div id="modal" class="fixed inset-0 bg-black/60 backdrop-blur-sm z-50 hidden flex items-end sm:items-center justify-center p-0 sm:p-4">
            <div class="bg-white w-full max-w-md rounded-t-[2.5rem] sm:rounded-[2rem] p-8 shadow-2xl animate-in slide-in-from-bottom duration-300">
                <div class="w-12 h-1.5 bg-slate-200 rounded-full mx-auto mb-6 sm:hidden"></div>
                
                <h2 id="modalTitle" class="text-2xl font-bold mb-6 text-slate-800">ボトルの追加</h2>
                
                <form id="whiskyForm" class="space-y-5">
                    <input type="hidden" id="editId">
                    <div>
                        <label class="block text-sm font-semibold text-slate-600 mb-1 ml-1">銘柄名</label>
                        <input type="text" id="name" required class="w-full p-4 bg-slate-50 border-none rounded-2xl focus:ring-2 focus:ring-amber-500 outline-none text-base" placeholder="例: 山崎 12年">
                    </div>

                    <div class="grid grid-cols-2 gap-4">
                        <div>
                            <label class="block text-sm font-semibold text-slate-600 mb-1 ml-1">タイプ</label>
                            <select id="type" class="w-full p-4 bg-slate-50 border-none rounded-2xl focus:ring-2 focus:ring-amber-500 outline-none text-base appearance-none">
                                <option value="Single Malt">シングルモルト</option>
                                <option value="Blended">ブレンデッド</option>
                                <option value="Bourbon">バーボン</option>
                                <option value="Japanese">ジャパニーズ</option>
                                <option value="Other">その他</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-semibold text-slate-600 mb-1 ml-1">残量: <span id="rangeVal">100</span>%</label>
                            <input type="range" id="remaining" min="0" max="100" step="5" value="100" class="w-full h-10 accent-amber-800">
                        </div>
                    </div>

                    <div>
                        <label class="block text-sm font-semibold text-slate-600 mb-1 ml-1">メモ</label>
                        <textarea id="note" class="w-full p-4 bg-slate-50 border-none rounded-2xl focus:ring-2 focus:ring-amber-500 outline-none text-base" rows="2" placeholder="購入日や味の感想など"></textarea>
                    </div>

                    <div class="flex gap-3 pt-2 safe-area-bottom">
                        <button type="button" id="cancelBtn" class="flex-1 py-4 bg-slate-100 text-slate-600 font-bold rounded-2xl active:opacity-70">キャンセル</button>
                        <button type="submit" class="flex-[2] py-4 bg-amber-800 text-white font-bold rounded-2xl shadow-lg active:opacity-90">保存する</button>
                    </div>
                </form>
            </div>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, doc, addDoc, updateDoc, deleteDoc, onSnapshot, query } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = JSON.parse(__firebase_config);
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'whisky-mobile-app';

        let currentUser = null;
        let allWhiskies = [];

        // Auth
        const initAuth = async () => {
            if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                await signInWithCustomToken(auth, __initial_auth_token);
            } else {
                await signInAnonymously(auth);
            }
        };
        initAuth();

        onAuthStateChanged(auth, (user) => {
            currentUser = user;
            if (user) loadData();
        });

        // Load Data
        function loadData() {
            const colRef = collection(db, 'artifacts', appId, 'users', currentUser.uid, 'whiskies');
            onSnapshot(colRef, (snapshot) => {
                allWhiskies = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                renderList(allWhiskies);
                updateStats();
            }, (err) => console.error(err));
        }

        // Render List
        function renderList(data) {
            const listEl = document.getElementById('whiskyList');
            const term = document.getElementById('searchInput').value.toLowerCase();
            const filtered = data.filter(w => w.name.toLowerCase().includes(term));

            if (filtered.length === 0) {
                listEl.innerHTML = `<div class="text-center py-20 text-slate-400">ボトルがありません</div>`;
                return;
            }

            listEl.innerHTML = filtered.map(w => `
                <div class="bg-white rounded-3xl p-5 shadow-sm border border-slate-100 flex items-center gap-4 active:scale-[0.98] transition-transform" onclick="editWhisky('${w.id}')">
                    <div class="flex-1">
                        <div class="flex items-center gap-2 mb-1">
                            <span class="text-[10px] font-bold px-2 py-0.5 bg-amber-100 text-amber-800 rounded-full uppercase">${w.type}</span>
                        </div>
                        <h3 class="font-bold text-lg leading-tight">${w.name}</h3>
                        ${w.note ? `<p class="text-xs text-slate-400 mt-1 line-clamp-1">${w.note}</p>` : ''}
                    </div>
                    <div class="text-right flex flex-col items-end gap-1">
                        <span class="text-xs font-bold ${w.remaining < 20 ? 'text-red-500' : 'text-amber-700'}">${w.remaining}%</span>
                        <div class="w-16 h-1.5 bg-slate-100 rounded-full overflow-hidden">
                            <div class="h-full ${w.remaining < 20 ? 'bg-red-500' : 'bg-amber-600'}" style="width: ${w.remaining}%"></div>
                        </div>
                    </div>
                </div>
            `).join('');
        }

        function updateStats() {
            document.getElementById('stats').innerText = `${allWhiskies.length} ボトル`;
        }

        // Modal Controls
        const modal = document.getElementById('modal');
        const form = document.getElementById('whiskyForm');
        
        document.getElementById('addBtn').onclick = () => {
            form.reset();
            document.getElementById('editId').value = '';
            document.getElementById('modalTitle').innerText = 'ボトルの追加';
            document.getElementById('rangeVal').innerText = '100';
            modal.classList.remove('hidden');
        };

        document.getElementById('cancelBtn').onclick = () => modal.classList.add('hidden');

        window.editWhisky = (id) => {
            const w = allWhiskies.find(i => i.id === id);
            if (!w) return;
            document.getElementById('editId').value = w.id;
            document.getElementById('name').value = w.name;
            document.getElementById('type').value = w.type;
            document.getElementById('remaining').value = w.remaining;
            document.getElementById('rangeVal').innerText = w.remaining;
            document.getElementById('note').value = w.note || '';
            document.getElementById('modalTitle').innerText = '情報を編集';
            modal.classList.remove('hidden');
        };

        document.getElementById('remaining').oninput = (e) => {
            document.getElementById('rangeVal').innerText = e.target.value;
        };

        document.getElementById('searchInput').oninput = () => renderList(allWhiskies);

        // Submit Form
        form.onsubmit = async (e) => {
            e.preventDefault();
            if (!currentUser) return;

            const id = document.getElementById('editId').value;
            const data = {
                name: document.getElementById('name').value,
                type: document.getElementById('type').value,
                remaining: parseInt(document.getElementById('remaining').value),
                note: document.getElementById('note').value,
                updatedAt: new Date().toISOString()
            };

            const colPath = `artifacts/${appId}/users/${currentUser.uid}/whiskies`;
            
            try {
                if (id) {
                    await updateDoc(doc(db, colPath, id), data);
                } else {
                    await addDoc(collection(db, colPath), data);
                }
                modal.classList.add('hidden');
            } catch (err) {
                console.error(err);
            }
        };
    </script>
</body>
</html>