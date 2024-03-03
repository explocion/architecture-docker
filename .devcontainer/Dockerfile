FROM archlinux:latest

# Setup pacman and Install Basic Tools
RUN pacman-key --init && pacman -Syy --noconfirm archlinux-keyring
RUN sed -i 's/#en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && locale-gen \
    && echo "LANG=en_US.UTF-8" >> /etc/locale.conf
ENV LANG en_US.UTF-8
RUN pacman -Syyu --noconfirm base-devel git zsh wget openssh tmux neovim bat eza
RUN echo 'EDITOR=nvim' >> /etc/environment

# Setup Default User
RUN useradd -m arch && echo "arch ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
USER arch
ENV USER arch
ENV HOME "/home/$USER"
WORKDIR $HOME
RUN mkdir $HOME/.ssh $HOME/.config $HOME/workspace

# Install AUR helper (paru)
RUN git clone --depth 1 'https://aur.archlinux.org/paru-bin.git' "$HOME/paru-bin" \
    && cd "$HOME/paru-bin" && makepkg --noconfirm -si
RUN rm -rf "$HOME/paru-bin" && paru -Syyu --noconfirm

# Install Rust
RUN paru -S --noconfirm rustup && rustup default stable
ENV PATH "$HOME/.cargo/bin:$PATH"

# SystemVerilog and RiscV Tools
RUN paru -S --noconfirm llvm gdb verilator verible-bin python-cocotb
RUN paru -S --noconfirm riscv64-unknown-elf-gcc spike

# Chisel3 Tools
RUN paru -S --noconfirm jdk-openjdk scala-cli sbt
RUN mkdir "$HOME/chisel-template" && cd "$HOME/chisel-template" \
    && curl -O -L 'https://github.com/chipsalliance/chisel/releases/latest/download/chisel-template.scala' \
    && scala-cli chisel-template.scala

# Clash Tools
RUN paru -S --noconfirm ghcup-hs-bin
ENV PATH "$HOME/.ghcup/bin:$PATH"
RUN ghcup install ghc && ghcup set ghc
RUN ghcup install stack && ghcup set stack
RUN ghcup install cabal && ghcup set cabal
RUN stack new clash-simple clash-lang/simple
RUN cd clash-simple && stack update && stack build

# Setup zsh
COPY --chown=arch .zshrc $HOME/.zshrc
RUN zsh <(curl -s https://raw.githubusercontent.com/zap-zsh/zap/master/install.zsh) --keep
RUN sudo chsh -s $(which zsh) "$USER"

# Clean package cache and pacnew files
RUN paru -Scc --noconfirm && sudo rm -f /etc/*.pacnew

# Shell
CMD ["zsh"]
