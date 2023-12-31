FROM base:local

# ***
# cdktf & Node.js Installation
# ***
ENV TERRAFORM_VERSION=1.6.6-1
ENV NODE_MAJOR=20
ENV NODE_VERSION=20.5.1-1nodesource1
ENV CDKTF_CLI_VERSION=0.19.2
ENV PIP_PIPENV_VERSION=2023.11.15

# g++ required for cdk-cli
RUN apt-get update && \
    apt-get install -y g++ && \
    curl -fsSL https://apt.releases.hashicorp.com/gpg | \
      gpg --dearmor | \
      tee /usr/share/keyrings/hashicorp-archive-keyring.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
      https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
      tee /etc/apt/sources.list.d/hashicorp.list && \
    apt-get update && \
    apt-get install -y terraform=$TERRAFORM_VERSION && \
    curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | \
      gpg --dearmor -o /usr/share/keyrings/nodesource.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/nodesource.gpg] \
      https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | \
      tee /etc/apt/sources.list.d/nodesource.list && \
    apt-get update && \
    apt-get install -y nodejs=$NODE_VERSION && \
    apt-get clean all

ENV CDKTF_HOME_DIR=/home/cdktf

RUN useradd -m cdktf -d $CDKTF_HOME_DIR

ENV PATH=$CDKTF_HOME_DIR/.local/bin:$PATH

USER cdktf

WORKDIR $CDKTF_HOME_DIR

RUN terraform --version  && \
    node --version || \
    exit 1

RUN npm install cdktf-cli@$CDKTF_CLI_VERSION

RUN npx cdktf --version || \
    exit 1

# TODO pin version
RUN curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py && \
    python3 get-pip.py --user && \
    rm get-pip.py

RUN python3 -m pip -V || exit 1

RUN python3 -m pip install --user pipenv==$PIP_PIPENV_VERSION

COPY --chown=cdktf:cdktk Pipfile Pipfile.lock cdktf.json ./

RUN pipenv install

RUN npx cdktf get
 
COPY --chown=cdktf:cdktk cdktf.out/ cdktf.out
COPY --chown=cdktf:cdktk Makefile provisioning.sh ./

ENTRYPOINT [ "make" ]

