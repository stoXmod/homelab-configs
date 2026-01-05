FROM ghcr.io/mopidy/mopidy:latest

RUN pip install \
    Mopidy-Iris \
    Mopidy-YTMusic \
    Mopidy-Mobile

COPY mopidy.conf /config/mopidy.conf
COPY ytmusic_auth.json /config/ytmusic_auth.json
