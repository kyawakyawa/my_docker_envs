FROM  rust:1.81-bullseye

RUN apt-get update && apt-get install -y libseccomp-dev && \
    cd / && git clone https://github.com/containers/youki.git  && \
    cd youki && git checkout v0.4.1 && cargo build --release && \
    cp target/release/youki /usr/local/bin/youki && \
    rm -rf /var/lib/apt/lists/*

CMD ["cp", "/usr/local/bin/youki", "/mnt/youki"]
