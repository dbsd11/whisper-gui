# Base-Stage
FROM condaforge/miniforge3 AS base
LABEL maintainer="Julian Reith <julianreith@gmx.de>"
LABEL description="Docker container for whisper-gui - A simple GUI made with gradio to use Whisper from https://github.com/Pikurrot/whisper-gui"

EXPOSE 7860
ENV TZ="Europe/Berlin"

# install apt prerequisites
WORKDIR /
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone && \
    apt update && apt install -y \
        curl \
        ffmpeg \
        git \
        vim \
        wget && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*
    
# create conda env and clone whisper-gui
SHELL ["bash", "-c"]
RUN conda create --name whisper-gui python=3.10 -y && \
    conda clean -a -y && \
    conda update -n base -c conda-forge -c pytorch conda && \
    echo "conda activate whisper-gui" >> ~/.bashrc && \
    source /opt/conda/etc/profile.d/conda.sh -y && \
    conda activate whisper-gui && \
    conda install -c conda-forge -c pytorch \
        "numpy<2" \
        pytorch==2.0.1 \
        torchaudio==2.0.2 \
        cpuonly && \
    pip install \
        ctranslate2==4.6.0 \
        faster-whisper==1.1.0 \
        gradio==5.25.2 \
        gradio-client==1.8.0 \
        git+https://github.com/m-bain/whisperx.git@v3.3.2 && \
    git clone https://github.com/dbsd11/whisper-gui

# prepare whisper-gui
WORKDIR /whisper-gui
COPY configs/config.json configs/config.json
RUN source /opt/conda/etc/profile.d/conda.sh && \
    conda activate whisper-gui && \
    mkdir -p outputs && \
    sed -i 's/demo\.launch(inbrowser=args\.autolaunch, share=args\.share)/demo.launch(server_name="0.0.0.0", server_port=7860)/' main.py && \
    sed -i 's/\(^[^d]*\)blockPrint()/# \1blockPrint()/' main.py && \
    sed -i 's/from whisperx.vads.pyannote import VoiceActivitySegmentation/from whisperx.vad import VoiceActivitySegmentation/g' scripts/whisper_model.py && \
    chmod +x *.py scripts/*.py && \
    chmod +x *.sh scripts/*.sh

# store versions in /VERSION
RUN source /opt/conda/etc/profile.d/conda.sh && \
    conda activate whisper-gui && \
    touch /VERSION && \
    whisper_gui_version="v2.3.1" && \
    echo "whisper-gui=$whisper_gui_version" > /VERSION && \
    whisperx_version=$(pip show whisperx | grep Version | cut -d' ' -f2) && \
    echo "whisperx=v$whisperx_version" >> /VERSION 

# setup entrypoint
COPY docker-entrypoint.sh /
RUN chmod +x /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]
