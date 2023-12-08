FROM registry.access.redhat.com/ubi8-init:8.9-1

# Configure shell environment
ENV SHELL /bin/bash
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV TZ America/New_York
ENV HOME /root
SHELL ["/bin/bash", "-c"]
ARG APP_ROOT='/app'
WORKDIR $APP_ROOT

# Expose some ports to host by default
EXPOSE 8080 8081 8082 8083 8084 8085

# Define Neovim COC (command completion) extensions to be installed
ARG COC='coc-css coc-cssmodules coc-docker coc-fzf-preview coc-git coc-html coc-htmldjango coc-jedi coc-json coc-lua coc-markdownlint coc-perl coc-powershell coc-pydocstring coc-sh coc-sql coc-swagger coc-tsserver coc-xml coc-yaml'

# We'll use Lazygit: here are configs
ARG LG='lazygit'
ARG LG_GITHUB='https://github.com/jesseduffield/lazygit/releases/download/v0.40.2/lazygit_0.40.2_Linux_x86_64.tar.gz'
ARG LG_ARCHIVE='lazygit.tar.gz'

# Set the working directory in the container
ARG LOG="/root/build.log"
RUN set -eux; \
	mkdir -p $APP_ROOT; \
	echo "Installed ubi8-init image" > $LOG

# Update packages list and install essential dependencies
RUN set -eux; \
	echo "Install dependencies" >> $LOG; \
	dnf install -y -v \
		cmake \
		curl \
		git \
		gcc \
		gdb \
		gettext \
		gettext-libs \
		make \
		python3-devel \
		python3-pip \
		python3-virtualenv \
		unzip; \
	echo "Finished installing dependencies" >> $LOG

# Install nvm
SHELL ["/bin/bash", "-c"]
ENV NVM_DIR="/root/.nvm"
RUN set -eux; \
	echo "Install nvm" >> $LOG; \
	mkdir -p $NVM_DIR; \
	curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash; \
	[ -s "$NVM_DIR/nvm.sh" ] && sed -i '/NVM_SCRIPT_SOURCE/s/$_/bash/' "$NVM_DIR/nvm.sh"; \
	echo done | tee $LOG

# Install node and npm
RUN set -eux; \
	echo "Install node and npm" >> $LOG; \
	[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"; \
	[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"; \
	nvm install node 2>&1 | tee $LOG; \
	nvm install-latest-npm 2>&1 | tee $LOG; \
	echo "Finished installing node and npm" >> $LOG

# NodeJS package manager tie-in with Nvim
RUN set -eux; \
	echo "Install NodeJS package manager tie-in with Nvim" >> $LOG; \
	[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"; \
	npm i -g neovim 2>&1 | tee $LOG; \
	echo "Finished installing NodeJS package manager tie-in with Nvim" >> $LOG

# Install Neovim
#RUN set -eux; \
#	echo "Install Neovim" >> $LOG; \
#	curl -L https://github.com/neovim/neovim/releases/download/v0.8.2/nvim-linux64.tar.gz | tar xzvf -;  \
#	cp -a nvim-linux64/bin/* /usr/local/bin/; \
#	cp -a nvim-linux64/lib/* /usr/local/lib/; \
#	cp -a nvim-linux64/share/* /usr/local/share/; \
#	ln -s /usr/local/bin/nvim /usr/bin/nvim; \
#	rm -rf nvim-linux64; \
#	echo "Finished installing dependencies" >> $LOG

# Build and install Neovim
ARG NBUILDDIR="/app/tmp"
RUN set -eux; \
	echo "Install Nvim" >> $LOG; \
	mkdir -p $NBUILDDIR; \
	cd $NBUILDDIR; \
	git clone https://github.com/neovim/neovim; \
	cd $NBUILDDIR/neovim; \
	git checkout stable; \
	nproc="$( nproc )"; \
	make -j${nproc}; \
	make install; \
	rm -rf $NBUILDDIR; \
	echo "Finished installing Nvim" >> $LOG

# Create a Python virtual environment and install required packages
RUN set -eux; \
	echo "Upgrade pip and install pynvim + dependencies" >> $LOG; \
	pip3 install --upgrade pip; \
	pip3 install flake8 black isort pynvim; \
	echo "Finished upgrading pip and installing pynvim + dependencies" >> $LOG

# Install vim-plug
RUN set -eux; \
	echo "Install vim-plug" >> $LOG; \
	curl -fLo /root/.local/share/nvim/site/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim; \
	echo "Finished installing vim-plug" >> $LOG

# COC extensions
RUN set -eux; \
	echo "Install COC extensions" >> $LOG; \
	mkdir -p /root/.config/coc/extensions; \
	cd /root/.config/coc/extensions; \
	[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"; \
	npm install $COC --global --only=prod 2>&1 | tee $LOG; \
	echo "Finished installing neovim command completion extensions" >> $LOG
	

# Lazygit
RUN set -eux; \
	echo "Install Lazygit" >> $LOG; \
	mkdir -p /root/tmp; \
	cd /root/tmp; \
	curl -L -o $LG_ARCHIVE $LG_GITHUB; \
	tar xzvf $LG_ARCHIVE; \
	mv $LG /usr/bin/; \
	rm -rf /root/tmp; \
	echo "Finshed installing Lazygit" >> $LOG

# Start Neovim
#CMD ["nvim"]
RUN set -eux; \
	echo "Finished building dev environment" >> $LOG
