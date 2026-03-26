FROM node:20-slim

# Install Chromium dependencies + ffmpeg + sqlite3 build tools + curl for audio
RUN apt-get update && apt-get install -y \
    chromium \
    fonts-liberation \
    libatk-bridge2.0-0 \
    libatk1.0-0 \
    libcups2 \
    libdbus-1-3 \
    libgdk-pixbuf2.0-0 \
    libnspr4 \
    libnss3 \
    libxcomposite1 \
    libxdamage1 \
    libxrandr2 \
    xdg-utils \
    ffmpeg \
    python3 \
    make \
    g++ \
    curl \
    --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*

# ── Built-in sports music ─────────────────────────────────────────────────────
# Kevin MacLeod - CC BY 3.0 (incompetech.com)
# Downloaded from chosic.com CDN — URLs verified March 2026.
# Uses curl -f so any 404/error causes an immediate build failure (no silent 0-byte files).
# The size check at the end double-confirms every file is valid before the image is committed.
RUN mkdir -p /audio && \
    curl -fsSL -A "Mozilla/5.0" \
      "https://www.chosic.com/wp-content/uploads/2022/06/Run-Amok(chosic.com).mp3" \
      -o /audio/02-run-amok.mp3 && \
    curl -fsSL -A "Mozilla/5.0" \
      "https://www.chosic.com/wp-content/uploads/2022/06/Sneaky-Snitch(chosic.com).mp3" \
      -o /audio/03-sneaky-snitch.mp3 && \
    curl -fsSL -A "Mozilla/5.0" \
      "https://www.chosic.com/wp-content/uploads/2022/10/Kevin-MacLeod-Investigations(chosic.com).mp3" \
      -o /audio/04-investigations.mp3 && \
    curl -fsSL -A "Mozilla/5.0" \
      "https://www.chosic.com/wp-content/uploads/2022/10/Scheming-Weasel-faster(chosic.com).mp3" \
      -o /audio/05-scheming-weasel.mp3 && \
    echo "=== Verifying audio files ===" && \
    for f in /audio/0*.mp3; do \
      size=$(stat -c%s "$f"); \
      echo "  $f = ${size} bytes"; \
      [ "$size" -gt 50000 ] || { echo "ABORT: $f is too small (${size} bytes) — download failed"; exit 1; }; \
    done && \
    printf '/audio/02-run-amok.mp3\n/audio/03-sneaky-snitch.mp3\n/audio/04-investigations.mp3\n/audio/05-scheming-weasel.mp3\n' \
      > /audio/playlist.m3u && \
    echo "=== Audio playlist ===" && cat /audio/playlist.m3u && \
    echo "=== Audio OK ==="

WORKDIR /app

COPY package.json .
RUN npm install --omit=dev

COPY manager.js .

ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true \
    PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium \
    PIPES_DIR=/tmp/pipes \
    HLS_DIR=/hls \
    MANAGER_PORT=3001 \
    DB_PATH=/config/scorestream.db

CMD ["node", "manager.js"]
