# syntax=docker/dockerfile:experimental
FROM debian:bookworm-slim as builder

RUN apt-get update && apt-get install -y curl clang openssl libssl-dev gcc g++ \
    pkg-config build-essential libclang-dev && \
    rm -rf /var/lib/apt/lists/*

ENV RUSTUP_HOME=/usr/local/rustup \
    CARGO_HOME=/usr/local/cargo \
    PATH=/usr/local/cargo/bin:$PATH

RUN curl https://sh.rustup.rs -sSf | bash -s -- -y && \
    rustup install nightly-2023-08-21 && \
    rustup default nightly-2023-08-21

WORKDIR /usr/src/zksync
COPY . .

RUN cargo build --release

FROM debian:bookworm-slim

RUN apt-get update && apt-get install -y curl libpq5 ca-certificates wget python3 && rm -rf /var/lib/apt/lists/*

# install zksolc 1.3.x
RUN skip_versions="v1.3.12 v1.3.15" && \
    for VERSION in $(seq -f "v1.3.%g" 0 18); do \
    if echo " $skip_versions " | grep -q -w " $VERSION "; then \
    continue; \
    fi; \
    mkdir -p /etc/zksolc-bin/$VERSION && \
    wget https://github.com/matter-labs/zksolc-bin/raw/main/linux-amd64/zksolc-linux-amd64-musl-$VERSION -O /etc/zksolc-bin/$VERSION/zksolc && \
    chmod +x /etc/zksolc-bin/$VERSION/zksolc; \
    done

# install zkvyper 1.3.x
RUN for VERSION in $(seq -f "v1.3.%g" 9 13); do \
    mkdir -p /etc/zkvyper-bin/$VERSION && \
    wget https://github.com/matter-labs/zkvyper-bin/raw/main/linux-amd64/zkvyper-linux-amd64-musl-$VERSION -O /etc/zkvyper-bin/$VERSION/zkvyper && \
    chmod +x /etc/zkvyper-bin/$VERSION/zkvyper; \
    done

# install solc
COPY docker/contract-verifier/install-all-solc.sh install-all-solc.sh
RUN sh ./install-all-solc.sh

# install vyper
RUN mkdir -p /etc/vyper-bin/0.3.3 \
    && wget -O vyper0.3.3 https://github.com/vyperlang/vyper/releases/download/v0.3.3/vyper.0.3.3%2Bcommit.48e326f0.linux \
    && mv vyper0.3.3 /etc/vyper-bin/0.3.3/vyper \
    && chmod +x /etc/vyper-bin/0.3.3/vyper
RUN mkdir -p /etc/vyper-bin/0.3.9 \
    && wget -O vyper0.3.9 https://github.com/vyperlang/vyper/releases/download/v0.3.9/vyper.0.3.9%2Bcommit.66b96705.linux \
    && mv vyper0.3.9 /etc/vyper-bin/0.3.9/vyper \
    && chmod +x /etc/vyper-bin/0.3.9/vyper
RUN mkdir -p /etc/vyper-bin/0.3.10 \
    && wget -O vyper0.3.10 https://github.com/vyperlang/vyper/releases/download/v0.3.10/vyper.0.3.10%2Bcommit.91361694.linux \
    && mv vyper0.3.10 /etc/vyper-bin/0.3.10/vyper \
    && chmod +x /etc/vyper-bin/0.3.10/vyper

COPY --from=builder /usr/src/zksync/target/release/zksync_contract_verifier /usr/bin/
COPY contracts/system-contracts/bootloader/build/artifacts/ /contracts/system-contracts/bootloader/build/artifacts/
COPY contracts/system-contracts/artifacts-zk /contracts/system-contracts/artifacts-zk

# CMD tail -f /dev/null
ENTRYPOINT ["zksync_contract_verifier"]
