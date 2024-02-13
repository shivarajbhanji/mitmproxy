FROM python:3.11-bullseye as wheelbuilder

ARG MITMPROXY_WHEEL
COPY $MITMPROXY_WHEEL /wheels/
ENV RUSTUP_HOME=/usr/local/rustup \
    CARGO_HOME=/usr/local/cargo \
    PATH=/usr/local/cargo/bin:$PATH \
    RUSTUP_INIT_SKIP_PATH_CHECK=true \
    RUST_BACKTRACE=1 \
    CARGO_PROFILE_RELEASE_BUILD_OVERRIDE_DEBUG=true
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
RUN apt-get update \
    && apt-get install -y musl-dev \
    && rm -rf /var/lib/apt/lists/*
RUN pip install wheel && pip wheel --wheel-dir /wheels /wheels/${MITMPROXY_WHEEL}

FROM python:3.11-slim-bullseye

RUN useradd -mU mitmproxy
RUN apt-get update \
    && apt-get install -y --no-install-recommends gosu nano \
    && rm -rf /var/lib/apt/lists/*

COPY --from=wheelbuilder /wheels /wheels
RUN pip install --no-index --find-links=/wheels mitmproxy
RUN rm -rf /wheels

VOLUME /home/mitmproxy/.mitmproxy

COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 8080 8081

CMD ["mitmproxy"]
