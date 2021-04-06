FROM rust:latest as deno_builder

RUN cargo install deno --locked

FROM ubuntu:20.04

ARG USERNAME=denouser
ARG USER_UID=1000
ARG USER_GID=$USER_UID

RUN groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    && apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y git wget curl zsh software-properties-common gnupg2 build-essential vim tzdata \
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# GitHub CLI
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-key C99B11DEB97541F0 \
    && apt-add-repository https://cli.github.com/packages \
    && apt update \
    && apt install -y gh \
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

USER denouser

WORKDIR /home/denouser

COPY --from=deno_builder /usr/local/cargo/bin/deno .deno/bin/deno
ENV PATH=/home/denouser/.deno/bin:$PATH

RUN wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | zsh || true
RUN git clone https://github.com/zsh-users/zsh-autosuggestions.git .oh-my-zsh/custom/plugins/zsh-autosuggestions
RUN git clone https://github.com/zsh-users/zsh-syntax-highlighting.git .oh-my-zsh/custom/plugins/zsh-syntax-highlighting
RUN mkdir -p $HOME/.oh-my-zsh/custom/plugins/deno && deno completions zsh > .oh-my-zsh/custom/plugins/deno/_deno
RUN sed -i 's/plugins=(git)/plugins=(git zsh-syntax-highlighting zsh-autosuggestions deno)/g' .zshrc

COPY gitconfig .gitconfig
COPY git.sh ./

ENTRYPOINT [ "/bin/zsh" ]
