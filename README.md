<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BPSC AI Hub</title>
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-auth-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-firestore-compat.js"></script>
    
    <style>
        :root { --primary: #800000; --bg: #f4f4f4; }
        body { font-family: sans-serif; background: var(--bg); margin: 0; }
        .hidden { display: none !important; }
        header { background: var(--primary); color: white; padding: 15px; text-align: center; font-weight: bold; }
        .box { background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); width: 85%; max-width: 400px; margin: 20px auto; text-align: center; }
        button { background: var(--primary); color: white; border: none; padding: 12px; width: 100%; cursor: pointer; border-radius: 8px; font-weight: bold; margin-bottom: 10px; }
        input { width: 90%; padding: 12px; margin: 10px 0; border: 1px solid #ccc; border-radius: 8px; }
        
        #quiz-area, #chat-area { height: 65vh; overflow-y: auto; padding: 15px; background: #e5ddd5; display: flex; flex-direction: column; gap: 10px; }
        .bpsc-card { background: white; padding: 15px; border-radius: 10px; border-left: 5px solid var(--primary); }
        .secret-area { background: #fff9c4; border-top: 1px dashed #800000; margin-top: 10px; padding: 10px; border-radius: 5px; }
    </style>
</head>
<body>

    <div id="auth-page">
        <header>BPSC AI - लॉगिन</header>
        <div class="box">
            <input type="email" id="email" placeholder="Email ID">
            <input type="password" id="password" placeholder="Password">
            <button onclick="handleAuth()">प्रवेश करें</button>
        </div>
    </div>

    <div id="menu-page" class="hidden">
        <header>BPSC AI Hub</header>
        <div class="box">
            <p>User: <b id="user-display"></b></p>
            <button onclick="startPractice()">📖 Self Practice (Sirf Question)</button>
            <button style="background:#2c3e50" onclick="showPartnerDash()">💬 Partner Chat (Secret)</button>
            <button style="background:#555" onclick="location.reload()">Logout</button>
        </div>
    </div>

    <div id="practice-page" class="hidden">
        <header>📖 BPSC Self Practice</header>
        <div id="quiz-area"></div>
        <div style="padding:15px; background:white; text-align:center;">
            <button onclick="getNewQuestion()" style="width:auto; padding:10px 30px;">Agla Question</button>
            <button onclick="goHome()" style="background:#555; width:auto; margin-left:10px;">Back</button>
        </div>
    </div>

    <div id="partner-page" class="hidden">
        <header>पार्टनर नेटवर्क</header>
        <div class="box">
            <input type="text" id="target-id" placeholder="Partner Email ID">
            <button onclick="sendRequest()">Request Bhein</button>
            <div id="req-list" style="margin-top:20px; text-align:left;"></div>
            <button style="background:#555" onclick="goHome()">Back</button>
        </div>
    </div>

    <div id="chat-page" class="hidden">
        <header>Secret Partner Chat</header>
        <div id="chat-area"></div>
        <div style="padding:10px; background:white; display:flex; gap:5px;">
            <input type="text" id="chatInput" placeholder="Apna secret msg likhein...">
            <button onclick="sendSecretMsg()" style="width:80px; margin:0;">Post</button>
        </div>
        <button onclick="goHome()" style="background:#555; border-radius:0;">Exit Chat</button>
    </div>

    <script>
        const firebaseConfig = {
            apiKey: "AIzaSyAvomznD7NeC5VAlA2KwJ_d7e-wAzFefl4",
            authDomain: "mysecretbpsc.firebaseapp.com",
            projectId: "mysecretbpsc",
            storageBucket: "mysecretbpsc.firebasestorage.app",
            messagingSenderId: "179352173594",
            appId: "1:179352173594:web:24f94e1300ab8ebc72e5a3"
        };

        firebase.initializeApp(firebaseConfig);
        const auth = firebase.auth();
        const db = firebase.firestore();
        const GEMINI_KEY = "AIzaSyC-z7Fbf9E8cGmdmnAXIodehH1q_m7OCaY";

        let userEmail = "";
        let chatID = "";

        function handleAuth() {
            const e = document.getElementById('email').value;
            const p = document.getElementById('password').value;
            auth.createUserWithEmailAndPassword(e, p).catch(() => auth.signInWithEmailAndPassword(e, p))
            .then(res => {
                userEmail = res.user.email;
                goHome();
                listenRequests();
            }).catch(err => alert(err.message));
        }

        function goHome() {
            document.querySelectorAll('div[id$="-page"]').forEach(d => d.classList.add('hidden'));
            document.getElementById('menu-page').classList.remove('hidden');
            document.getElementById('user-display').innerText = userEmail;
        }

        // --- SELF PRACTICE: NO CHAT ---
        function startPractice() {
            document.getElementById('menu-page').classList.add('hidden');
            document.getElementById('practice-page').classList.remove('hidden');
            getNewQuestion();
        }

        async function getNewQuestion() {
            const area = document.getElementById('quiz-area');
            area.innerHTML = "<p style='text-align:center'>Generating Question...</p>";
            try {
                const res = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${GEMINI_KEY}`, {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({ contents: [{ parts: [{ text: "Generate 1 tough BPSC question in Hindi and 1-word answer. Format: Q | A" }] }] })
                });
                const data = await res.json();
                const [q, a] = data.candidates[0].content.parts[0].text.split('|');
                area.innerHTML = `<div class="bpsc-card">
                    <b>प्रश्न:</b> ${q}<br>
                    <button onclick="this.nextElementSibling.classList.remove('hidden'); this.remove();" style="margin-top:10px; background:#555;">Sahi Uttar Dekhein</button>
                    <div class="secret-area hidden"><b>सही उत्तर:</b> ${a}</div>
                </div>`;
            } catch(e) { area.innerHTML = "API Error! Internet ya key check karein."; }
        }

        // --- PARTNER CHAT: WITH CHAT OPTION ---
        function showPartnerDash() {
            document.getElementById('menu-page').classList.add('hidden');
            document.getElementById('partner-page').classList.remove('hidden');
        }

        function sendRequest() {
            const target = document.getElementById('target-id').value;
            db.collection("requests").add({ from: userEmail, to: target, status: "pending" });
            alert("Request Bhej Di Gayi!");
        }

        function listenRequests() {
            db.collection("requests").where("to", "==", userEmail).onSnapshot(snap => {
                const list = document.getElementById('req-list');
                list.innerHTML = "<h4>Aayi hui requests:</h4>";
                snap.forEach(doc => {
                    if(doc.data().status === "pending") {
                        list.innerHTML += `<p>${doc.data().from} <button onclick="accept('${doc.id}', '${doc.data().from}')" style="width:auto; padding:5px;">Accept</button></p>`;
                    }
                });
            });
            db.collection("requests").where("from", "==", userEmail).where("status", "==", "accepted").onSnapshot(snap => {
                snap.forEach(doc => openChat(doc.data().to));
            });
        }

        function accept(id, partner) {
            db.collection("requests").doc(id).update({ status: "accepted" });
            openChat(partner);
        }

        function openChat(partner) {
            chatID = [userEmail, partner].sort().join("_");
            document.querySelectorAll('div[id$="-page"]').forEach(d => d.classList.add('hidden'));
            document.getElementById('chat-page').classList.remove('hidden');
            listenChatMsgs();
        }

        async function sendSecretMsg() {
            const inp = document.getElementById('chatInput');
            const val = inp.value; if(!val) return;
            inp.disabled = true;

            try {
                const res = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${GEMINI_KEY}`, {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({ contents: [{ parts: [{ text: "Generate 1 tough BPSC exam question in Hindi. Provide 1-word answer. Format: Q | A" }] }] })
                });
                const data = await res.json();
                const [q, a] = data.candidates[0].content.parts[0].text.split('|');

                db.collection("chats").doc(chatID).collection("messages").add({
                    q: q.trim(), a: a.trim(), m: val, sender: userEmail, time: Date.now()
                });
                inp.value = "";
            } catch(e) { alert("Msg send nahi hua. Key ya Internet check karein."); }
            inp.disabled = false;
        }

        function listenChatMsgs() {
            db.collection("chats").doc(chatID).collection("messages").orderBy("time").onSnapshot(snap => {
                const box = document.getElementById('chat-area');
                box.innerHTML = "";
                snap.forEach(doc => {
                    const d = doc.data();
                    const card = document.createElement('div');
                    card.className = "bpsc-card";
                    card.innerHTML = `<b>Q: ${d.q}</b><br>
                        <button onclick="reveal(this)" style="margin-top:5px;">Secret Msg Dekhein</button>
                        <div class="secret-area hidden"><b>Ans: ${d.a}</b><p style="color:maroon"><b>Msg:</b> ${d.m}</p></div>`;
                    box.appendChild(card);
                });
                box.scrollTop = box.scrollHeight;
            });
        }

        function reveal(btn) {
            const area = btn.nextElementSibling;
            area.classList.remove('hidden');
            btn.classList.add('hidden');
            setTimeout(() => { 
                btn.parentElement.style.opacity = "0.1";
                area.innerHTML = "Expired!";
                setTimeout(() => btn.parentElement.remove(), 1000);
            }, 5000);
        }
    </script>
</body>
</html>
