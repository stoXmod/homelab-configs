FROM python:3.11-slim-bookworm

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    gstreamer1.0-plugins-good \
    gstreamer1.0-plugins-ugly \
    gstreamer1.0-plugins-bad \
    gstreamer1.0-alsa \
    gstreamer1.0-tools \
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

# Create directories
RUN mkdir -p /config /var/lib/mopidy/local /var/lib/mopidy/media /var/lib/mopidy/cache /var/lib/mopidy/data

COPY mopidy.conf /config/mopidy.conf

EXPOSE 6680 6600

CMD ["mopidy", "--config", "/config/mopidy.conf"]
