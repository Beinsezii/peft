# Builds GPU docker image of PyTorch
# Uses multi-staged approach to reduce size
# Stage 1
# Use base conda image to reduce time
FROM continuumio/miniconda3:latest AS compile-image
# Specify py version
ENV PYTHON_VERSION=3.11
# Install apt libs - copied from https://github.com/huggingface/accelerate/blob/main/docker/accelerate-gpu/Dockerfile
RUN apt-get update && \
    apt-get install -y curl git wget software-properties-common git-lfs && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists*

# Install audio-related libraries 
RUN apt-get update && \
    apt install -y ffmpeg

RUN apt install -y libsndfile1-dev
RUN git lfs install

# Create our conda env - copied from https://github.com/huggingface/accelerate/blob/main/docker/accelerate-gpu/Dockerfile
RUN conda create --name peft python=${PYTHON_VERSION} ipython jupyter pip
RUN python3 -m pip install --no-cache-dir --upgrade pip

# Below is copied from https://github.com/huggingface/accelerate/blob/main/docker/accelerate-gpu/Dockerfile
# We don't install pytorch here yet since CUDA isn't available
# instead we use the direct torch wheel
ENV PATH /opt/conda/envs/peft/bin:$PATH
# Activate our bash shell
RUN chsh -s /bin/bash
SHELL ["/bin/bash", "-c"]

# Stage 2
FROM nvidia/cuda:12.6.3-devel-ubuntu22.04 AS build-image
COPY --from=compile-image /opt/conda /opt/conda
ENV PATH /opt/conda/bin:$PATH

RUN chsh -s /bin/bash
SHELL ["/bin/bash", "-c"]

# Install apt libs
RUN apt-get update && \
    apt-get install -y curl git wget cmake && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists*

# Activate the conda env and install transformers + accelerate from latest pypi
# Also clone BNB and build it from source.
RUN source activate peft && \
    python3 -m pip install -U --no-cache-dir \
    librosa \
    "soundfile>=0.12.1" \
    scipy \
    transformers \
    accelerate \
    peft \
    optimum \
    auto-gptq && \
    git clone https://github.com/bitsandbytes-foundation/bitsandbytes && cd bitsandbytes && \
    cmake -B . -DCOMPUTE_BACKEND=cuda -S . && \
    cmake --build . && \
    pip install -e . && \ 
    pip freeze | grep bitsandbytes

RUN echo "source activate peft" >> ~/.profile

# Activate the virtualenv
CMD ["/bin/bash"]
