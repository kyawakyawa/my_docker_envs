FROM gcc:14.2.0-bookworm

RUN cd /  && git clone https://passt.top/passt && \
    cd passt && \
    git checkout 2024_09_06.6b38f07 && \
    make && \
    cp passt /usr/local/bin/pasta

CMD ["cp", "/usr/local/bin/pasta", "/mnt"]
