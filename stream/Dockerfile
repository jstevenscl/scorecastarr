FROM node:20-slim

# Install Chromium dependencies + ffmpeg + sqlite3 build tools + wget for audio
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
    wget \
    --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*

# ── Built-in sports music ─────────────────────────────────────────────────────
# Kevin MacLeod - CC-BY license (incompetech.com)
# Downloaded from Internet Archive at build time — baked into the image.
# Uses ; not && so a failed download doesn't abort the build.
# Falls back to a 60s silent mp3 if all downloads fail (no internet at build time).
RUN mkdir -p /audio && \
    wget -q --user-agent="Mozilla/5.0" \
      "https://archive.org/download/incompetech-all-the-music-2020/Run%20Amok.mp3" \
      -O /audio/01-run-amok.mp3 ; \
    wget -q --user-agent="Mozilla/5.0" \
      "https://archive.org/download/incompetech-all-the-music-2020/Sport%20of%20Kings.mp3" \
      -O /audio/02-sport-of-kings.mp3 ; \
    wget -q --user-agent="Mozilla/5.0" \
      "https://archive.org/download/incompetech-all-the-music-2020/Pump.mp3" \
      -O /audio/03-pump.mp3 ; \
    wget -q --user-agent="Mozilla/5.0" \
      "https://archive.org/download/incompetech-all-the-music-2020/Hep%20Cats.mp3" \
      -O /audio/04-hep-cats.mp3 ; \
    TRACKS=$(find /audio -name '*.mp3' -size +0c 2>/dev/null | wc -l) ; \
    if [ "$TRACKS" -eq "0" ]; then \
      echo "No tracks downloaded — generating silent fallback" ; \
      ffmpeg -f lavfi -i anullsrc=r=44100:cl=stereo -t 60 \
        -c:a libmp3lame -b:a 128k /audio/01-fallback.mp3 -y ; \
    fi ; \
    printf '#EXTM3U\n' > /audio/playlist.m3u && \
    find /audio -name '*.mp3' -size +0c | sort >> /audio/playlist.m3u && \
    echo "Audio playlist ready:" && cat /audio/playlist.m3u

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
