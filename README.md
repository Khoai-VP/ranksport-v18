<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>RankSport MVP - Commercial Edition</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@700&family=Inter:wght@400;600;700;800&display=swap" rel="stylesheet">
    <style>
        :root { --bg: #0B0F1A; --card: #161B2D; --primary: #C5A059; --accent: #3ABFF8; }
        body { background: var(--bg); color: #F1F5F9; font-family: 'Inter', sans-serif; overflow-x: hidden; }
        .rank-gradient { background: linear-gradient(90deg, #C5A059 0%, #FDE68A 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        .cyber-card { background: var(--card); border: 1px solid rgba(197, 160, 89, 0.1); border-radius: 20px; transition: 0.3s; }
        .cyber-card:active { transform: scale(0.98); border-color: var(--primary); }
        .nav-btn { font-size: 11px; font-weight: 800; letter-spacing: 1px; text-transform: uppercase; transition: 0.3s; }
        .nav-btn.active { color: var(--primary); border-bottom: 2px solid var(--primary); }
        .hidden { display: none !important; }
        ::-webkit-scrollbar { display: none; }
    </style>
</head>
<body>

    <div id="auth-screen" class="min-h-screen flex flex-col justify-center items-center px-10 text-center">
        <div class="mb-12">
            <h1 style="font-family:'Orbitron'" class="text-5xl rank-gradient mb-2 italic tracking-tighter">RANK SPORT</h1>
            <p class="text-slate-500 text-[10px] uppercase tracking-[0.4em] font-black">Professional Athlete Identity</p>
        </div>
        <button id="login-google" class="w-full bg-white text-black font-extrabold py-4 rounded-2xl flex items-center justify-center gap-3 shadow-2xl hover:bg-slate-100 transition">
            <img src="https://www.gstatic.com/firebasejs/ui/2.0.0/images/auth/google.svg" width="22">
            TIẾP TỤC VỚI GOOGLE
        </button>
        <p class="mt-8 text-slate-600 text-[10px] italic leading-relaxed">Bằng cách đăng nhập, ông đồng ý với điều khoản<br>vận động viên chuyên nghiệp của RankSport.</p>
    </div>

    <div id="main-app" class="hidden min-h-screen flex flex-col">
        <div class="p-6 flex justify-between items-start sticky top-0 bg-[#0B0F1A]/90 backdrop-blur-xl z-50">
            <div class="flex items-center gap-4">
                <div class="relative">
                    <img id="user-avatar" src="" class="w-14 h-14 rounded-2xl border-2 border-[#C5A059] object-cover">
                    <div class="absolute -bottom-1 -right-1 bg-green-500 w-4 h-4 rounded-full border-2 border-[#0B0F1A]"></div>
                </div>
                <div>
                    <h2 id="user-name" class="font-black text-xl italic tracking-tight uppercase">VĐV NAME</h2>
                    <div class="flex items-center gap-2 mt-0.5">
                        <span id="user-rank" class="bg-[#C5A059] text-black text-[9px] font-black px-2 py-0.5 rounded italic">GOLD III</span>
                        <span class="text-[9px] text-slate-500 font-bold uppercase tracking-tighter italic">Verified</span>
                    </div>
                </div>
            </div>
            <div class="text-right">
                <p id="user-elo" class="text-3xl font-black italic text-[#C5A059] leading-none">----</p>
                <p class="text-[9px] text-slate-500 font-bold uppercase mt-1 tracking-tighter">Elo Rating</p>
            </div>
        </div>

        <div class="flex px-6 border-b border-white/5 bg-[#0B0F1A]">
            <button onclick="switchTab('ranking')" id="btn-ranking" class="nav-btn active py-4 mr-6">Xếp hạng</button>
            <button onclick="switchTab('courts')" id="btn-courts" class="nav-btn py-4 mr-6 text-slate-500">Sân chơi</button>
            <button onclick="switchTab('history')" id="btn-history" class="nav-btn py-4 text-slate-500">Lịch sử</button>
        </div>

        <main class="flex-1 p-6 pb-20 overflow-y-auto">
            
            <section id="tab-ranking" class="space-y-3">
                <div class="flex justify-between items-center mb-4">
                    <h3 class="text-[10px] font-black text-slate-500 uppercase tracking-widest italic font-bold">Top Leaderboard</h3>
                    <span class="text-[9px] text-[#C5A059] font-bold">Cập nhật: Real-time</span>
                </div>
                <div id="ranking-list" class="space-y-3">
                    </div>
            </section>

            <section id="tab-courts" class="hidden space-y-4">
                <h3 class="text-[10px] font-black text-slate-500 uppercase tracking-widest italic font-bold">Sân chơi hệ thống</h3>
                <div id="court-list" class="space-y-4">
                    </div>
            </section>

        </main>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/9.22.1/firebase-app.js";
        import { getAuth, signInWithPopup, GoogleAuthProvider } from "https://www.gstatic.com/firebasejs/9.22.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/9.22.1/firebase-firestore.js";

        const firebaseConfig = {
            apiKey: "AIzaSyBpytj0KQybf55iabBzmeg59VL-9oADPHA",
            authDomain: "badrank-89mrd.firebaseapp.com",
            projectId: "badrank-89mrd",
            storageBucket: "badrank-89mrd.firebasestorage.app",
            messagingSenderId: "94437268200",
            appId: "1:94437268200:web:b1726f5d55cc5e631c475d"
        };

        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const provider = new GoogleAuthProvider();

        // MOCK DATA ĐỂ TEST
        const mockVdv = [
            { name: "Phạm Minh Khoai", elo: 1650, rank: "DIAMOND II", img: "https://api.dicebear.com/7.x/avataaars/svg?seed=Felix" },
            { name: "Nguyễn Công Vinh", elo: 1420, rank: "PLATINUM I", img: "https://api.dicebear.com/7.x/avataaars/svg?seed=Aneka" },
            { name: "Lê Văn Tám", elo: 1210, rank: "GOLD III", img: "https://api.dicebear.com/7.x/avataaars/svg?seed=Jack" }
        ];

        const mockCourts = [
            { name: "Sân Cầu Lông Kỳ Hòa", address: "Sư Vạn Hạnh, Q10", slots: "4/10", icon: "🏸" },
            { name: "Sân Đào Duy Anh", address: "Hồ Văn Huê, Phú Nhuận", slots: "Đã đầy", icon: "🏆" }
        ];

        // XỬ LÝ ĐĂNG NHẬP GOOGLE
        document.getElementById('login-google').onclick = async () => {
            try {
                const result = await signInWithPopup(auth, provider);
                const user = result.user;
                showMainApp(user);
                setupRealtime(user);
                renderMockContent();
            } catch (error) {
                console.error(error);
                alert("Lỗi đăng nhập: " + error.message);
            }
        };

        function showMainApp(user) {
            document.getElementById('auth-screen').classList.add('hidden');
            document.getElementById('main-app').classList.remove('hidden');
            document.getElementById('user-name').innerText = user.displayName;
            document.getElementById('user-avatar').src = user.photoURL;
        }

        function renderMockContent() {
            // Render VĐV Rankings
            document.getElementById('ranking-list').innerHTML = mockVdv.map((v, i) => `
                <div class="cyber-card p-4 flex justify-between items-center shadow-lg">
                    <div class="flex items-center gap-4">
                        <span class="text-xs font-black text-slate-700 italic">#0${i+1}</span>
                        <img src="${v.img}" class="w-10 h-10 rounded-full bg-slate-800">
                        <div>
                            <p class="font-bold text-sm tracking-tight">${v.name}</p>
                            <p class="text-[8px] text-[#C5A059] font-black italic uppercase">${v.rank}</p>
                        </div>
                    </div>
                    <div class="text-right">
                        <p class="font-black italic text-[#C5A059]">${v.elo}</p>
                        <p class="text-[7px] text-slate-500 font-bold uppercase">Elo</p>
                    </div>
                </div>
            `).join('');

            // Render Courts
            document.getElementById('court-list').innerHTML = mockCourts.map(c => `
                <div class="cyber-card p-5 relative overflow-hidden">
                    <div class="flex justify-between items-start">
                        <div>
                            <div class="flex items-center gap-2 mb-1">
                                <span class="text-xl">${c.icon}</span>
                                <h4 class="font-black text-sm uppercase italic tracking-tight">${c.name}</h4>
                            </div>
                            <p class="text-[10px] text-slate-400 font-medium tracking-tight">📍 ${c.address}</p>
                        </div>
                        <span class="bg-white/5 text-[9px] px-2 py-1 rounded-md font-bold text-[#C5A059]">${c.slots}</span>
                    </div>
                </div>
            `).join('');
        }

        function setupRealtime(user) {
            onSnapshot(doc(db, "users", user.uid), (docSnap) => {
                if (docSnap.exists()) {
                    const data = docSnap.data();
                    document.getElementById('user-elo').innerText = data.elo || 1200;
                    document.getElementById('user-rank').innerText = data.rank || "NEWBIE";
                } else {
                    // Tạo profile mặc định nếu lần đầu đăng nhập
                    setDoc(doc(db, "users", user.uid), {
                        displayName: user.displayName,
                        elo: 1200,
                        rank: "BRONZE I",
                        photo: user.photoURL
                    });
                }
            });
        }

        window.switchTab = (tab) => {
            const tabs = ['ranking', 'courts', 'history'];
            tabs.forEach(t => {
                document.getElementById(`tab-${t}`)?.classList.toggle('hidden', t !== tab);
                document.getElementById(`btn-${t}`)?.classList.toggle('active', t === tab);
                document.getElementById(`btn-${t}`)?.classList.toggle('text-slate-500', t !== tab);
            });
        };
    </script>
</body>
</html>
