# whisky-stock
ウイスキー在庫管理
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>My Whisky Stock</title>
<script src="https://www.google.com/search?q=https://cdn.tailwindcss.com"></script>
<style>body { -webkit-tap-highlight-color: transparent; }</style>
</head>
<body class="bg-slate-50 text-slate-900">
<div id="app" class="max-w-md mx-auto min-h-screen relative">
<header class="bg-white border-b px-5 py-4 flex justify-between items-center sticky top-0 z-10">
<h1 class="text-xl font-bold text-amber-900">🥃 Whisky Stock</h1>
<div id="stats" class="text-xs bg-slate-100 px-3 py-1 rounded-full font-bold">0本</div>
</header>
<div id="whiskyList" class="px-4 mt-6 space-y-3 pb-24 text-center">
読込中...
</div>
<button id="addBtn" class="fixed bottom-8 right-6 w-16 h-16 bg-amber-800 text-white rounded-full shadow-2xl flex items-center justify-center text-4xl z-20">＋</button>
</div>
<script type="module">
import { initializeApp } from "https://www.google.com/search?q=https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.google.com/search?q=https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
import { getFirestore, collection, addDoc, onSnapshot } from "https://www.google.com/search?q=https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
    const firebaseConfig = {
        apiKey: &quot;AIzaSyAk41dGtCDcDGMmvXmknE6MXu_fE_6_R-o&quot;,
        authDomain: &quot;my-whisky-app.firebaseapp.com&quot;,
        projectId: &quot;my-whisky-app&quot;,
        storageBucket: &quot;my-whisky-app.firebasestorage.app&quot;,
        messagingSenderId: &quot;599572459740&quot;,
        appId: &quot;1:599572459740:web:9f8e4e7e6f8a4e7e6f8a4e&quot; 
    };

    const app = initializeApp(firebaseConfig);
    const auth = getAuth(app);
    const db = getFirestore(app);

    onAuthStateChanged(auth, async (user) =&gt; {
        if (user) {
            const colRef = collection(db, &quot;users&quot;, user.uid, &quot;whiskies&quot;);
            onSnapshot(colRef, (snapshot) =&gt; {
                const data = snapshot.docs.map(d =&gt; ({id: d.id, ...d.data()}));
                data.sort((a, b) =&gt; b.createdAt - a.createdAt);
                document.getElementById(&#39;stats&#39;).innerText = `${data.length}本`;
                const listEl = document.getElementById(&#39;whiskyList&#39;);
                if (data.length === 0) {
                    listEl.innerHTML = &#39;&lt;div class=&quot;py-20 text-slate-400&quot;&gt;ボトルがありません&lt;/div&gt;&#39;;
                } else {
                    listEl.innerHTML = data.map(w =&gt; `
                        &lt;div class=&quot;bg-white rounded-2xl p-4 shadow-sm border border-slate-100 flex items-center justify-between text-left&quot;&gt;
                            &lt;div&gt;&lt;h3 class=&quot;font-bold text-lg&quot;&gt;${w.name}&lt;/h3&gt;&lt;/div&gt;
                            &lt;div class=&quot;text-amber-800 font-bold text-xl&quot;&gt;${w.remaining || 100}%&lt;/div&gt;
                        &lt;/div&gt;
                    `).join(&#39;&#39;);
                }
            });
        } else {
            await signInAnonymously(auth);
        }
    });

    document.getElementById(&#39;addBtn&#39;).onclick = async () =&gt; {
        const name = prompt(&quot;銘柄名を入力してください&quot;);
        if (name &amp;&amp; auth.currentUser) {
            await addDoc(collection(db, &quot;users&quot;, auth.currentUser.uid, &quot;whiskies&quot;), {
                name: name,
                remaining: 100,
                createdAt: Date.now()
            });
        }
    };
&lt;/script&gt;
</body>
</html>


