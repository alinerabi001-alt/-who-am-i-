# -who-am-i-<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <meta name="theme-color" content="#6366f1" />
  <title>من أنا؟ - WHO AM I?</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
  <script src="https://unpkg.com/html5-qrcode"></script>
  <style>
    body {
      background-color: #090d16;
      color: #f8fafc;
      font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      -webkit-tap-highlight-color: transparent;
    }
  </style>
</head>
<body class="min-h-screen flex flex-col items-center p-4">
  <div id="app" class="w-full max-w-md flex flex-col flex-1">
    <!-- الهيدر -->
    <header class="flex items-center justify-between bg-slate-900 border border-slate-800 rounded-2xl p-3 mb-4">
      <div class="flex items-center gap-2">
        <span class="text-2xl">🎭</span>
        <div>
          <h1 class="font-black text-base text-indigo-400 leading-tight">من أنا؟</h1>
          <p class="text-[11px] text-slate-400">WHO AM I? • أونلاين</p>
        </div>
      </div>
      <div class="flex gap-1 text-xs font-bold">
        <button onclick="switchTab('game')" id="tab-game" class="px-3 py-1.5 rounded-xl bg-indigo-600 text-white">اللعب</button>
        <button onclick="switchTab('history')" id="tab-history" class="px-3 py-1.5 rounded-xl bg-slate-800 text-slate-400">السجل</button>
        <button onclick="switchTab('leaders')" id="tab-leaders" class="px-3 py-1.5 rounded-xl bg-slate-800 text-slate-400">الأوائل</button>
      </div>
    </header>

    <!-- تنبيه الربط اللحظي -->
    <div id="sync-banner" class="hidden bg-amber-500/15 border border-amber-500/40 text-amber-200 rounded-2xl p-3 text-xs mb-3">
      ⚠️ لكي يلعب شخصان على هاتفين مختلفين عبر الإنترنت، أضف رابط ومفتاح <b>Supabase</b> في إعدادات الكود بالأسفل. حالياً يعمل كوضع تجربة محلي سريع.
    </div>

    <!-- قسم اللعب -->
    <main id="view-game" class="flex-1 flex flex-col"></main>

    <!-- قسم السجل -->
    <section id="view-history" class="hidden space-y-3"></section>

    <!-- قسم المتصدرين -->
    <section id="view-leaders" class="hidden space-y-3"></section>
  </div>

  <script>
    // ==========================================
    // إعدادات الربط الأونلاين (اختياري لربط هاتفين عبر الإنترنت)
    // ==========================================
    const SUPABASE_URL = ""; // مثال: "https://xxxx.supabase.co"
    const SUPABASE_KEY = ""; // مثال: "eyJhbGciOiJIUzI1NiIsInR5cCI6..."

    const FALLBACK_IMG = "https://upload.wikimedia.org/wikipedia/commons/7/7c/Profile_avatar_placeholder_large.png";

    const CATEGORIES = [
      { id: "football", label: "لاعبو كرة قدم", icon: "⚽" },
      { id: "anime", label: "شخصيات أنمي", icon: "🎌" },
      { id: "celebrities", label: "مشاهير العالم", icon: "🌟" },
      { id: "history", label: "شخصيات تاريخية", icon: "🏛️" },
      { id: "movies", label: "أفلام ومسلسلات", icon: "🎬" }
    ];

    const CHARACTERS = [
      { id: 1, name: "ليونيل ميسي", category: "football", img: "https://upload.wikimedia.org/wikipedia/commons/b/b4/Lionel-Messi-Argentina-2022-FIFA-World-Cup_%28cropped%29.jpg" },
      { id: 2, name: "كريستيانو رونالدو", category: "football", img: "https://upload.wikimedia.org/wikipedia/commons/8/8c/Cristiano_Ronaldo_2018.jpg" },
      { id: 3, name: "كيليان مبابي", category: "football", img: "https://upload.wikimedia.org/wikipedia/commons/5/57/2019-07-17_SG_Dynamo_Dresden_vs._Paris_Saint-Germain_by_Sandro_Halank%E2%80%93129_%28cropped%29.jpg" },
      { id: 4, name: "محمد صلاح", category: "football", img: "https://upload.wikimedia.org/wikipedia/commons/c/c1/Mohamed_Salah_2018.jpg" },
      { id: 5, name: "ليفاي أكرمان", category: "anime", img: "https://cdn.myanimelist.net/images/characters/2/241413.jpg" },
      { id: 6, name: "مونكي دي لوفي", category: "anime", img: "https://cdn.myanimelist.net/images/characters/9/310307.jpg" },
      { id: 7, name: "ناروتو أوزوماكي", category: "anime", img: "https://cdn.myanimelist.net/images/characters/2/284121.jpg" },
      { id: 8, name: "غوكو", category: "anime", img: "https://cdn.myanimelist.net/images/characters/10/100345.jpg" },
      { id: 9, name: "ألبرت أينشتاين", category: "history", img: "https://upload.wikimedia.org/wikipedia/commons/d/d3/Albert_Einstein_Head.jpg" },
      { id: 10, name: "صلاح الدين الأيوبي", category: "history", img: "https://upload.wikimedia.org/wikipedia/commons/e/e4/Saladin_by_Jules_David.jpg" },
      { id: 11, name: "كليوباترا", category: "history", img: "https://upload.wikimedia.org/wikipedia/commons/3/3e/Kleopatra-VII.-Altes-Museum.JPG" },
      { id: 12, name: "يوليوس قيصر", category: "history", img: "https://upload.wikimedia.org/wikipedia/commons/0/0a/Caesar_Tusculum.jpg" },
      { id: 13, name: "دواين جونسون", category: "celebrities", img: "https://upload.wikimedia.org/wikipedia/commons/1/1f/Dwayne_Johnson_2014_%28cropped%29.jpg" },
      { id: 14, name: "توم كروز", category: "celebrities", img: "https://upload.wikimedia.org/wikipedia/commons/3/33/Tom_Cruise_by_Gage_Skidmore_2019.jpg" },
      { id: 15, name: "سيلينا غوميز", category: "movies", img: "https://upload.wikimedia.org/wikipedia/commons/8/85/Selena_Gomez_OMW2015.jpg" },
      { id: 16, name: "والتر وايت", category: "movies", img: "https://upload.wikimedia.org/wikipedia/commons/0/03/Walter_White_S5B.png" }
    ];

    let state = {
      playerId: "p_" + Math.random().toString(36).slice(2, 8),
      playerName: "",
      roomCode: "",
      status: "home", // home, waiting, selecting, playing, finished
      p1Name: "",
      p2Name: "",
      p1Char: null,
      p2Char: null,
      categories: ["football"],
      duration: 120,
      timeLeft: 120,
      turn: "p1",
      question: "",
      answer: null,
      p1Score: 0,
      p2Score: 0,
      winner: "",
      reason: "",
      scannerOpen: false,
      feedback: ""
    };

    let timerInterval = null;

    function playSound(type) {
      try {
        const Ctx = window.AudioContext || window.webkitAudioContext;
        const ctx = new Ctx();
        const osc = ctx.createOscillator();
        const gain = ctx.createGain();
        osc.connect(gain);
        gain.connect(ctx.destination);
        if (type === "win") {
          osc.frequency.setValueAtTime(523, ctx.currentTime);
          osc.frequency.setValueAtTime(784, ctx.currentTime + 0.15);
          gain.gain.setValueAtTime(0.2, ctx.currentTime);
          osc.start(); osc.stop(ctx.currentTime + 0.35);
        } else if (type === "lose") {
          osc.type = "sawtooth";
          osc.frequency.setValueAtTime(260, ctx.currentTime);
          osc.frequency.setValueAtTime(150, ctx.currentTime + 0.18);
          gain.gain.setValueAtTime(0.15, ctx.currentTime);
          osc.start(); osc.stop(ctx.currentTime + 0.38);
        } else {
          osc.frequency.setValueAtTime(900, ctx.currentTime);
          gain.gain.setValueAtTime(0.08, ctx.currentTime);
          osc.start(); osc.stop(ctx.currentTime + 0.07);
        }
      } catch (e) {}
    }

    function saveHistory(winnerName, reasonText) {
      const list = JSON.parse(localStorage.getItem("whoami_history") || "[]");
      list.unshift({
        p1: state.p1Name,
        p2: state.p2Name,
        s1: state.p1Score,
        s2: state.p2Score,
        winner: winnerName,
        reason: reasonText,
        date: new Date().toLocaleString("ar")
      });
      localStorage.setItem("whoami_history", JSON.stringify(list.slice(0, 25)));

      const leaders = JSON.parse(localStorage.getItem("whoami_leaders") || "{}");
      [state.p1Name, state.p2Name].forEach((name) => {
        if (!name) return;
        if (!leaders[name]) leaders[name] = { wins: 0, losses: 0, streak: 0, points: 0 };
        if (winnerName === name) {
          leaders[name].wins += 1;
          leaders[name].streak += 1;
          leaders[name].points += 10;
        } else {
          leaders[name].losses += 1;
          leaders[name].streak = 0;
        }
      });
      localStorage.setItem("whoami_leaders", JSON.stringify(leaders));
    }

    function switchTab(tab) {
      ["game", "history", "leaders"].forEach((t) => {
        document.getElementById("view-" + t).classList.toggle("hidden", t !== tab);
        const btn = document.getElementById("tab-" + t);
        btn.className = t === tab
          ? "px-3 py-1.5 rounded-xl bg-indigo-600 text-white"
          : "px-3 py-1.5 rounded-xl bg-slate-800 text-slate-400";
      });
      if (tab === "history") renderHistory();
      if (tab === "leaders") renderLeaders();
    }

    function renderHistory() {
      const list = JSON.parse(localStorage.getItem("whoami_history") || "[]");
      const el = document.getElementById("view-history");
      if (!list.length) {
        el.innerHTML = `<p class="text-center text-slate-400 text-sm py-10">لا توجد مباريات مسجلة بعد.</p>`;
        return;
      }
      el.innerHTML = `<h2 class="font-black text-lg mb-2">📜 سجل المباريات</h2>` + list.map(m => `
        <div class="bg-slate-900 border border-slate-800 rounded-2xl p-3.5 text-sm">
          <div class="flex justify-between font-bold mb-1">
            <span>${m.p1} (${m.s1})</span>
            <span class="text-indigo-400">VS</span>
            <span>${m.p2} (${m.s2})</span>
          </div>
          <p class="text-emerald-400 text-xs font-bold">النتيجة: ${m.winner}</p>
          <p class="text-[11px] text-slate-500 mt-1">${m.date}</p>
        </div>
      `).join("");
    }

    function renderLeaders() {
      const leaders = JSON.parse(localStorage.getItem("whoami_leaders") || "{}");
      const sorted = Object.entries(leaders).sort((a, b) => b[1].points - a[1].points);
      const el = document.getElementById("view-leaders");
      if (!sorted.length) {
        el.innerHTML = `<p class="text-center text-slate-400 text-sm py-10">لا توجد نقاط مسجلة بعد.</p>`;
        return;
      }
      el.innerHTML = `<h2 class="font-black text-lg mb-2">🏆 لوحة المتصدرين</h2>` + sorted.map(([name, s], i) => `
        <div class="bg-slate-900 border border-slate-800 rounded-2xl p-3.5 flex items-center justify-between">
          <div>
            <p class="font-bold text-sm">${i + 1}. ${name}</p>
            <p class="text-xs text-slate-400">فوز: ${s.wins} | خسارة: ${s.losses} | سلسلة: ${s.streak} 🔥</p>
          </div>
          <span class="font-black text-amber-400">${s.points} ن</span>
        </div>
      `).join("");
    }

    function createRoom() {
      const input = document.getElementById("player-name-input");
      if (!input || !input.value.trim()) return alert("اكتب اسمك أولاً");
      state.playerName = input.value.trim();
      state.p1Name = state.playerName;
      state.roomCode = Math.random().toString(36).substring(2, 7).toUpperCase();
      state.status = "waiting";
      renderGame();
      setTimeout(() => {
        const qrBox = document.getElementById("qrcode-box");
        if (qrBox && window.QRCode) {
          qrBox.innerHTML = "";
          new QRCode(qrBox, { text: state.roomCode, width: 180, height: 180 });
        }
      }, 50);
    }

    function joinRoom(codeVal) {
      const nameInput = document.getElementById("player-name-input");
      const name = (nameInput && nameInput.value.trim()) || state.playerName;
      if (!name) return alert("اكتب اسمك أولاً");
      const clean = (codeVal || "").trim().toUpperCase();
      if (!clean) return alert("أدخل كود الغرفة");
      state.playerName = name;
      state.p1Name = state.p1Name || "اللاعب الأول";
      state.p2Name = name;
      state.roomCode = clean;
      state.status = "selecting";
      state.scannerOpen = false;
      renderGame();
    }

    function toggleScanner() {
      state.scannerOpen = !state.scannerOpen;
      renderGame();
      if (state.scannerOpen && window.Html5QrcodeScanner) {
        setTimeout(() => {
          const sc = new Html5QrcodeScanner("qr-scanner", { fps: 10, qrbox: { width: 220, height: 220 } }, false);
          sc.render((decoded) => {
            sc.clear().catch(() => {});
            joinRoom(decoded);
          }, () => {});
        }, 100);
      }
    }

    function toggleCat(id) {
      if (state.categories.includes(id)) {
        if (state.categories.length > 1) {
          state.categories = state.categories.filter(c => c !== id);
        }
      } else {
        state.categories.push(id);
      }
      renderGame();
    }

    function setDuration(sec) {
      state.duration = sec;
      state.timeLeft = sec;
      renderGame();
    }

    function startRound() {
      clearInterval(timerInterval);
      const pool = CHARACTERS.filter(c => state.categories.includes(c.category));
      const list = pool.length >= 2 ? pool : CHARACTERS;
      const idx1 = Math.floor(Math.random() * list.length);
      let idx2 = Math.floor(Math.random() * list.length);
      if (idx2 === idx1) idx2 = (idx1 + 1) % list.length;

      state.p1Char = list[idx1];
      state.p2Char = list[idx2];
      state.timeLeft = state.duration;
      state.question = "";
      state.answer = null;
      state.feedback = "";
      state.status = "playing";
      renderGame();

      timerInterval = setInterval(() => {
        state.timeLeft -= 1;
        if (state.timeLeft <= 10 && state.timeLeft > 0) playSound("tick");
        if (state.timeLeft <= 0) {
          clearInterval(timerInterval);
          state.timeLeft = 0;
          state.status = "finished";
          state.winner = "انتهى الوقت";
          state.reason = "انتهى وقت الجولة قبل تخمين الشخصية!";
          playSound("lose");
          saveHistory(state.winner, state.reason);
          renderGame();
        } else {
          const tEl = document.getElementById("timer-badge");
          if (tEl) {
            tEl.textContent = "⏱️ " + state.timeLeft + "s";
            if (state.timeLeft <= 10) tEl.className = "px-3 py-1 rounded-xl font-mono font-black text-sm bg-red-600 text-white animate-pulse";
          }
        }
      }, 1000);
    }

    function askQuestion() {
      const q = document.getElementById("q-input").value.trim();
      if (!q) return;
      state.question = q;
      state.answer = null;
      renderGame();
    }

    function giveAnswer(ans) {
      state.answer = ans;
      renderGame();
    }

    function submitGuess() {
      const g = document.getElementById("guess-input").value.trim().toLowerCase();
      if (!g) return;
      // نتحقق من شخصية اللاعب الحالي دون إظهارها له على الشاشة
      const secretName = (state.p1Char ? state.p1Char.name : "").toLowerCase();
      if (secretName && (secretName.includes(g) || g === secretName)) {
        clearInterval(timerInterval);
        state.p1Score += 10;
        state.status = "finished";
        state.winner = state.playerName || state.p1Name;
        state.reason = `خمّن الشخصية "${state.p1Char.name}" بشكل صحيح!`;
        playSound("win");
        saveHistory(state.winner, state.reason);
        renderGame();
      } else {
        playSound("lose");
        state.feedback = "❌ تخمين غير صحيح! جرّب سؤالاً آخر.";
        renderGame();
      }
    }

    function renderGame() {
      const view = document.getElementById("view-game");

      if (state.status === "home") {
        view.innerHTML = `
          <div class="flex-1 flex flex-col justify-center">
            <div class="bg-slate-900 border border-slate-800 rounded-3xl p-6 space-y-4">
              <h2 class="text-2xl font-black text-center">ابدأ التحدي</h2>
              <p class="text-xs text-slate-400 text-center">تحدي التخمين المباشر بين لاعبين</p>
              <input id="player-name-input" type="text" value="${state.playerName}" placeholder="اكتب اسمك هنا..." class="w-full p-3.5 rounded-2xl bg-slate-950 border border-slate-800 outline-none focus:border-indigo-500" />
              <button onclick="createRoom()" class="w-full py-3.5 rounded-2xl bg-indigo-600 hover:bg-indigo-500 font-black">➕ إنشاء غرفة جديدة</button>
              <div class="border-t border-slate-800 pt-4 space-y-2.5">
                <p class="text-xs text-slate-400 text-center">أو انضم عبر كود الغرفة / QR</p>
                <div class="flex gap-2">
                  <input id="join-code-input" type="text" placeholder="كود الغرفة" class="flex-1 p-3 rounded-2xl bg-slate-950 border border-slate-800 text-center font-mono uppercase" />
                  <button onclick="joinRoom(document.getElementById('join-code-input').value)" class="px-5 rounded-2xl bg-emerald-600 font-bold">دخول</button>
                </div>
                <button onclick="toggleScanner()" class="w-full py-3 rounded-2xl bg-slate-800 text-indigo-300 text-sm font-bold">${state.scannerOpen ? "إغلاق الكاميرا" : "📷 مسح رمز QR بالكاميرا"}</button>
                <div id="qr-scanner"></div>
              </div>
            </div>
          </div>
        `;
        return;
      }

      if (state.status === "waiting") {
        view.innerHTML = `
          <div class="flex-1 flex flex-col items-center justify-center text-center bg-slate-900 border border-slate-800 rounded-3xl p-6 space-y-4">
            <h2 class="text-xl font-black">بانتظار اللاعب الثاني...</h2>
            <p class="text-xs text-slate-400">اطلب من صديقك مسح رمز QR أو إدخال الكود</p>
            <div id="qrcode-box" class="bg-white p-3 rounded-2xl"></div>
            <div class="bg-slate-950 border border-slate-800 rounded-2xl px-6 py-3">
              <span class="text-xs text-slate-400 block">كود الغرفة</span>
              <span class="text-2xl font-mono font-black text-indigo-400">${state.roomCode}</span>
            </div>
            <button onclick="joinRoom(state.roomCode)" class="w-full py-3 rounded-2xl bg-emerald-600 font-bold text-sm">متابعة إلى اختيار الفئات ←</button>
          </div>
        `;
        return;
      }

      if (state.status === "selecting") {
        view.innerHTML = `
          <div class="flex-1 flex flex-col justify-between bg-slate-900 border border-slate-800 rounded-3xl p-5">
            <div>
              <div class="flex justify-between items-center bg-slate-950 p-3 rounded-2xl mb-4 text-xs font-bold">
                <span class="text-indigo-400">${state.p1Name}</span>
                <span class="bg-slate-800 px-2.5 py-1 rounded-full">VS</span>
                <span class="text-emerald-400">${state.p2Name}</span>
              </div>
              <h2 class="font-black text-sm mb-2">اختر الفئات (يمكن دمج أكثر من فئة)</h2>
              <div class="space-y-2 mb-4">
                ${CATEGORIES.map(c => {
                  const active = state.categories.includes(c.id);
                  return `<button onclick="toggleCat('${c.id}')" class="w-full p-3 rounded-2xl border flex justify-between items-center text-sm font-bold ${active ? "border-indigo-500 bg-indigo-950/50 text-white" : "border-slate-800 bg-slate-950 text-slate-400"}">
                    <span>${c.icon} ${c.label}</span><span>${active ? "✅" : "⬜"}</span>
                  </button>`;
                }).join("")}
              </div>
              <h3 class="font-bold text-xs mb-2">مدة الجولة</h3>
              <div class="grid grid-cols-3 gap-2">
                ${[60, 120, 180].map(sec => `
                  <button onclick="setDuration(${sec})" class="py-2 rounded-xl text-xs font-black border ${state.duration === sec ? "border-amber-400 bg-amber-500/20 text-amber-300" : "border-slate-800 bg-slate-950 text-slate-400"}">${sec / 60} ${sec === 60 ? "دقيقة" : "دقائق"}</button>
                `).join("")}
              </div>
            </div>
            <button onclick="startRound()" class="w-full py-4 mt-5 rounded-2xl bg-emerald-600 hover:bg-emerald-500 font-black text-lg">🚀 ابدأ الجولة</button>
          </div>
        `;
        return;
      }

      if (state.status === "playing") {
        // نعرض شخصية الخص