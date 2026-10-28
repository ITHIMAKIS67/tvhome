<!DOCTYPE html>
<html lang="el">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Ελληνική Τηλεόραση - HOME BETA CC</title>

    <!-- HLS -->
    <script src="https://cdn.jsdelivr.net/npm/hls.js@1.6.16/dist/hls.min.js"></script>

    <!-- MPEG-DASH -->
    <script src="https://cdn.dashjs.org/latest/dash.all.min.js"></script>

    <style>
        /* =========================
           ΒΑΣΙΚΑ
        ========================= */
        * {
            box-sizing: border-box;
        }

        body {
            margin: 0;
            padding: 15px;
            background: #0b0b0e;
            color: white;
            font-family: Arial, sans-serif;
        }

        .container {
            width: 100%;
            max-width: 1000px;
            margin: auto;
        }

        h1 {
            text-align: center;
            font-size: 24px;
            margin-bottom: 15px;
        }

        /* =========================
           VIDEO
        ========================= */
        .video-wrapper {
            position: relative;
            width: 100%;
            background: black;
            border-radius: 14px;
            overflow: hidden;
        }

        video {
            display: block;
            width: 100%;
            aspect-ratio: 16 / 9;
            background: black;
        }

        /* =========================
           ΥΠΟΤΙΤΛΟΙ
        ========================= */
        #captionOverlay {
            position: absolute;
            left: 4%;
            right: 4%;
            bottom: 8%;
            text-align: center;
            pointer-events: none;
            z-index: 20;
            font-size: 34px;
            font-weight: bold;
            line-height: 1.40;
            color: white;
            text-shadow: 2px 2px 5px black, -2px -2px 5px black, 0 0 8px black;
        }

        .caption-line {
            display: block;
            min-height: 45px;
        }

        /* =========================
           STATUS
        ========================= */
        .status {
            margin-top: 10px;
            padding: 11px 14px;
            background: #171a21;
            border: 1px solid #303744;
            border-radius: 8px;
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .dot {
            width: 10px;
            height: 10px;
            min-width: 10px;
            background: #32d74b;
            border-radius: 50%;
            box-shadow: 0 0 7px #32d74b;
        }

        .dot.offline {
            background: #ff453a;
            box-shadow: 0 0 7px #ff453a;
        }

        /* =========================
           BUTTONS
        ========================= */
        .buttons {
            display: flex;
            gap: 8px;
            flex-wrap: wrap;
            margin-top: 12px;
        }

        button {
            flex: 1;
            min-width: 130px;
            padding: 12px;
            border: 1px solid #394150;
            border-radius: 8px;
            background: #1b2029;
            color: white;
            cursor: pointer;
            font-size: 15px;
        }

        button:hover {
            background: #293241;
        }

        /* =========================
           CC ΣΑΝ TV / KODI
        ========================= */
        #ccBtn {
            background: #1b2029;
            color: white;
            border: 2px solid #596273;
            font-size: 18px;
            font-weight: 900;
        }

        #ccBtn.active {
            background: #087cff;
            color: white;
            border: 3px solid #39d353;
            box-shadow: 0 0 10px rgba(57, 211, 83, 0.65);
        }

        /* =========================
           ΚΑΝΑΛΙΑ
        ========================= */
        #channelPanel {
            display: none;
            margin-top: 12px;
            padding: 12px;
            background: #141720;
            border: 1px solid #303744;
            border-radius: 10px;
        }

        #channelPanel.open {
            display: block;
        }

        #search {
            width: 100%;
            padding: 12px;
            margin-bottom: 10px;
            border-radius: 7px;
            border: 1px solid #394150;
            background: #0e1117;
            color: white;
            font-size: 16px;
        }

        #channelGrid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
            gap: 7px;
            max-height: 400px;
            overflow-y: auto;
        }

        .channel {
            padding: 10px 6px;
            background: #1b2029;
            border: 1px solid #394150;
            border-radius: 7px;
            color: white;
            cursor: pointer;
        }

        .channel:hover {
            background: #293241;
        }

        .channel.selected {
            background: #245b99;
            border-color: #4a9eff;
        }

        /* =========================
           INFO
        ========================= */
        .info {
            margin-top: 12px;
            text-align: center;
            color: #8993a4;
            font-size: 13px;
        }

        /* =========================
           FULLSCREEN
        ========================= */
        .video-wrapper:fullscreen {
            width: 100vw;
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            background: black;
        }

        .video-wrapper:fullscreen video {
            width: 100vw;
            height: 100vh;
            object-fit: contain;
        }

        .video-wrapper:fullscreen #captionOverlay {
            font-size: 46px;
        }

        /* =========================
           ΚΙΝΗΤΟ
        ========================= */
        @media(max-width:600px) {
            body { padding: 5px; }
            h1 { font-size: 18px; }
            #captionOverlay { font-size: 24px; left: 3%; right: 3%; bottom: 8%; }
            .caption-line { min-height: 32px; }
            #channelGrid { grid-template-columns: repeat(2, 1fr); }
            button { min-width: 100px; font-size: 13px; }
            #ccBtn { font-size: 17px; }
        }
    </style>
