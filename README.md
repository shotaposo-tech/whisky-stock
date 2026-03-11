# whisky-stock
ウイスキー在庫管理
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <title>My Whisky Stock</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { -webkit-tap-highlight-color: transparent; font-family: -apple-system, sans-serif; background-color: #f8fafc; }
    </style>
</head>
<body class="text-slate-900 pb-24">
    <div id="app" class="max-w-md mx-auto min-h-screen relative">
        <header class="sticky top-0 z-30 bg-white/80 backdrop-blur-md border-b px-5 py-4 flex justify-between items-center">
            <h1 class="text-xl font-bold text-amber-900">🥃 Whisky Stock</h1>
            <div id="stats" class="text-xs font-medium text-slate-500 bg-slate-100 px-3 py-1 rounded-full">読込中...</div>
        </header>

        <div id="whiskyList" class="px-4 mt-6 space-y-3 pb-20">
            <div class="text-center py-10 opacity-50">読み込み中...</div>
        </div>

        <button id="addBtn" class="fixed bottom-8 right-6 w-14 h-14 bg-amber-800 text-white rounded-full shadow-2xl flex items-center justify-center text-3xl z-40">＋</button>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // あなたのFirebase設定を反映済みです
        const firebaseConfig = {
            apiKey: "AIzaSyAk41dGtCDcDGMmvXmknE6MXu_fE_6_R-o",
            authDomain: "my-whisky-app.firebaseapp.com",
            projectId: "my-whisky-app",
            storageBucket: "my-whisky-app.firebasestorage.app",
            messagingSenderId: "599572459740",
            appId: "1:599572459740:web:9f8e4e7e6f8a4e7e6f8a4e" 
        };

        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);

        onAuthStateChanged(auth, user => {
            if (user) {
                const colRef = collection(db, "users", user.uid, "whiskies");
                onSnapshot(colRef, (snapshot) => {
                    const data = snapshot.docs.map(d => ({id: d.id, ...d.data()}));
                    render(data);
                }, (error) => {
                    console.error("Firebase Error:", error);
                });
            } else {
                signInAnonymously(auth);
            }
        });

        function render(data) {
            const list = document.getElementById('whiskyList');
            document.getElementById('stats').innerText = `${data.length}本`;
            if (data.length === 0) {
                list.innerHTML = '<div class="text-center py-20 text-slate-400">ボトルがありません</div>';
                return;
            }
            list.innerHTML = data.map(w => `
                <div class="bg-white rounded-2xl p-4 shadow-sm border border-slate-100 flex items-center justify-between">
                    <div>
                        <h3 class="font-bold text-lg">${w.name}</h3>
                        <p class="text-xs text-slate-400">${new Date(w.createdAt).toLocaleDateString()}</p>
                    </div>
                    <div class="text-amber-800 font-bold">${w.remaining}%</div>
                </div>
            `).join('');
        }

        document.getElementById('addBtn').onclick = async () => {
            const name = prompt("ウイスキーの名前を入力してください");
            if (!name) return;
            try {
                await addDoc(collection(db, "users", auth.currentUser.uid, "whiskies"), {
                    name: name,
                    remaining: 100,
                    createdAt: Date.now()
                });
            } catch (e) {
                alert("保存に失敗しました。Firestoreの「ルール」タブで、テストモード（allow read, write: if true;）になっているか確認してください。");
            }
        };
    </script>
</body>
</html>

