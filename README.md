<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Video Player</title>

    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Outfit:wght@300;400;600&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

    <style>
        :root {
            --bg-start: #0a0a0a;
            --bg-end: #000000;
            --surface: rgba(15, 15, 15, 0.95);
            --surface-border: rgba(212, 175, 55, 0.32);
            --text-main: #f5f5f5;
            --text-muted: #b8b8b8;
            --accent: #d4af37;
            --accent-strong: #c9a227;
            --accent-soft: rgba(212, 175, 55, 0.28);
            --radius-xl: 30px;
            --radius-lg: 18px;
            --shadow-main: 0 34px 96px rgba(0, 0, 0, 0.68);
        }

        * { box-sizing: border-box; }

        html, body {
            margin: 0;
            min-height: 100%;
            font-family: 'Outfit', sans-serif;
            color: var(--text-main);
            color-scheme: dark;
            background: #000000;
        }

        .container {
            width: 100%;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: flex-start;
            padding: 20px 24px 40px;
        }

        .video-wrapper {
            width: min(980px, 100%);
            padding: 0;
            margin: 0 auto;
            display: flex;
            flex-direction: column;
            gap: 12px;
        }

        /* ── Button Group ── */
        .button-group {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 10px;
            margin: 0;
            padding: 2px 2px 10px;
        }

        .folder-btn {
            text-decoration: none;
            color: var(--text-main);
            padding: 9px 14px;
            border-radius: 12px;
            border: 1px solid rgba(212, 175, 55, 0.34);
            background: linear-gradient(135deg, rgba(20,20,20,0.92), rgba(10,10,10,0.96));
            display: inline-flex;
            align-items: center;
            gap: 8px;
            font-size: 0.88rem;
            font-weight: 600;
            letter-spacing: 0.2px;
            transition: transform .2s ease, border-color .2s ease, background .2s ease, box-shadow .2s ease;
        }

        .folder-btn i { color: var(--accent); }

        .folder-btn:hover {
            transform: translateY(-2px);
            border-color: rgba(212, 175, 55, 0.65);
            background: linear-gradient(135deg, rgba(30,25,15,0.96), rgba(15,15,15,0.98));
            box-shadow: 0 12px 24px rgba(212, 175, 55, 0.34);
        }

        /* ── Video Shell ── */
        .video-shell {
            position: relative;
            border-radius: var(--radius-lg);
            overflow: hidden;
            border: 1px solid rgba(212, 175, 55, 0.24);
            background: #000;
            box-shadow: 0 20px 40px rgba(0,0,0,0.45), 0 0 0 1px rgba(212,175,55,0.1) inset;
        }

        video {
            width: 100%;
            max-height: 74vh;
            display: block;
            outline: none;
            background: #000;
        }

        /* ── Ad Overlay ── */
        .ad-overlay {
            display: none;
            position: absolute;
            inset: 0;
            z-index: 10;
            background: transparent;
            cursor: default;
        }

        .ad-overlay.active { display: block; }

        /* ── Loading State ── */
        .loading-screen {
            display: none;
            position: absolute;
            inset: 0;
            z-index: 20;
            background: rgba(0,0,0,0.85);
            justify-content: center;
            align-items: center;
            flex-direction: column;
            gap: 14px;
            border-radius: var(--radius-lg);
        }

        .loading-screen.active { display: flex; }

        .spinner {
            width: 48px;
            height: 48px;
            border: 3px solid rgba(212,175,55,0.2);
            border-top-color: var(--accent);
            border-radius: 50%;
            animation: spin 0.8s linear infinite;
        }

        @keyframes spin {
            to { transform: rotate(360deg); }
        }

        .loading-text {
            color: var(--text-muted);
            font-size: 0.88rem;
            font-weight: 400;
            letter-spacing: 0.3px;
        }

        /* ── Error State ── */
        .error-screen {
            display: none;
            position: absolute;
            inset: 0;
            z-index: 20;
            background: rgba(0,0,0,0.92);
            justify-content: center;
            align-items: center;
            flex-direction: column;
            gap: 14px;
            border-radius: var(--radius-lg);
            text-align: center;
            padding: 24px;
        }

        .error-screen.active { display: flex; }

        .error-icon {
            font-size: 2.4rem;
            color: var(--accent);
            opacity: 0.8;
        }

        .error-title {
            font-size: 1rem;
            font-weight: 600;
            color: var(--text-main);
            margin: 0;
        }

        .error-sub {
            font-size: 0.82rem;
            color: var(--text-muted);
            margin: 0;
        }

        /* ── Ticker ── */
        .notice-ticker {
            position: relative;
            overflow: hidden;
            margin-top: 10px;
            border-radius: 14px;
            border: 1px solid rgba(212, 175, 55, 0.32);
            background: linear-gradient(120deg, rgba(25,20,10,0.6), rgba(10,10,10,0.9));
            box-shadow: inset 0 0 0 1px rgba(255,255,255,0.04);
        }

        .notice-track {
            display: flex;
            align-items: center;
            width: max-content;
            gap: 36px;
            padding: 10px 0;
            animation: ticker 14s linear infinite;
        }

        .notice-text {
            display: inline-flex;
            align-items: center;
            gap: 8px;
            padding: 0 14px;
            white-space: nowrap;
            font-size: 0.9rem;
            font-weight: 600;
            color: #e8e8e8;
            text-shadow: 0 0 14px rgba(212, 175, 55, 0.3);
        }

        .notice-text i { color: var(--accent); }

        @keyframes ticker {
            from { transform: translateX(0); }
            to   { transform: translateX(-50%); }
        }

        /* ── Floating Telegram ── */
        .telegram-floating-btn {
            position: fixed;
            right: 20px;
            bottom: 20px;
            z-index: 1000;
            text-decoration: none;
            color: #000;
            font-weight: 600;
            padding: 11px 20px;
            border-radius: 999px;
            border: 1px solid rgba(255,255,255,0.24);
            background: linear-gradient(135deg, var(--accent), var(--accent-strong));
            box-shadow: 0 10px 26px var(--accent-soft);
            display: inline-flex;
            align-items: center;
            gap: 8px;
            transition: transform .2s ease, box-shadow .2s ease;
        }

        .telegram-floating-btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 14px 30px rgba(212,175,55,0.4);
        }

        @media (max-width: 768px) {
            .container { padding: 12px 12px 28px; align-items: flex-start; }
            .video-wrapper { margin-top: 0; gap: 10px; padding: 0; }
            .button-group { gap: 8px; padding: 0; }
            .telegram-floating-btn { bottom: 14px; right: 14px; padding: 10px 16px; font-size: .9rem; }
        }
    </style>
