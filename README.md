<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>TruthSeeker | Digital Feed</title>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&display=swap" rel="stylesheet">
    <style>
        :root {
            --primary: #1877F2;
            --bg: #F0F2F5;
            --white: #ffffff;
            --text: #050505;
        }

        body {
            font-family: 'Inter', sans-serif;
            background-color: var(--bg);
            margin: 0;
            padding: 0;
            color: var(--text);
        }

        /* Fixed Top Header */
        header {
            background: var(--white);
            padding: 15px;
            position: sticky;
            top: 0;
            z-index: 100;
            text-align: center;
            border-bottom: 1px solid #ddd;
            font-weight: 800;
            font-size: 1.2rem;
            color: var(--primary);
        }

        .feed-container {
            max-width: 500px;
            margin: 0 auto;
            padding: 10px;
            padding-bottom: 100px; /* Space for FAB */
        }

        /* Post Card Style */
        .post-card {
            background: var(--white);
            border-radius: 12px;
            margin-bottom: 15px;
            box-shadow: 0 1px 3px rgba(0,0,0,0.1);
            padding: 15px;
        }

        .post-title { font-weight: 800; font-size: 1.1rem; color: var(--primary); margin-bottom: 5px; }
        .verse-box {
            background: #f7f9fc;
            border-left: 4px solid var(--primary);
            padding: 10px;
            margin: 10px 0;
            font-style: italic;
            font-size: 0.9rem;
            color: #444;
        }

        /* FLOATING ACTION BUTTON (FAB) */
        .fab {
            position: fixed;
            bottom: 30px;
            right: 30px;
            width: 60px;
            height: 60px;
            background: var(--primary);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-size: 30px;
            box-shadow: 0 4px 15px rgba(24, 119, 242, 0.4);
            cursor: pointer;
            z-index: 200;
            transition: transform 0.2s;
        }
        .fab:active { transform: scale(0.9); }

        /* POST MODAL (Hidden by default) */
        #postModal {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.5);
            z-index: 300;
            align-items: center;
            justify-content: center;
        }

        .modal-content {
            background: white;
            width: 90%;
            max-width: 400px;
            padding: 20px;
            border-radius: 15px;
        }

        input, textarea {
            width: 100%;
            padding: 12px;
            margin: 8px 0;
            border: 1px solid #ddd;
            border-radius: 8px;
            box-sizing: border-box;
            font-size: 16px;
        }

        .btn-group { display: flex; gap: 10px; margin-top: 10px; }
        .btn { flex: 1; padding: 12px; border-radius: 8px; border: none; font-weight: 700; cursor: pointer; }
        .btn-post { background: var(--primary); color: white; }
        .btn-cancel { background: #eee; color: #555; }
    </style>
</head>
<body>

<header>TRUTHSEEKER</header>

<div class="feed-container" id="mainFeed">
    <p style="text-align: center; color: #888; margin-top: 20px;">Scrolling for Signs...</p>
</div>

<div class="fab" id="openModal">+</div>

<div id="postModal">
    <div class="modal-content">
        <h3>New Evidence</h3>
        <input type="text" id="t" placeholder="Topic (e.g. Iron)">
        <input type="text" id="v" placeholder="Verse (e.g. 57:25)">
        <textarea id="c" placeholder="What is the miracle?"></textarea>
        <div class="btn-group">
            <button class="btn btn-cancel" id="closeModal">Cancel</button>
            <button class="btn btn-post" id="postBtn">Publish</button>
        </div>
    </div>
</div>

<script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
    import { getDatabase, ref, push, onValue, serverTimestamp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-database.js";

    const firebaseConfig = {
      apiKey: "AIzaSyBY-S8Xi_KC4JDbov4xD0a5fITfzId7qjA",
      authDomain: "truthseeker-9efda.firebaseapp.com",
      databaseURL: "https://truthseeker-9efda-default-rtdb.firebaseio.com",
      projectId: "truthseeker-9efda",
      storageBucket: "truthseeker-9efda.firebasestorage.app",
      messagingSenderId: "344387098355",
      appId: "1:344387098355:web:852cfb72d24dee35d84986"
    };

    const app = initializeApp(firebaseConfig);
    const db = getDatabase(app);

    // Modal Control
    const modal = document.getElementById('postModal');
    document.getElementById('openModal').onclick = () => modal.style.display = 'flex';
    document.getElementById('closeModal').onclick = () => modal.style.display = 'none';

    // Posting
    document.getElementById('postBtn').onclick = () => {
        const title = document.getElementById('t').value;
        const verse = document.getElementById('v').value;
        const content = document.getElementById('c').value;

        if(title && verse && content) {
            push(ref(db, 'posts'), {
                title, verse, content,
                time: serverTimestamp()
            }).then(() => {
                modal.style.display = 'none';
                document.getElementById('t').value = '';
                document.getElementById('v').value = '';
                document.getElementById('c').value = '';
            });
        }
    };

    // Feed Loading
    onValue(ref(db, 'posts'), (snap) => {
        const feed = document.getElementById('mainFeed');
        feed.innerHTML = '';
        const data = snap.val();
        if(data) {
            Object.entries(data).reverse().forEach(([id, p]) => {
                const card = `
                    <div class="post-card">
                        <div class="post-title">${p.title}</div>
                        <p>${p.content}</p>
                        <div class="verse-box">"${p.verse}"</div>
                    </div>
                `;
                feed.insertAdjacentHTML('beforeend', card);
            });
        }
    });
</script>

</body>
</html>
