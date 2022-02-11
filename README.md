# GitLab-CI

Pré requisito:

|Tool    |Link|
|-------------|-----------|
|`Vagrant`| https://releases.hashicorp.com/vagrant/2.2.19/vagrant_2.2.19_x86_64.msi
|`VirtualBox`| https://download.virtualbox.org/virtualbox/6.1.30/VirtualBox-6.1.30-148432-Win.exe

# Pipeline
"Pipeline é basicamente onde todos os processos acontecem..."<br>
Composta por jobs, são exatamente as etapas que a pipeline deve executar.<br>

# Instalação inicial GitLab
```
> cd install
!! edit .env (GITLAB_IP='192.168.0.220')
!! edit files/gitlab.rb (external_url 'http://192.168.0.220')
> vagrant up (irá subir a VM do GitLab e centos_srv02)
> vagrant ssh gitlab_srv -c 'sudo cat /etc/gitlab/initial_root_password | grep Password:' (anotar!)

http://192.168.0.220
    root | (password)
```

<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/090220221639.jpg">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/090220221642.jpg">
</kbd>
<br />
<br />
Criar novo usuário admin.<br>
<br />
<br />
Criar novo projeto.<br>
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/100220221010.png">
</kbd>
<br />
<br />
Configurar o runner.<br>
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/100220221040.png">
</kbd>
<br />
<br />
Anotar token!
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/100220221101.png">
</kbd>
<br />
<br />

```
Acessar servidor gitlab e executar o registro do Runner.
> cd install
> vagrant ssh gitlab_srv
    $ sudo gitlab-runner register --docker-volumes "/var/run/docker.sock:/var/run/docker.sock"
    Enter the GitLab instance URL (for example, https://gitlab.com/):
        http://192.168.0.220
    Enter the registration token:
        "token"
    Enter a description for the runner:
        [gitlab-srv]: DockerRunner
    Enter tags for the runner (comma-separated):
        "Enter"
    Enter an executor: ssh, docker-ssh+machine, custom, docker, docker-ssh, parallels, shell,virtualbox, docker+machine, kubernetes:
        docker                                                                                             
    Enter the default Docker image (for example, ruby:2.6):
        docker:stable
```

# Hello World com Pipeline
Criar .gitlab-ci.yml<br>
Irá conter todos os jobs e serviços que a pipeline irá executar.
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/100220221021.png">
</kbd>
<br />
<br />
Conteúdo .gitlab-ci.yml
```
job1:
  script:
  - echo "hello world"
```

Commit message
```
iniciando com gitlab-ci
```

<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/100220221107.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/110220221035.png">
</kbd>
<br />
<br />
CI Lint para validação manual de código.
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/100220221127.png">
</kbd>
<br />
<br />

# Criar par de chaves VM GitLab
```
> cd install
> vagrant ssh centos_srv02
    $ ssh-keygen -t rsa -b 4096 -C "fabio.kerber" (conta GitLab) (Enter 3x)
    $ cat ~/.ssh/id_rsa.pub (anotar chave pública!)
    $ cat ~/.ssh/id_rsa (anotar chave privada!)
    $ git config --global user.name "fabio.kerber"
    $ git config --global user.email "fabio.kerber@gmail.com"
    $ git config --global push.default simple
```

Adicionar chave pública ao GitLab<br>
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/110220220842.jpg">
</kbd>
<br />
<br />

# Teste chave pública e download do projeto
```
> cd install
> vagrant ssh centos_srv02
    $ ssh -T git@192.168.0.220 (yes)
    $ cd ~ && git clone http://192.168.0.220/fabio.kerber/bytebank.git
    $ cp -R /vagrant/files/devops/* ~/bytebank
```

# Primeiro commit
```
> cd install
> vagrant ssh centos_srv02
    $ git add .
    $ git commit -m "Apontando versionamento"
    $ git push -f (força o push para a branch pois já existem arquivos lá)
```

<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/110220220910.jpg">
</kbd>
<br />
<br />

# Criando nova Pipeline
```
> cd install
> vagrant ssh centos_srv02
    $ cd ~/bytebank
    $ vi .gitlab-ci.yml 
```
```
tarefa1:
  script:
  - echo "hello world"
  - date
```
```
    $ git add . && git commit -m "Iniciando a pipeline" && git push
```

<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/110220220948.png">
</kbd>
<br />
<br />

# Build Dockerfile (aplicação)
```
> cd install
> vagrant ssh centos_srv02
    $ cd ~/bytebank
    $ vi .gitlab-ci.yml 
```
```
image: docker:stable

services:
- docker:dind:

before_script:
- docker info

build-docker:
    stage: build
    script: 
    - docker build -t minha-imagem .
```
```
    $ git add . && git commit -m "Iniciando a pipeline" && git push
```

# Trabalhando com Variáveis 
Exemplo realizando login no DockerHub para publicar a imagem.<br>
```
> cd install
> vagrant ssh centos_srv02
    $ cd ~/bytebank
    $ vi .gitlab-ci.yml 
```
```
image: docker:stable

services:
- docker:dind:

before_script:
- docker info
- docker login - u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD

build-docker:
    stage: build
    script:
    - docker build - t minha-imagem .
    - docker tag minha-imagem <usuario_dockerhub>/minha-imagem: latest
    - docker push <usuario_dockerhub>/minha-imagem: latest
```
Essas mesmas variáveis serão configuradas dentro da pipeline.
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/110220221056.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/110220221057.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/110220221058.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/110220221101.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/110220221102.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/110220221103.png">
</kbd>
<br />
<br />