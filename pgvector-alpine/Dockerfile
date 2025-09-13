# Use the official PostgreSQL 17.6 Alpine image as the base
FROM postgres:17.6-alpine3.22

# Install build dependencies, clone pgvector, build and install it, then clean up
RUN echo 'https://mirrors.tuna.tsinghua.edu.cn/alpine/v3.22/main' > /etc/apk/repositories && \
    echo 'https://mirrors.tuna.tsinghua.edu.cn/alpine/v3.22/community' >> /etc/apk/repositories && \
    apk update && \
    apk add --no-cache \
    build-base \
    clang \
    clang19 \
    git \
    llvm \
    llvm19 \
    postgresql-dev \
    && git clone https://github.com/pgvector/pgvector.git /tmp/pgvector \
    && cd /tmp/pgvector \
    && make NO_LTO=1 \
    && make install \
    && cd / \
    && rm -rf /tmp/pgvector \
    && apk del build-base clang clang19 git llvm llvm19 postgresql-dev \
    && rm -rf /var/cache/apk/*

# Set the default command to run PostgreSQL
CMD ["postgres"]
