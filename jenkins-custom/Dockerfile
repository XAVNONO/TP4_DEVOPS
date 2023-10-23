# Image Officielle de Jenkins (dernière version)
FROM jenkins/jenkins:lts

# Utilisateur actif dans le container docker
USER root

# Installation des mises à jour packages
RUN apt-get update -qq \
    && apt-get install -qqy apt-transport-https ca-certificates curl gnupg2 software-properties-common

# Téléchargement de la clé GPG du référentiel de docker 
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
# Ajoute le référentiel Docker au fichier de configuration des sources APT 
RUN add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"

# Mises à jour des paquets et Installation de Docker
RUN apt-get update  -qq \
    && apt-get -y install docker-ce

# Ajoute l'utilisateur Jenkins au groupe docker pour communication avec le daemon docker    
RUN usermod -aG docker jenkins

# Installation de docker-compose et nettoyage de l'espace disque utilisé pour le téléchargement des paquets
RUN apt-get update -qq \
    && apt-get install -qqy docker-compose && \
    rm -rf /var/lib/apt/lists/*

# Installation de dependence wget, gnupg et lsb-release 
RUN apt-get update -qq \
    && apt-get install -y \
    wget \
    gnupg \
    lsb-release \
    && rm -rf /var/lib/apt/lists/*

# Installation de Terraform
RUN wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
RUN echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/hashicorp.list
RUN apt-get update && apt-get install -y terraform
RUN terraform --version
# RUN wget --quiet https://releases.hashicorp.com/terraform/0.11.3/terraform_0.11.3_linux_amd64.zip \
#     && unzip terraform_0.11.3_linux_amd64.zip \
#     && mv terraform /usr/bin \
#     && rm terraform_0.11.3_linux_amd64.zip \
#     && rm -rf /var/lib/apt/lists/*

# Installation de Kubectl
RUN curl -LO "https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl" && \
    chmod +x kubectl && \
    mv kubectl /usr/local/bin/

# installation de dependence conntrack
RUN apt-get update -qq \
    && apt-get install -y conntrack \
    && rm -rf /var/lib/apt/lists/*

# Installation de Minikube
RUN curl -LO "https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64" && \
    chmod +x minikube-linux-amd64 && \
    mv minikube-linux-amd64 /usr/local/bin/minikube

CMD     tail -F /dev/null