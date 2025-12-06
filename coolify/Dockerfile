# ====================================================================
# WHISPER-WEBUI DOCKERFILE FOR COOLIFY
# ====================================================================
# Оптимизирован для:
# - NVIDIA RTX 3080 (10GB VRAM)
# - CUDA 12.6 (совместим с 12.9 драйвером)
# - Русский язык (whisper-ct2-podlodka-turbo)
# - torchaudio 2.8.x (фикс pyannote совместимости)
# ====================================================================

# Stage 1: Builder
FROM nvidia/cuda:12.6.3-cudnn-runtime-ubuntu22.04 AS builder

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y \
        curl \
        git \
        python3 \
        python3-pip \
        python3-venv \
    && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/* \
    && mkdir -p /Whisper-WebUI

WORKDIR /Whisper-WebUI

# Копируем requirements и устанавливаем зависимости
COPY requirements.txt .

RUN python3 -m venv venv && \
    . venv/bin/activate && \
    pip install --upgrade pip wheel setuptools && \
    pip install -U -r requirements.txt

# Stage 2: Runtime
FROM nvidia/cuda:12.6.3-cudnn-runtime-ubuntu22.04 AS runtime

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y \
        curl \
        ffmpeg \
        python3 \
        libsndfile1 \
    && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*

WORKDIR /Whisper-WebUI

# Копируем код приложения
COPY . .

# Копируем venv из builder
COPY --from=builder /Whisper-WebUI/venv /Whisper-WebUI/venv

# Копируем кастомные конфиги для русского языка
COPY coolify/configs/default_parameters.yaml /Whisper-WebUI/configs/default_parameters.yaml

# Создаём директории для моделей и выходных файлов
RUN mkdir -p /Whisper-WebUI/models \
             /Whisper-WebUI/outputs \
             /Whisper-WebUI/configs

# Volumes
VOLUME ["/Whisper-WebUI/models"]
VOLUME ["/Whisper-WebUI/outputs"]
VOLUME ["/Whisper-WebUI/configs"]

# Environment
ENV PATH="/Whisper-WebUI/venv/bin:$PATH"
ENV LD_LIBRARY_PATH="/Whisper-WebUI/venv/lib/python3.10/site-packages/nvidia/cublas/lib:/Whisper-WebUI/venv/lib/python3.10/site-packages/nvidia/cudnn/lib"
ENV PYTHONUNBUFFERED=1
ENV GRADIO_SERVER_NAME=0.0.0.0
ENV GRADIO_SERVER_PORT=7860

# Healthcheck
HEALTHCHECK --interval=30s --timeout=15s --start-period=300s --retries=5 \
    CMD curl -sf http://localhost:7860/ || exit 1

# Default entrypoint
ENTRYPOINT ["python", "app.py", "--server_port", "7860", "--server_name", "0.0.0.0"]
