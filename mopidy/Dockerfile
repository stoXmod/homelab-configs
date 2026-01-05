<<<<<<< Updated upstream
FROM debian:bookworm-slim

# Install Mopidy and dependencies via apt (includes proper GObject/GStreamer bindings)
RUN apt-get update && apt-get install -y --no-install-recommends \
    gnupg2 \
    curl \
    ca-certificates \
    python3 \
    python3-pip \
    python3-gst-1.0 \
    python3-gi \
    gir1.2-gstreamer-1.0 \
    gir1.2-gst-plugins-base-1.0 \
=======
FROM python:3.11-slim-bookworm

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
>>>>>>> Stashed changes
    gstreamer1.0-plugins-good \
    gstreamer1.0-plugins-ugly \
    gstreamer1.0-plugins-bad \
    gstreamer1.0-alsa \
    gstreamer1.0-tools \
<<<<<<< Updated upstream
    mopidy \
    && rm -rf /var/lib/apt/lists/*

# Install only Mopidy extensions via pip (avoid reinstalling mopidy itself)
# ytmusicapi is needed for Mopidy-YTMusic
RUN pip3 install --break-system-packages --no-cache-dir \
    ytmusicapi \
    Mopidy-Iris \
    Mopidy-YTMusic \
    Mopidy-Local \
    && pip3 uninstall -y mopidy 2>/dev/null || true
=======
    python3-gst-1.0 \
    libgirepository1.0-dev \
    gcc \
    pkg-config \
    libcairo2-dev \
    && rm -rf /var/lib/apt/lists/*

# Install Mopidy and extensions
RUN pip install --no-cache-dir \
    mopidy \
    Mopidy-Iris \
    Mopidy-YTMusic \
    Mopidy-Local
>>>>>>> Stashed changes

# Create directories
RUN mkdir -p /config /var/lib/mopidy/local /var/lib/mopidy/media /var/lib/mopidy/cache /var/lib/mopidy/data

COPY mopidy.conf /config/mopidy.conf

EXPOSE 6680 6600

<<<<<<< Updated upstream
# Use the system-installed mopidy (from apt)
CMD ["/usr/bin/mopidy", "--config", "/config/mopidy.conf"]
=======
CMD ["mopidy", "--config", "/config/mopidy.conf"]
>>>>>>> Stashed changes
