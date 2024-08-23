FROM ubuntu:24.04

RUN apt-get update \
  && apt-get install -y \
  curl \
  gpg \
  wget

RUN wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
RUN install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
RUN echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list
RUN rm -f packages.microsoft.gpg

RUN  apt-get install -y apt-transport-https \
  && apt-get update \
  && apt-get install -y \
  --no-install-recommends \
  code sudo ca-certificates iptables openssl pigz xz-utils build-essential

RUN curl --proto =https --tlsv1.2 -sSf -L https://install.determinate.systems/nix | sh -s -- install linux --extra-conf "sandbox = false" --init none --no-confirm
ENV PATH=${PATH}:/nix/var/nix/profiles/default/bin
RUN sed -i 's/auto-allocate-uids = true/auto-allocate-uids = false/g' /etc/nix/nix.conf

ARG USER=coder
ARG USER_ID=30033
ARG GROUP_ID=30033

RUN apt-get install -y ruby git direnv unzip

RUN addgroup --gid $GROUP_ID ${USER} \
  && useradd --groups sudo --no-create-home --shell /bin/bash ${USER} --uid ${USER_ID} --gid ${GROUP_ID} \
  && echo "${USER} ALL=(ALL) NOPASSWD:ALL" >/etc/sudoers.d/${USER} \
  && chmod 0440 /etc/sudoers.d/${USER}
RUN mkdir -p /home/${USER} && chown -R ${USER}:${USER} /home/${USER}

ENV DOCKER_TLS_CERTDIR=/certs
RUN mkdir /certs /certs/client && chmod 1777 /certs /certs/client
COPY --from=docker:27-dind /usr/local/bin/ /usr/local/bin/
COPY --from=docker:27-dind /usr/local/libexec/ /usr/local/libexec/
VOLUME /var/lib/docker

USER ${USER}

ENV PATH=${PATH}:/home/${USER}/.nix-profile/bin

RUN sudo chown -R ${USER}:${USER} /nix/store /nix/var

ENV USER=${USER}

RUN echo "trusted-users = root $USER" | sudo tee -a /etc/nix/nix.conf

RUN nix profile install --accept-flake-config github:cachix/cachix

RUN cachix use devenv
RUN nix profile install --accept-flake-config github:cachix/devenv/latest

SHELL ["bash", "-c"]

RUN echo 'extra-trusted-substituters = https://cache.flox.dev' | sudo tee -a /etc/nix/nix.conf \
  && echo 'extra-trusted-public-keys = flox-cache-public-1:7F4OyH7ZCnFhcze3fJdfyXYLQw/aV7GEed86nQ7IsOs=' | sudo tee -a /etc/nix/nix.conf

RUN nix profile install --impure --experimental-features "nix-command flakes" --accept-flake-config 'github:flox/flox'

RUN curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh | bash

WORKDIR /home/${USER}

RUN  curl https://pkgx.sh | sh

ENV PATH=${PATH}:/home/${USER}/.local/bin

RUN echo 'eval "$(pkgx --shellcode)"' >> .bashrc

ENV PATH=${PATH}:/home/linuxbrew/.linuxbrew/bin

RUN sudo usermod -a -G nixbld ${USER}

ENV FORCE=1
RUN curl -fsSL https://get.jetpack.io/devbox | bash

RUN sudo chown -R ${USER}:${USER} /usr/local

RUN curl -s https://ohmyposh.dev/install.sh | bash -s -- -d /usr/local/bin

RUN git clone https://github.com/JanDeDobbeleer/oh-my-posh.git .oh-my-posh

RUN echo 'eval "$(oh-my-posh init bash --config ~/.oh-my-posh/themes/tokyonight_storm.omp.json)"' >> .bashrc

RUN curl --proto '=https' --tlsv1.2 -sSf https://setup.atuin.sh | bash

RUN curl -fsSL https://github.com/cashapp/hermit/releases/download/stable/install.sh | /bin/bash

ENV PATH=${PATH}:/home/${USER}/bin

ENV BASH_ENV=/home/${USER}/.bashrc

ARG PACKAGES

RUN if [ -n "$PACKAGES" ]; then \
  brew install ${PACKAGES}; exit 0;\
  fi

COPY entry.sh /usr/local/bin/entry.sh

RUN sudo chmod a+x /usr/local/bin/entry.sh

ENTRYPOINT [ "entry.sh" ]