</head>
<body>

<div class="container">
    <h1>📺 HOME BETA CC</h1>

    <!-- TV -->
    <div class="video-wrapper" id="videoWrapper">
        <video id="video" controls autoplay muted playsinline></video>
        <!-- LIVE CC -->
        <div id="captionOverlay">
            <span class="caption-line" id="caption1"></span>
            <span class="caption-line" id="caption2"></span>
        </div>
    </div>

    <!-- STATUS -->
    <div class="status">
        <span class="dot" id="dot"></span>
        <span id="status">Φόρτωση καναλιών...</span>
    </div>

    <!-- BUTTONS -->
    <div class="buttons">
        <button id="channelsBtn">📺 Κανάλια</button>
        <button id="restartBtn">↻ Επανεκκίνηση</button>
        <button id="ccBtn" title="Ζωντανοί υπότιτλοι">CC</button>
        <button id="fullscreenBtn">⛶ Πλήρης</button>
    </div>

    <!-- CHANNEL PANEL -->
    <div id="channelPanel">
        <input id="search" type="text" placeholder="🔍 Αναζήτηση καναλιού...">
        <div id="channelGrid">Φόρτωση...</div>
    </div>

    <div class="info">
        HOME BETA CC • HLS + MPEG-DASH • Live CC Ελληνικά
    </div>
</div>

