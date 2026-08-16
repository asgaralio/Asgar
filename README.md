<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>AI Image Studio</title>
    <style>
        :root {
            --bg-primary: #0b0d12;
            --bg-secondary: #141720;
            --bg-card: #1c2128;
            --text-primary: #ffffff;
            --text-secondary: #8b949e;
            --accent-purple: #7c3aed;
            --accent-purple-light: #a78bfa;
            --border-color: #2d333b;
        }

        * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
        
        body { background: var(--bg-primary); color: var(--text-primary); display: flex; justify-content: center; min-height: 100vh; }
        
        #app { width: 100%; max-width: 450px; background: var(--bg-primary); min-height: 100vh; position: relative; padding-bottom: 80px; }

        /* Header */
        .header { display: flex; justify-content: space-between; align-items: center; padding: 16px 20px; }
        .header h1 { font-size: 24px; font-weight: 700; background: linear-gradient(135deg, #a78bfa, #ec4899); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        .header-right { display: flex; gap: 16px; align-items: center; }
        .header-right button { background: var(--bg-card); border: 1px solid var(--border-color); color: var(--text-primary); padding: 8px 14px; border-radius: 20px; font-size: 13px; cursor: pointer; }
        .avatar { width: 36px; height: 36px; border-radius: 50%; background: linear-gradient(135deg, #7c3aed, #6d28d9); display: flex; align-items: center; justify-content: center; font-weight: 600; font-size: 14px; }

        /* Toast */
        #toast { position: fixed; top: 20px; left: 50%; transform: translateX(-50%); background: rgba(0,0,0,0.9); color: #fff; padding: 12px 20px; border-radius: 12px; font-size: 14px; z-index: 999; display: none; border: 1px solid var(--border-color); }

        /* Generate Section */
        .generate-section { padding: 0 20px 20px; }
        .prompt-container { background: var(--bg-secondary); border: 1px solid var(--border-color); border-radius: 16px; padding: 12px; display: flex; gap: 12px; align-items: flex-end; }
        .prompt-container textarea { flex: 1; background: transparent; border: none; color: var(--text-primary); resize: none; height: 48px; font-size: 15px; padding: 4px; outline: none; line-height: 1.4; font-family: inherit; }
        .prompt-container textarea::placeholder { color: var(--text-secondary); }
        .generate-btn { background: linear-gradient(135deg, #7c3aed, #6d28d9); border: none; border-radius: 12px; width: 44px; height: 44px; display: flex; align-items: center; justify-content: center; cursor: pointer; flex-shrink: 0; transition: 0.2s; color: #fff; font-size: 20px; }
        .generate-btn:active { transform: scale(0.95); }

        /* Toolbar (Aspect Ratio & Styles) */
        .toolbar { padding: 0 20px 16px; display: flex; gap: 12px; flex-wrap: wrap; }
        .toolbar-select { background: var(--bg-secondary); border: 1px solid var(--border-color); color: var(--text-primary); padding: 8px 14px; border-radius: 12px; font-size: 13px; cursor: pointer; outline: none; flex: 1; min-width: 80px; }
        .toolbar-select option { background: var(--bg-primary); }

        .style-chips { display: flex; gap: 8px; flex-wrap: wrap; margin: 0 20px 20px; }
        .style-chip { background: var(--bg-secondary); border: 1px solid var(--border-color); padding: 6px 14px; border-radius: 20px; font-size: 12px; font-weight: 500; cursor: pointer; transition: 0.2s; color: var(--text-secondary); }
        .style-chip.active { background: var(--accent-purple); border-color: var(--accent-purple); color: #fff; }
        .style-chip:active { transform: scale(0.95); }

        /* Image Grid */
        .section-header { display: flex; justify-content: space-between; align-items: center; padding: 0 20px 12px; }
        .section-header h3 { font-size: 17px; font-weight: 600; }
        .section-header span { color: var(--text-secondary); font-size: 13px; }

        .image-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; padding: 0 20px 20px; }
        .image-card { background: var(--bg-card); border-radius: 16px; overflow: hidden; position: relative; aspect-ratio: 1; border: 1px solid var(--border-color); }
        .image-card img { width: 100%; height: 100%; object-fit: cover; display: block; }
        .image-card .overlay { position: absolute; bottom: 0; left: 0; right: 0; background: linear-gradient(to top, rgba(0,0,0,0.8), transparent); padding: 12px; display: flex; justify-content: space-between; align-items: flex-end; opacity: 0; transition: 0.3s; }
        .image-card:hover .overlay, .image-card:active .overlay { opacity: 1; }
        .image-card .overlay button { background: rgba(255,255,255,0.2); backdrop-filter: blur(4px); border: none; color: #fff; padding: 6px 12px; border-radius: 8px; font-size: 12px; cursor: pointer; }

        /* Loading Skeleton */
        .loading-card { background: var(--bg-card); border-radius: 16px; aspect-ratio: 1; display: flex; align-items: center; justify-content: center; border: 1px solid var(--border-color); }
        .spinner { width: 32px; height: 32px; border: 3px solid var(--border-color); border-top-color: var(--accent-purple); border-radius: 50%; animation: spin 0.8s linear infinite; }
        @keyframes spin { to { transform: rotate(360deg); } }

        /* Bottom Nav */
        .bottom-nav { position: fixed; bottom: 0; left: 50%; transform: translateX(-50%); width: 100%; max-width: 450px; background: var(--bg-secondary); border-top: 1px solid var(--border-color); display: flex; justify-content: space-around; padding: 8px 0 16px; z-index: 100; }
        .nav-item { display: flex; flex-direction: column; align-items: center; gap: 4px; cursor: pointer; color: var(--text-secondary); font-size: 10px; font-weight: 500; border: none; background: none; padding: 6px 16px; transition: 0.2s; }
        .nav-item.active { color: var(--text-primary); }
        .nav-item svg { width: 22px; height: 22px; fill: currentColor; }

        /* Modal - Image Preview */
        .modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.85); backdrop-filter: blur(8px); z-index: 200; display: none; justify-content: center; align-items: center; padding: 20px; }
        .modal-overlay.active { display: flex; }
        .modal-img { max-width: 100%; max-height: 80vh; border-radius: 16px; box-shadow: 0 20px 60px rgba(0,0,0,0.8); }
        .modal-close { position: absolute; top: 20px; right: 20px; background: rgba(255,255,255,0.1); border: none; color: #fff; width: 40px; height: 40px; border-radius: 50%; font-size: 20px; cursor: pointer; backdrop-filter: blur(4px); }

    </style>
</head>
<body>

<div id="toast">Notification</div>

<div id="app">
    <!-- Header -->
    <div class="header">
        <h1>✦ Imagify</h1>
        <div class="header-right">
            <button>✨ Credits: 25</button>
            <div class="avatar">AI</div>
        </div>
    </div>

    <!-- Generate Input -->
    <div class="generate-section">
        <div class="prompt-container">
            <textarea id="promptInput" placeholder="Describe your image... (e.g., A futuristic city at sunset, cyberpunk style)">A majestic dragon flying over a neon-lit cyberpunk city at night, hyper-realistic, 8k</textarea>
            <button class="generate-btn" onclick="generateImage()">✦</button>
        </div>
    </div>

    <!-- Toolbar -->
    <div class="toolbar">
        <select class="toolbar-select" id="aspectSelect">
            <option value="1:1">1:1 Square</option>
            <option value="16:9">16:9 Landscape</option>
            <option value="9:16">9:16 Portrait</option>
        </select>
        <select class="toolbar-select" id="modelSelect">
            <option value="v2">Model V2</option>
            <option value="v3">Model V3 (Premium)</option>
        </select>
    </div>

    <!-- Style Chips -->
    <div class="style-chips" id="styleChips">
        <div class="style-chip active" onclick="selectStyle(this)">Realistic</div>
        <div class="style-chip" onclick="selectStyle(this)">Anime</div>
        <div class="style-chip" onclick="selectStyle(this)">3D Render</div>
        <div class="style-chip" onclick="selectStyle(this)">Oil Painting</div>
        <div class="style-chip" onclick="selectStyle(this)">Cinematic</div>
    </div>

    <!-- Gallery Header -->
    <div class="section-header">
        <h3>Recent Generations</h3>
        <span onclick="showToast('History loaded')">See All →</span>
    </div>

    <!-- Image Grid -->
    <div class="image-grid" id="imageGrid">
        <!-- Dummy initial images -->
        <div class="image-card">
            <img src="https://images.unsplash.com/photo-1682687220742-aba13b6e50ba?w=400&h=400&fit=crop" alt="AI Gen">
            <div class="overlay"><button>Download</button></div>
        </div>
        <div class="image-card">
            <img src="https://images.unsplash.com/photo-1682687982501-1e58f8100c71?w=400&h=400&fit=crop" alt="AI Gen">
            <div class="overlay"><button>Download</button></div>
        </div>
        <div class="image-card">
            <img src="https://images.unsplash.com/photo-1682695794816-7b9da18ed470?w=400&h=400&fit=crop" alt="AI Gen">
            <div class="overlay"><button>Download</button></div>
        </div>
        <div class="image-card">
            <img src="https://images.unsplash.com/photo-1682685797507-d44d838b2ac7?w=400&h=400&fit=crop" alt="AI Gen">
            <div class="overlay"><button>Download</button></div>
        </div>
    </div>

    <!-- Bottom Navigation -->
    <nav class="bottom-nav">
        <button class="nav-item active" onclick="switchPage('home')">
            <svg viewBox="0 0 24 24"><path d="M10 20v-6h4v6h5v-8h3L12 3 2 12h3v8z"/></svg>
            Home
        </button>
        <button class="nav-item" onclick="switchPage('explore')">
            <svg viewBox="0 0 24 24"><path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"/></svg>
            Explore
        </button>
        <button class="nav-item" onclick="switchPage('favorites')">
            <svg viewBox="0 0 24 24"><path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/></svg>
            Saved
        </button>
        <button class="nav-item" onclick="switchPage('profile')">
            <svg viewBox="0 0 24 24"><path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z"/></svg>
            Profile
        </button>
    </nav>
</div>

<!-- Image Preview Modal -->
<div class="modal-overlay" id="previewModal" onclick="closePreview()">
    <button class="modal-close" onclick="closePreview()">✕</button>
    <img class="modal-img" id="previewImage" src="" alt="Preview">
</div>

<script>
    // --- Helper: Toast ---
    function showToast(msg) {
        const el = document.getElementById('toast');
        el.innerText = msg;
        el.style.display = 'block';
        setTimeout(() => { el.style.display = 'none'; }, 2500);
    }

    // --- Style Chips ---
    function selectStyle(el) {
        document.querySelectorAll('.style-chip').forEach(c => c.classList.remove('active'));
        el.classList.add('active');
    }

    // --- Mock Image Generation (UI Simulation) ---
    function generateImage() {
        const prompt = document.getElementById('promptInput').value.trim();
        if(!prompt) { showToast('Please write a prompt!'); return; }

        const grid = document.getElementById('imageGrid');
        
        // 1. Add Loading Skeleton
        const skeleton = document.createElement('div');
        skeleton.className = 'loading-card';
        skeleton.id = 'loadingSkeleton';
        skeleton.innerHTML = '<div class="spinner"></div>';
        grid.prepend(skeleton);

        // 2. Simulate AI Wait (2 seconds)
        showToast('🎨 AI is generating your image...');
        setTimeout(() => {
            // Remove skeleton
            skeleton.remove();

            // 3. Generate a random image URL (Simulating AI output)
            const aspect = document.getElementById('aspectSelect').value;
            let dim = '400x400';
            if(aspect === '16:9') dim = '600x338';
            if(aspect === '9:16') dim = '338x600';

            const style = document.querySelector('.style-chip.active').innerText.toLowerCase();
            const randomId = Date.now() + Math.floor(Math.random() * 1000);
            
            // Using Unsplash source for demo (Realistic look)
            const imageUrl = `https://images.unsplash.com/photo-${randomId}?w=600&h=600&fit=crop&crop=entropy`;

            // 4. Create Image Card
            const card = document.createElement('div');
            card.className = 'image-card';
            card.innerHTML = `
                <img src="${imageUrl}" alt="${prompt}" loading="lazy" onclick="openPreview('${imageUrl}')">
                <div class="overlay">
                    <button onclick="event.stopPropagation(); showToast('Downloading image...')">⬇ Download</button>
                    <button onclick="event.stopPropagation(); showToast('Image saved!')">❤️</button>
                </div>
            `;
            
            // 5. Add to Grid
            grid.prepend(card);
            showToast('✅ Image generated successfully!');
        }, 2000);
    }

    // --- Image Preview Modal ---
    function openPreview(url) {
        document.getElementById('previewImage').src = url;
        document.getElementById('previewModal').classList.add('active');
    }

    function closePreview() {
        document.getElementById('previewModal').classList.remove('active');
    }

    // --- Bottom Nav Switching (Demo) ---
    function switchPage(page) {
        document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
        // Find button by text content (rough method for demo)
        const btns = document.querySelectorAll('.nav-item');
        const map = { home:0, explore:1, favorites:2, profile:3 };
        if(map[page] !== undefined) btns[map[page]].classList.add('active');
        
        showToast(`Navigated to ${page.charAt(0).toUpperCase() + page.slice(1)}`);
    }
</script>

</body>
</html>
