FROM ubuntu:22.04

# Prevent interactive prompts during build
ENV DEBIAN_FRONTEND=noninteractive

# Install Node.js and utilities
RUN apt-get update && apt-get install -y \
    curl \
    tar \
    unzip \
    ca-certificates \
    findutils \
    && curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/*

# Create directory for ITURHFProp
WORKDIR /opt/iturhfprop

# Download individual binary files from v14.3 release (latest)
RUN curl -L -o ITURHFProp "https://github.com/ITU-R-Study-Group-3/ITU-R-HF/releases/download/v14.3/ITURHFProp" \
    && curl -L -o libp533.so "https://github.com/ITU-R-Study-Group-3/ITU-R-HF/releases/download/v14.3/libp533.so" \
    && curl -L -o libp372.so "https://github.com/ITU-R-Study-Group-3/ITU-R-HF/releases/download/v14.3/libp372.so" \
    && chmod +x ITURHFProp

# Download source to get Data files - try ITU first, then GitHub source
RUN curl -L -o itu-hf.zip "https://www.itu.int/en/ITU-R/study-groups/rsg3/rwp3m/Software%20Products/ITU-R-HF_14.3.zip" 2>/dev/null || true \
    && if [ -f itu-hf.zip ] && [ $(stat -c%s itu-hf.zip) -gt 1000000 ]; then \
        echo "=== Using ITU download ===" && unzip -o itu-hf.zip; \
    else \
        echo "=== ITU download failed, using GitHub source ===" && \
        curl -L -o source.tar.gz "https://github.com/ITU-R-Study-Group-3/ITU-R-HF/archive/refs/tags/v14.3.tar.gz" && \
        tar -xzf source.tar.gz; \
    fi \
    && echo "=== Top level contents ===" \
    && ls -la \
    && echo "=== All directories ===" \
    && find . -type d | head -30 \
    && echo "=== All .bin files ===" \
    && find . -name "*.bin" \
    && echo "=== All .ant files ===" \
    && find . -name "*.ant" \
    && echo "=== All COEFF files ===" \
    && find . -name "COEFF*.BIN" | head -5

# Copy Data directory from wherever it is, or build it from individual files
RUN mkdir -p /opt/iturhfprop/Data && \
    DATA_DIR=$(find . -type d -name "Data" | grep -v "\.git" | head -1) && \
    if [ -n "$DATA_DIR" ] && [ -d "$DATA_DIR" ]; then \
        echo "Found Data at: $DATA_DIR" && \
        cp -r "$DATA_DIR"/* /opt/iturhfprop/Data/ 2>/dev/null || true; \
    fi && \
    echo "=== Copying all data files to Data/ ===" && \
    find . -name "COEFF*.BIN" -exec cp {} /opt/iturhfprop/Data/ \; 2>/dev/null || true && \
    find . -name "COEFF*.txt" -exec cp {} /opt/iturhfprop/Data/ \; 2>/dev/null || true && \
    find . -name "ionos*.bin" -exec cp {} /opt/iturhfprop/Data/ \; 2>/dev/null || true && \
    find . -name "*.ant" -exec cp {} /opt/iturhfprop/Data/ \; 2>/dev/null || true && \
    find . -name "P1239*.txt" -exec cp {} /opt/iturhfprop/Data/ \; 2>/dev/null || true && \
    find . -name "P372*.txt" -exec cp {} /opt/iturhfprop/Data/ \; 2>/dev/null || true && \
    echo "=== Final Data contents ===" && ls -la /opt/iturhfprop/Data/ | head -30

# Cleanup downloaded archives
RUN rm -rf ITU-R-HF* itu-hf.zip source.tar.gz P533 *.exe *.dll 2>/dev/null || true

# Set library path so ITURHFProp can find shared libs
ENV LD_LIBRARY_PATH=/opt/iturhfprop:$LD_LIBRARY_PATH

# Verify installation - show all files in Data
RUN echo "=== Binary files ===" && ls -la /opt/iturhfprop/*.so /opt/iturhfprop/ITURHFProp \
    && echo "=== Data directory contents ===" && ls -la /opt/iturhfprop/Data/ 2>/dev/null || echo "Data directory not found or empty"

# Set up the API service
WORKDIR /app
COPY package.json ./
RUN npm install --production

COPY server.js ./

# Environment
ENV PORT=3000
ENV ITURHFPROP_PATH=/opt/iturhfprop/ITURHFProp
ENV ITURHFPROP_DATA=/opt/iturhfprop

EXPOSE 3000

CMD ["node", "server.js"]