<script>
    /* ==========================================
       PLAYLIST
    ========================================== */
    const M3U_URL = 'https://raw.githubusercontent.com/ITHIMAKIS67/home-beta-cc/main/playlist_greece.m3u';
    const STORAGE_KEY = 'home_beta_cc_channel';

    /* ==========================================
       ELEMENTS
    ========================================== */
    const video = document.getElementById('video');
    const videoWrapper = document.getElementById('videoWrapper');
    const statusText = document.getElementById('status');
    const dot = document.getElementById('dot');
    const channelsBtn = document.getElementById('channelsBtn');
    const restartBtn = document.getElementById('restartBtn');
    const ccBtn = document.getElementById('ccBtn');
    const fullscreenBtn = document.getElementById('fullscreenBtn');
    const channelPanel = document.getElementById('channelPanel');
    const channelGrid = document.getElementById('channelGrid');
    const search = document.getElementById('search');
    const caption1 = document.getElementById('caption1');
    const caption2 = document.getElementById('caption2');

    /* ==========================================
       VARIABLES
    ========================================== */
    let channels = [];
    let currentIndex = 0;
    let hls = null;
    let dashPlayer = null;
    let recognition = null;
    let captionsOn = false;
    let captionTimer = null;

    /* ==========================================
       STATUS
    ========================================== */
    function setStatus(text, online = true) {
        statusText.textContent = text;
        if (online) {
            dot.classList.remove('offline');
        } else {
            dot.classList.add('offline');
        }
    }

    /* ==========================================
       M3U PARSER
    ========================================== */
    function parseM3U(text) {
        const lines = text.split(/\r?\n/);
        const result = [];
        let currentName = null;
        let currentId = '';

        for (let i = 0; i < lines.length; i++) {
            const line = lines[i].trim();
            if (!line) continue;

            if (line.startsWith('#EXTINF:')) {
                const comma = line.indexOf(',');
                if (comma !== -1) {
                    currentName = line.substring(comma + 1).trim();
                } else {
                    currentName = 'Κανάλι';
                }
                const idMatch = line.match(/tvg-id="([^"]*)"/i);
                currentId = idMatch ? idMatch[1] : '';
            } else if (line.startsWith('#')) {
                continue;
            } else if (currentName && /^https?:\/\//i.test(line)) {
                result.push({
                    name: currentName,
                    url: line,
                    id: currentId
                });
                currentName = null;
                currentId = '';
            }
        }

        // REMOVE DUPLICATES
        const unique = new Map();
        result.forEach(channel => {
            const key = channel.id || channel.name.toLowerCase();
            if (!unique.has(key)) {
                unique.set(key, channel);
            }
        });

        return Array.from(unique.values());
    }

    /* ==========================================
       LOAD PLAYLIST
    ========================================== */
    async function loadPlaylist() {
        setStatus('Φόρτωση playlist...', false);
        try {
            const response = await fetch(M3U_URL);
            if (!response.ok) throw new Error('HTTP ' + response.status);

            const text = await response.text();
            channels = parseM3U(text);

            if (!channels.length) throw new Error('Δεν βρέθηκαν κανάλια');

            renderChannels();

            let saved = null;
            try {
                saved = localStorage.getItem(STORAGE_KEY);
            } catch(e) {}

            let startIndex = 0;
            if (saved) {
                const found = channels.findIndex(channel => channel.url === saved);
                if (found >= 0) startIndex = found;
            }

            switchChannel(startIndex);

        } catch(error) {
            console.error('Playlist:', error);
            setStatus('❌ Σφάλμα φόρτωσης playlist', false);
            channelGrid.innerHTML = 'Δεν ήταν δυνατή η φόρτωση των καναλιών.';
        }
    }

    /* ==========================================
       CHANNEL LIST
    ========================================== */
    function renderChannels(filter = '') {
        channelGrid.innerHTML = '';
        const query = filter.toLowerCase().trim();
        let found = 0;

        channels.forEach((channel, index) => {
            if (query && !channel.name.toLowerCase().includes(query)) return;

            found++;
            const button = document.createElement('button');
            button.className = 'channel';
            if (index === currentIndex) button.classList.add('selected');

            let type = 'TV';
            if (channel.url.toLowerCase().includes('.mpd')) type = 'DASH';
            else if (channel.url.toLowerCase().includes('.m3u8')) type = 'HLS';

            button.innerHTML = channel.name + '<br><small>' + type + '</small>';
            button.addEventListener('click', () => {
                switchChannel(index);
                channelPanel.classList.remove('open');
            });

            channelGrid.appendChild(button);
        });

        if (!found) channelGrid.innerHTML = 'Δεν βρέθηκε κανάλι.';
    }

    /* ==========================================
       STOP PLAYER
    ========================================== */
    function stopPlayer() {
        if (hls) {
            try { hls.destroy(); } catch(e) {}
            hls = null;
        }
        if (dashPlayer) {
            try { dashPlayer.reset(); } catch(e) {}
            dashPlayer = null;
        }
        try {
            video.pause();
            video.removeAttribute('src');
            video.load();
        } catch(e) {}
    }

    /* ==========================================
       SWITCH CHANNEL (Με Fallback για CORS)
    ========================================== */
    function switchChannel(index) {
        if (!channels.length) return;

        currentIndex = index;
        const channel = channels[index];

        setStatus('Φόρτωση: ' + channel.name, false);

        try {
            localStorage.setItem(STORAGE_KEY, channel.url);
        } catch(e) {}

        renderChannels(search.value);
        stopPlayer();

        let originalUrl = channel.url;
        const lowerUrl = originalUrl.toLowerCase();
        let url = originalUrl; // Το URL που θα χρησιμοποιήσουμε τελικά

        /* =====================================
           DASH .MPD
        ===================================== */
        if (lowerUrl.includes('.mpd')) {
            setStatus('Φόρτωση DASH: ' + channel.name, false);
            try {
                dashPlayer = dashjs.MediaPlayer().create();
                dashPlayer.initialize(video, url, true);
                dashPlayer.on(dashjs.MediaPlayer.events.STREAM_INITIALIZED, function() {
                    setStatus('🟢 Ζωντανά: ' + channel.name + ' • DASH', true);
                });
                dashPlayer.on(dashjs.MediaPlayer.events.ERROR, function(error) {
                    console.error('DASH:', error);
                    setStatus('❌ Πρόβλημα DASH: ' + channel.name, false);
                });
                return;
            } catch(error) {
                console.error('DASH:', error);
                setStatus('❌ Πρόβλημα DASH: ' + channel.name, false);
                return;
            }
        }

        /* =====================================
           HLS .M3U8
        ===================================== */
        if (Hls.isSupported()) {
            setStatus('Φόρτωση HLS: ' + channel.name, false);

            hls = new Hls({
                enableWorker: true,
                lowLatencyMode: true,
                enableWebVTT: true
            });

            hls.loadSource(url);
            hls.attachMedia(video);

            hls.on(Hls.Events.MANIFEST_PARSED, function() {
                setStatus('🟢 Ζωντανά: ' + channel.name + ' • HLS', true);
                video.play().catch(() => {
                    setStatus(channel.name + ' • πάτησε ▶', true);
                });
            });

            // Εδώ γίνεται η "μαγική" διόρθωση για το CORS (π.χ. ANT1)
            hls.on(Hls.Events.ERROR, function(event, data) {
                console.error('HLS Error Details:', data.details, data);
                
                if (data.fatal) {
                    switch (data.type) {
                        case Hls.ErrorTypes.NETWORK_ERROR:
                            // Αν το URL δεν έχει ήδη proxy, δοκίμασε ξανά με CORS Proxy
                            if (!url.includes('corsproxy.io')) {
                                console.log('⚠️ Σφάλμα δικτύου (CORS). Εναλλαγή σε proxy...');
                                setStatus('⚠️ Απόπειρα μέσω Proxy: ' + channel.name, false);
                                url = "https://corsproxy.io/?" + encodeURIComponent(originalUrl);
                                hls.loadSource(url);
                            } else {
                                setStatus('❌ Πρόβλημα δικτύου (CORS): ' + channel.name, false);
                                hls.destroy();
                            }
                            break;
                        case Hls.ErrorTypes.MEDIA_ERROR:
                            console.log('Σφάλμα βίντεο. Προσπάθεια ανάκτησης...');
                            hls.recoverMediaError();
                            break;
                        default:
                            setStatus('❌ Πρόβλημα HLS: ' + channel.name, false);
                            hls.destroy();
                            break;
                    }
                }
            });
            return;
        }

        /* =====================================
           SAFARI / IPHONE (Native Player)
        ===================================== */
        if (video.canPlayType('application/vnd.apple.mpegurl')) {
            video.src = url;
            video.addEventListener('loadedmetadata', function() {
                setStatus('🟢 Ζωντανά: ' + channel.name, true);
                video.play().catch(() => {});
            }, { once: true });
            return;
        }

        setStatus('❌ Μη υποστηριζόμενη ροή.', false);
    }

    /* ==========================================
       BUTTON EVENTS
    ========================================== */
    channelsBtn.addEventListener('click', function() {
        channelPanel.classList.toggle('open');
    });

    search.addEventListener('input', function() {
        renderChannels(search.value);
    });

    restartBtn.addEventListener('click', function() {
        switchChannel(currentIndex);
    });

    /* ==========================================
       CAPTION DISPLAY (Live CC)
    ========================================== */
    function showCaption(text) {
        text = text.trim();
        if (!text) return;

        if (text.length > 100) {
            text = text.slice(-100);
        }

        const middle = Math.floor(text.length / 2);
        let split = text.lastIndexOf(' ', middle);

        if (split < 1) {
            split = text.indexOf(' ', middle);
        }

        if (split < 1) {
            caption1.textContent = text;
            caption2.textContent = '';
        } else {
            caption1.textContent = text.substring(0, split);
            caption2.textContent = text.substring(split + 1);
        }

        clearTimeout(captionTimer);
        captionTimer = setTimeout(function() {
            caption1.textContent = '';
            caption2.textContent = '';
        }, 10000);
    }

    function startCC() {
        const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        if (!SpeechRecognition) {
            alert('Ο browser δεν υποστηρίζει Live CC.');
            return;
        }

        recognition = new SpeechRecognition();
        recognition.lang = 'el-GR';
        recognition.continuous = true;
        recognition.interimResults = true;
        recognition.maxAlternatives = 1;

        recognition.onresult = function(event) {
            let text = '';
            for (let i = event.resultIndex; i < event.results.length; i++) {
                text += event.results[i][0].transcript + ' ';
            }
            showCaption(text);
        };

        recognition.onerror = function(event) {
            console.log('CC Error:', event.error);
            if (event.error === 'not-allowed') {
                alert('Χρειάζεται άδεια μικροφώνου για το Live CC.');
                stopCC();
            }
        };

        recognition.onend = function() {
            if (captionsOn) {
                try { recognition.start(); } catch(e) {}
            }
        };

        captionsOn = true;
        ccBtn.textContent = 'CC ✓';
        ccBtn.classList.add('active');
        ccBtn.title = 'Υπότιτλοι ανοικτοί';

        try { recognition.start(); } catch(e) {}

        if (channels.length) {
            setStatus('🟢 Live CC Ελληνικά • ' + channels[currentIndex].name, true);
        }
    }

    function stopCC() {
        captionsOn = false;
        ccBtn.textContent = 'CC';
        ccBtn.classList.remove('active');
        ccBtn.title = 'Υπότιτλοι κλειστοί';

        clearTimeout(captionTimer);
        caption1.textContent = '';
        caption2.textContent = '';

        if (recognition) {
            try { recognition.stop(); } catch(e) {}
            recognition = null;
        }

        if (channels.length) {
            setStatus('🟢 Ζωντανά: ' + channels[currentIndex].name, true);
        }
    }

    ccBtn.addEventListener('click', function() {
        if (captionsOn) {
            stopCC();
        } else {
            startCC();
        }
    });

    /* ==========================================
       FULLSCREEN
    ========================================== */
    fullscreenBtn.addEventListener('click', async function() {
        try {
            if (!document.fullscreenElement) {
                await videoWrapper.requestFullscreen();
            } else {
                await document.exitFullscreen();
            }
        } catch(error) {
            console.log('Fullscreen:', error);
        }
    });

    video.addEventListener('dblclick', function() {
        fullscreenBtn.click();
    });

    /* ==========================================
       START
    ========================================== */
    loadPlaylist();

</script>
</body>
</html>