<!DOCTYPE html>
<html lang="az">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kod Studio PRO</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;600;700&family=JetBrains+Mono:wght@400;500&display=swap');
        
        body { font-family: 'Plus Jakarta Sans', sans-serif; background: #0b0f19; color: #e2e8f0; margin: 0; padding-bottom: 100px; }
        .font-mono { font-family: 'JetBrains Mono', monospace; }
        
        .panel { background: #111827; border: 1px solid #1f2937; border-radius: 12px; position: relative; }
        
        .close-btn {
            display: none;
            position: absolute; top: 15px; right: 15px;
            width: 35px; height: 35px; border-radius: 50%;
            background: rgba(239, 68, 68, 0.2); color: #f87171;
            align-items: center; justify-content: center;
            cursor: pointer; z-index: 300; transition: 0.3s;
        }
        .close-btn:hover { background: rgba(239, 68, 68, 0.4); }

        .fs-mode .close-btn { display: flex !important; }

        .bottom-nav {
            position: fixed; bottom: 25px; left: 50%; transform: translateX(-50%);
            background: rgba(17, 24, 39, 0.7); backdrop-filter: blur(20px);
            border: 1px solid rgba(255,255,255,0.1); border-radius: 50px;
            padding: 12px 35px; display: flex; gap: 40px;
            box-shadow: 0 10px 40px -10px rgba(99, 102, 241, 0.4); z-index: 100;
        }

        .nav-btn { 
            display: flex; align-items: center; justify-content: center;
            color: #94a3b8; font-size: 20px; transition: 0.3s;
            width: 45px; height: 45px; border-radius: 50%;
        }
        .nav-btn:hover { color: #818cf8; background: rgba(255,255,255,0.05); }

        .fs-mode { position: fixed !important; top: 0; left: 0; width: 100vw !important; height: 100vh !important; z-index: 200; padding: 10px; background: #0b0f19; }
        
        iframe { width: 100%; height: 100%; border: none; background: white; border-radius: 8px; }
        
        .status-pill { transition: all 0.3s ease; }
    </style>
</head>
<body class="p-4">

    <header class="flex flex-wrap justify-between items-center mb-6 gap-4">
        <div class="flex items-center gap-3">
            <h1 class="text-lg font-bold flex items-center gap-2"><i class="fa-solid fa-code bg-indigo-600 p-2 rounded-lg text-white"></i> Kod Studio <span class="text-[9px] bg-indigo-900 text-indigo-300 px-1.5 py-0.5 rounded">PRO</span></h1>
        </div>
        <div id="saveStatus" class="flex gap-3 text-emerald-500 text-xs items-center px-3 py-1 bg-emerald-500/10 rounded-full status-pill">
            <i class="fa-solid fa-check-circle"></i> <span>Hazır</span>
        </div>
    </header>

    <main class="grid grid-cols-1 lg:grid-cols-2 gap-4 h-[calc(100vh-180px)]">
        <!-- Editor Paneli -->
        <section id="editorSection" class="panel p-3 flex flex-col h-full">
            <div class="close-btn" onclick="toggleFS('editorSection')"><i class="fa-solid fa-xmark"></i></div>
            <div class="flex justify-between items-center mb-2">
                <span class="text-xs font-mono text-indigo-400"><i class="fa-brands fa-html5 mr-2"></i>index.html</span>
            </div>
            <textarea id="codeEditor" class="flex-grow bg-transparent font-mono text-xs resize-none focus:outline-none w-full" placeholder="Kod yazın..."></textarea>
        </section>

        <!-- Preview Paneli -->
        <section id="previewSection" class="panel p-3 flex flex-col h-full">
            <div class="close-btn" onclick="toggleFS('previewSection')"><i class="fa-solid fa-xmark"></i></div>
            <div class="flex justify-between items-center mb-2">
                <span class="text-xs font-mono text-emerald-400"><i class="fa-regular fa-eye mr-2"></i>ÖNİZLƏMƏ</span>
            </div>
            <iframe id="previewFrame"></iframe>
        </section>
    </main>

    <!-- Naviqasiya -->
    <nav class="bottom-nav">
        <button class="nav-btn" onclick="toggleFS('editorSection')" title="Kod"><i class="fa-solid fa-code"></i></button>
        <button class="nav-btn" onclick="toggleFS('previewSection')" title="Bax"><i class="fa-solid fa-eye"></i></button>
        <button class="nav-btn text-red-400" onclick="clearCode()" title="Sil"><i class="fa-solid fa-trash"></i></button>
        <button class="nav-btn text-emerald-400" onclick="copyCode()" title="Kopyala"><i class="fa-solid fa-copy"></i></button>
    </nav>

    <script>
        const editor = document.getElementById('codeEditor');
        const status = document.getElementById('saveStatus');

        function updateStatus(isSaving) {
            if (isSaving) {
                status.innerHTML = '<i class="fa-solid fa-spinner animate-spin"></i> <span>Yadda saxlanılır...</span>';
                status.className = "flex gap-3 text-amber-500 text-xs items-center px-3 py-1 bg-amber-500/10 rounded-full status-pill";
            } else {
                status.innerHTML = '<i class="fa-solid fa-check-circle"></i> <span>Saxlandı</span>';
                status.className = "flex gap-3 text-emerald-500 text-xs items-center px-3 py-1 bg-emerald-500/10 rounded-full status-pill";
            }
        }

        editor.addEventListener('input', () => {
            updateStatus(true);
            setTimeout(() => {
                localStorage.setItem('userCode', editor.value);
                document.getElementById('previewFrame').srcdoc = editor.value;
                updateStatus(false);
            }, 500);
        });

        window.onload = () => {
            const saved = localStorage.getItem('userCode');
            if(saved) { editor.value = saved; document.getElementById('previewFrame').srcdoc = saved; }
        };

        function toggleFS(id) { document.getElementById(id).classList.toggle('fs-mode'); }
        
        function copyCode() {
            editor.select();
            document.execCommand('copy');
            alert("Kod kopyalandı!");
        }
        
        function clearCode() {
            if(confirm("Silmək istədiyinizə əminsiniz?")) {
                editor.value = ''; localStorage.removeItem('userCode'); document.getElementById('previewFrame').srcdoc = '';
            }
        }
    </script>
</body>
</html>
