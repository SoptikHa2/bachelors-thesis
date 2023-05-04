FROM debian:11

RUN apt update
RUN apt install -y sudo vim git zsh gcc python3 python3-dev python3-venv build-essential python3-pip
RUN ln -s /usr/bin/python3 /usr/bin/python

RUN useradd -ms /bin/zsh s2e
RUN usermod -aG sudo s2e
RUN echo "s2e:s2e" | chpasswd
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

USER s2e
WORKDIR /home/s2e

COPY zshrc /home/s2e/.zshrc
RUN sudo chown s2e:s2e /home/s2e/.zshrc

RUN git clone https://github.com/s2e/s2e-env.git
WORKDIR /home/s2e/s2e-env
RUN echo 'export PATH="$PATH":/home/s2e/.local/bin' >> ~/.zshrc
RUN echo 'source /home/s2e/s2e/s2e_activate' >> ~/.zshrc

RUN pip install --upgrade pip
RUN pip install .

RUN export PATH="$PATH":/home/s2e/.local/bin && \
    s2e init /home/s2e/s2e && \
    . /home/s2e/s2e/s2e_activate && \
    s2e build

RUN sudo usermod -aG kvm,docker s2e

RUN export PATH="$PATH":/home/s2e/.local/bin && \
    . /home/s2e/s2e/s2e_activate && \
    s2e image_build -d debian-11.3-x86_64

ENTRYPOINT [ "/bin/zsh" ]
