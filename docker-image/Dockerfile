FROM debian:11

# Install packages required for s2e
RUN apt update
RUN apt install -y sudo vim git zsh gcc python3 python3-dev python3-venv build-essential python3-pip linux-perf
RUN ln -s /usr/bin/python3 /usr/bin/python

# Setup s2e:s2e user
RUN useradd -ms /bin/zsh s2e
RUN usermod -aG sudo s2e
RUN echo "s2e:s2e" | chpasswd
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

USER s2e
WORKDIR /home/s2e

COPY zshrc /home/s2e/.zshrc
RUN sudo chown s2e:s2e /home/s2e/.zshrc

# Fetch s2e-env
RUN git clone https://github.com/s2e/s2e-env.git
WORKDIR /home/s2e/s2e-env

# Install s2e-env
RUN pip install --upgrade pip
RUN pip install .

WORKDIR /home/s2e

RUN echo 'export PATH="$PATH":/home/s2e/.local/bin' >> ~/.zshrc
RUN echo 'source /home/s2e/s2e/s2e_activate' >> ~/.zshrc
# Use s2e-env to install s2e
RUN export PATH="$PATH":/home/s2e/.local/bin && \
    s2e init /home/s2e/s2e

# Switch s2e to custom fork
WORKDIR /home/s2e/s2e/source/s2e
RUN git remote set-url s2e https://github.com/SoptikHa2/s2e.git; git fetch s2e; git checkout chef
WORKDIR /home/s2e

# Build s2e
RUN export PATH="$PATH":/home/s2e/.local/bin && \
    . /home/s2e/s2e/s2e_activate && \
    s2e build

RUN echo 'export PATH="$PATH":/home/s2e/.local/bin' >> ~/.zshrc
RUN echo 'source /home/s2e/s2e/s2e_activate' >> ~/.zshrc

RUN sudo usermod -aG kvm,docker s2e

# Download base linux x86 image. Remove this if you want a slim image (might break in the future)
RUN export PATH="$PATH":/home/s2e/.local/bin && \
    . /home/s2e/s2e/s2e_activate && \
    s2e image_build -d debian-11.3-x86_64

# Setup ssh. This is needed for any kind of jetbrains remote development.
# See here: https://www.jetbrains.com/help/clion/remote-development-overview.html
# Run "sudo /etc/init.d/ssh start" to start ssh afterwards.
# It's turned off by default.
COPY id_rsa.pub /home/s2e/.ssh/authorized_keys
RUN sudo apt install -y openssh-server

ENTRYPOINT [ "/bin/zsh" ]