</head>

<body>

    <div class="container">
        <main class="video-wrapper">

            <div class="button-group">
                <a href="https://crn77.com/4/10413165" class="folder-btn">
                    <i class="fas fa-folder"></i> SMP
                </a>
                <a href="https://crn77.com/4/10413165" class="folder-btn">
                    <i class="fas fa-folder"></i> SMA
                </a>
                <a href="https://crn77.com/4/10413165" class="folder-btn">
                    <i class="fas fa-folder"></i> HIJAB
                </a>
            </div>

            <section class="video-shell">
                <video id="videoPlayer" controls autoplay playsinline>
                    Your browser does not support the video tag.
                </video>

                <!-- Ad overlay -->
                <div class="ad-overlay" id="adOverlay"></div>

                <!-- Loading overlay -->
                <div class="loading-screen active" id="loadingScreen">
                    <div class="spinner"></div>
                    <span class="loading-text">Memuat video...</span>
                </div>

                <!-- Error / redirect overlay -->
                <div class="error-screen" id="errorScreen">
                    <div class="error-icon"><i class="fas fa-circle-exclamation"></i></div>
                    <p class="error-title">Video tidak dapat diputar</p>
                    <p class="error-sub">Mengalihkan ke sumber alternatif…</p>
                </div>
            </section>

            <section class="notice-ticker">
                <div class="notice-track">
                    <span class="notice-text"><i class="fas fa-heart"></i> Jangan lupa ikuti channel sayang</span>
                    <span class="notice-text"><i class="fas fa-heart"></i> Jangan lupa ikuti channel sayang</span>
                    <span class="notice-text"><i class="fas fa-heart"></i> Jangan lupa ikuti channel sayang</span>
                    <span class="notice-text"><i class="fas fa-heart"></i> Jangan lupa ikuti channel sayang</span>
                </div>
            </section>

        </main>
    </div>

    <a href="https://t.me/+H2RT7T_EIgw4NDg1" target="_blank" class="telegram-floating-btn">
        <i class="fab fa-telegram"></i> Join Now
    </a>

    <script>
        const FALLBACK_URL  = "https://crn77.com/4/10413165";
        const TIMEOUT_MS    = 8000;   // max wait before redirect if video never plays

        const path     = window.location.pathname.replace(/^\//, '').trim();
        const videoUrl = `https://cdn2.videy.co/${path}.mp4`;

        const video         = document.getElementById("videoPlayer");
        const overlay       = document.getElementById("adOverlay");
        const loadingScreen = document.getElementById("loadingScreen");
        const errorScreen   = document.getElementById("errorScreen");

        /* ── State ── */
        let playbackStarted  = false;
        let errorTriggered   = false;
        let timeoutHandle    = null;

        /* ── Helpers ── */
        function hideLoading() {
            loadingScreen.classList.remove('active');
        }

        function triggerFallback() {
            if (errorTriggered) return;
            errorTriggered = true;

            clearTimeout(timeoutHandle);
            hideLoading();
            errorScreen.classList.add('active');

            // Langsung redirect tanpa countdown
            window.location.href = FALLBACK_URL;
        }

        /* ── Load video ── */
        video.src = videoUrl;
        video.load();

        /* ── Events: hide loading when video is ready ── */
        video.addEventListener('canplay', () => {
            hideLoading();
        });

        video.addEventListener('playing', () => {
            playbackStarted = true;
            hideLoading();
            clearTimeout(timeoutHandle);
        });

        /* ── Error: immediate fallback ── */
        video.addEventListener('error', () => {
            triggerFallback();
        });

        /* ── Stall detection: if nothing plays within TIMEOUT_MS ── */
        timeoutHandle = setTimeout(() => {
            if (!playbackStarted) {
                triggerFallback();
            }
        }, TIMEOUT_MS);

        /* ── Extra: if video stalls mid-playback for too long ── */
        let stallTimer = null;
        video.addEventListener('waiting', () => {
            stallTimer = setTimeout(() => {
                if (!video.ended) triggerFallback();
            }, 12000); // 12s stall after initial play → fallback
        });
        video.addEventListener('playing', () => {
            clearTimeout(stallTimer);
        });

        /* ════════════════════════════════
           ADS
        ════════════════════════════════ */
        const ads = [
            { url: "https://crn77.com/4/10413165",  triggerType: 'play', triggered: false },
            { url: "https://crn77.com/4/10413165",  triggerType: 'time', triggerAt: 10, triggered: false },
            { url: "https://crn77.com/4/10413165", triggerType: 'time', triggerAt: 30, triggered: false },
            { url: "https://crn77.com/4/10413165", triggerType: 'time', triggerAt: 50, triggered: false },
        ];

        function showAdOverlay(adUrl) {
            overlay.classList.add('active');
            window.open(adUrl, '_blank');
            setTimeout(() => overlay.classList.remove('active'), 5000);
        }

        video.addEventListener('play', () => {
            const ad = ads.find(a => a.triggerType === 'play' && !a.triggered);
            if (ad) {
                ad.triggered = true;
                setTimeout(() => showAdOverlay(ad.url), 300);
            }
        }, { once: true });

        video.addEventListener('timeupdate', () => {
            const t = video.currentTime;
            ads.forEach(ad => {
                if (ad.triggerType === 'time' && !ad.triggered && t >= ad.triggerAt) {
                    ad.triggered = true;
                    showAdOverlay(ad.url);
                }
            });
        });
    </script>

    <script defer
        src="https://static.cloudflareinsights.com/beacon.min.js/v8c78df7c7c0f484497ecbca7046644da1771523124516"
        integrity="sha512-8DS7rgIrAmghBFwoOTujcf6D9rXvH8xm8JQ1Ja01h9QX8EzXldiszufYa4IFfKdLUKTTrnSFXLDkUEOTrZQ8Qg=="
        data-cf-beacon='{"version":"2024.11.0","token":"206a3bb33ddf4ff4bb13a4bd5b4dad7e","r":1}'
        crossorigin="anonymous">
    </script>

</body>
</html>
