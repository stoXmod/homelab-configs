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
    gstreamer1.0-plugins-good \
    gstreamer1.0-plugins-ugly \
    gstreamer1.0-plugins-bad \
    gstreamer1.0-alsa \
    gstreamer1.0-tools \
    mopidy \
    && rm -rf /var/lib/apt/lists/*

# Install Mopidy extensions via pip (use --break-system-packages for Debian 12)
RUN pip3 install --break-system-packages --no-cache-dir \
    Mopidy-Iris \
    Mopidy-YTMusic \
    Mopidy-Local

# Create directories
RUN mkdir -p /config /var/lib/mopidy/local /var/lib/mopidy/media /var/lib/mopidy/cache /var/lib/mopidy/data

COPY mopidy.conf /config/mopidy.conf

EXPOSE 6680 6600

CMD ["mopidy", "--config", "/config/mopidy.conf"]
