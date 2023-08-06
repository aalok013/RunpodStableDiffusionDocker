FROM nvidia/cuda:11.8.0-devel-ubuntu22.04

ARG WEBUI_VERSION=v1.5.1
ARG DREAMBOOTH_VERSION=1.0.14

ENV DEBIAN_FRONTEND noninteractive
ENV SHELL=/bin/bash
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib/x86_64-linux-gnu
ENV PATH="/workspace/venv/bin:$PATH"
ENV TORCH_COMMAND="pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118"

WORKDIR /workspace

# Set up shell and update packages
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Copy the Python dependencies
COPY requirements*.txt .
COPY cache-sd-model.py install-automatic.py ./

# Package installation and setup
RUN apt update --yes && \
    apt upgrade --yes && \
    apt install --yes --no-install-recommends \
        git openssh-server libglib2.0-0 libsm6 libgl1 libxrender1 libxext6 ffmpeg wget curl psmisc rsync vim nginx \
        pkg-config libffi-dev libcairo2 libcairo2-dev libgoogle-perftools4 libtcmalloc-minimal4 apt-transport-https \
        software-properties-common ca-certificates

RUN update-ca-certificates && \
    add-apt-repository ppa:deadsnakes/ppa && \
    apt install python3.10-dev python3.10-venv -y --no-install-recommends && \
    ln -s /usr/bin/python3.10 /usr/bin/python && \
    rm /usr/bin/python3 && \
    ln -s /usr/bin/python3.10 /usr/bin/python3 && \
    curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py && \
    python get-pip.py && \
    pip install -U --no-cache-dir pip

RUN python -m venv /workspace/venv && \
    export PATH="/workspace/venv/bin:$PATH" && \
    pip install -U --no-cache-dir jupyterlab jupyterlab_widgets ipykernel ipywidgets

RUN git clone https://github.com/aalok013/stable-diffusion-webui.git

RUN mv /workspace/install-automatic.py /workspace/stable-diffusion-webui/
RUN mv /workspace/requirements.txt ./requirements.txt
RUN mv /workspace/requirements_versions.txt ./requirements_versions.txt

RUN python -m install-automatic --skip-torch-cuda-test


RUN cd /workspace/stable-diffusion-webui && \
    git clone https://github.com/deforum-art/sd-webui-deforum extensions/deforum && \
    cd extensions/deforum && \
    pip install -r requirements.txt

RUN cd /workspace/stable-diffusion-webui && \
    git clone https://github.com/Mikubill/sd-webui-controlnet.git extensions/sd-webui-controlnet && \
    cd extensions/sd-webui-controlnet && \
    pip install -r requirements.txt

RUN cd /workspace/stable-diffusion-webui && \
    git clone https://github.com/thomasasfk/sd-webui-aspect-ratio-helper.git extensions/sd-webui-aspect-ratio-helper && \
    cd extensions/sd-webui-aspect-ratio-helper && \
    pip install -r requirements.txt

RUN cd /workspace/stable-diffusion-webui && \
    git clone https://github.com/DominikDoom/a1111-sd-webui-tagcomplete.git extensions/a1111-sd-webui-tagcomplete.git && \
    cd extensions/a1111-sd-webui-tagcomplete.git && \
    pip install -r requirements.txt

# Uncomment and customize the following lines as needed
# RUN cd /workspace/stable-diffusion-webui && \
#     git clone https://github.com/d8ahazard/sd_dreambooth_extension.git extensions/sd_dreambooth_extension && \
#     cd extensions/sd_dreambooth_extension && \
#     git checkout dev && \
#     git reset ${DREAMBOOTH_VERSION} --hard && \
#     mv /requirements_dreambooth.txt ./requirements.txt && pip install -r requirements.txt

RUN cd /workspace/stable-diffusion-webui && \
    mv /workspace/cache-sd-model.py /workspace/stable-diffusion-webui/ && \
    python cache-sd-model.py --use-cpu=all --ckpt /sd-models/SDv1-5.ckpt && \
    pip cache purge && \
    apt clean && \
    mv /workspace/venv /venv && \
    mv /workspace/stable-diffusion-webui /stable-diffusion-webui

# Cache Models
RUN mkdir /sd-models && mkdir /cn-models && \
    wget https://huggingface.co/runwayml/stable-diffusion-v1-5/resolve/main/v1-5-pruned.ckpt -O /sd-models/v1-5-pruned.ckpt && \
    wget https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0/resolve/main/sd_xl_base_1.0.safetensors -O /sd-models/sd_xl_base_1.0.safetensors && \
    wget https://huggingface.co/stabilityai/stable-diffusion-xl-refiner-1.0/resolve/main/sd_xl_refiner_1.0.safetensors -O /sd-models/sd_xl_refiner_1.0.safetensors && \
    wget https://huggingface.co/lllyasviel/ControlNet-v1-1/resolve/main/control_v11p_sd15_canny.pth -O /cn-models/control_v11p_sd15_canny.pth


COPY pre_start.sh /pre_start.sh
COPY relauncher.py webui-user.sh /stable-diffusion-webui/

# NGINX Proxy
COPY nginx.conf /etc/nginx/nginx.conf
COPY readme.html /usr/share/nginx/html/readme.html

# Copy the README.md
COPY README.md /usr/share/nginx/html/README.md

# Start Scripts
COPY pre_start.sh /pre_start.sh
COPY start.sh /start.sh
RUN chmod +x /start.sh && chmod +x /pre_start.sh

# Start Custom Scripts
COPY model-download.sh /model-download.sh
RUN chmod +x /model-download.sh

SHELL ["/bin/bash", "--login", "-c"]
ENTRYPOINT [ "/bin/bash", "-c", "/model-download.sh && /start.sh" ]
