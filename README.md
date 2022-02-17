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
        executor-tarefas
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
O "services" e o "before_script" antes dos stages, então os mesmos serão executados em todos os jobs/stages.<br>
O services: docker:dind" significa que utilizará o serviço docker para subir um container docker localmente.<br>
Obs: dind = Docker in Docker<br>
```
> cd install
> vagrant ssh centos_srv02
    $ cd ~/bytebank
    $ vi .gitlab-ci.yml 
```
```
image: docker:stable

services:
  - docker:dind

before_script:
  - docker info

build-docker:
  stage: build
  tags:
  - executor-tarefas
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
  - docker:dind

before_script:
  - docker info
  - docker login - u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD

build-docker:
  stage: build
  tags:
  - executor-tarefas
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
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/160220220846.png">
</kbd>
<br />
<br />
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/110220221103.png">
</kbd>
<br />
<br />

# Dependência no build
Adicionando as dependências da aplicação e utilizando a tag "executor-tarefas".<br>
Utilizando esta tag, o GitLab sabe qual runner ele irá utilizar.<br>
Já as dependências "forçam" uma sequência, ou seja, uma determinada etapa só será executada, quando a anterior for realizada com sucesso.<br>
Boas práticas definir as stages de forma correta.<br>
GitLab se divide basicamente em quatro passos podendo ser expandidos:<br>
1. Pre-Build<br>
2. Build<br>
3. Testes<br>
4. Deploy<br>

```
> cd install
> vagrant ssh centos_srv02
    $ cd ~/bytebank
    $ vi .gitlab-ci.yml 
```
```
image: docker:stable

stages:
- pre-build
- build
- test
- deploy

services:
- docker:dind

before_script:
- docker info

build-docker:
  stage: pre-build
  tags:
  - executor-tarefas
  script:
  - docker build -t minha-imagem .

build-project:
  stage: build
  tags:
  - executor-tarefas
  dependencies:
  - build-docker
  script:
  - echo "runner"
```

Pipeline com os stages definidos e o segundo aguardando o primeiro finalizar com sucesso.<br>
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/110220221409.png">
</kbd>
<br />
<br />

# Concluindo o build do projeto
Exemplo realizando login no DockerHub para publicar a imagem.<br>
Obs:<br>
1. Removido o services e before_script da execução "geral".<br>
2. Estas varíaveis também poderiam ser incluídas diretamente no GitLab, conforme visto anteriormente.<br>
3. Variáveis criadas conforme arquivo env da aplicação.<br>
4. Imagem enviada ao Docker Hub.<br>
```
> cd install
> vagrant ssh centos_srv02
    $ cd ~/bytebank
    $ vi .gitlab-ci.yml 
```
```
image: docker:stable

stages:
- pre-build
- build
- test
- deploy

build-docker:
  services:
  - docker:dind

  before_script:
  - docker info
  - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD

  stage: pre-build
  script:
  - docker build -t minha-imagem .
  - docker tag minha-imagem fabiokerber/minha-imagem:latest
  - docker push fabiokerber/minha-imagem:latest

  tags:
  - executor-tarefas

build-project:
  image: fabiokerber/minha-imagem:latest

  services:
  - docker:dind

  - mysql:5.7
  variables:
    MYSQL_USER: devops_dev
    MYSQL_PASSWORD: mestre
    MYSQL_DATABASE: todo_dev
    MYSQL_ROOT_PASSWORD: senha  
    DB_NAME: 'todo_dev'
    DB_USER: 'devops_dev'
    DB_PASSWORD: 'mestre'
    DB_PORT: '3306'
    DB_HOST: 'mysql'
    SECRET_KEY: 'r*5ltfzw-61ksdm41fuul8+hxs$86yo9%k1%k=(!@=-wv4qtyv'

  stage: build
  image: fabiokerber/minha-imagem:latest

  dependencies:
  - build-docker

  script:
  - python manage.py makemigrations
  - python manage.py migrate

  tags:
  - executor-tarefas
```
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/160220220913.png">
</kbd>
<br />
<br />

# Realizando testes unitários
Obs:<br>
1. Por se tratar de uma aplicação em pyhon (build-project), o teste será realizado com o .<br>
2. Este teste unitário (python -m unittest setUp), já foi desenvolvido na aplicação.<br>
3. Os testes podem ser realizados em paralelo aos stages.<br>
4. Durante o teste é necessário a conexão com o banco de dados...<br>
5. Por segurança e boas práticas, deve-se utilizar varíaveis direto no Gitlab da mesma forma com o login do docker... (MYSQL_USER: $DB_USER).<br>
```
> cd install
> vagrant ssh centos_srv02
    $ cd ~/bytebank
    $ vi .gitlab-ci.yml 
```
```
image: docker:stable

stages:
- pre-build
- build
- test
- deploy

build-docker:
  services:
  - docker:dind

  before_script:
  - docker info
  - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD

  stage: pre-build
  script:
  - docker build -t minha-imagem .
  - docker tag minha-imagem fabiokerber/minha-imagem:latest
  - docker push fabiokerber/minha-imagem:latest

  tags:
  - executor-tarefas

build-project:
  stage: build
  image: fabiokerber/minha-imagem:latest

  services:
  - docker:dind

  - mysql:5.7
  variables:
    MYSQL_USER: devops_dev
    MYSQL_PASSWORD: mestre
    MYSQL_DATABASE: todo_dev
    MYSQL_ROOT_PASSWORD: senha  
    DB_NAME: 'todo_dev'
    DB_USER: 'devops_dev'
    DB_PASSWORD: 'mestre'
    DB_PORT: '3306'
    DB_HOST: 'mysql'
    SECRET_KEY: 'r*5ltfzw-61ksdm41fuul8+hxs$86yo9%k1%k=(!@=-wv4qtyv'

  dependencies:
  - build-docker

  script:
  - python manage.py makemigrations
  - python manage.py migrate

  tags:
  - executor-tarefas

test-project:
  stage: test
  image: fabiokerber/minha-imagem:latest

  services:
  - docker:dind

  - mysql:5.7
  variables:
    MYSQL_USER: devops_dev
    MYSQL_PASSWORD: mestre
    MYSQL_DATABASE: todo_dev
    MYSQL_ROOT_PASSWORD: senha  
    DB_NAME: 'todo_dev'
    DB_USER: 'devops_dev'
    DB_PASSWORD: 'mestre'
    DB_PORT: '3306'
    DB_HOST: 'mysql'
    SECRET_KEY: 'r*5ltfzw-61ksdm41fuul8+hxs$86yo9%k1%k=(!@=-wv4qtyv'

  dependencies:
  - build-project

  script:
  - python -m unittest setUp

  tags:
  - executor-tarefas
```
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/160220221307.png">
</kbd>
<br />
<br />

# Configurando runner para deploy
Obs:<br>
1. Ao invés de utilizarmos o runner docker (executor-tarefas), utilizaremos um runner mais leve.<br>
2. Serão realizadas tarefas básicas: compressão de todos os arquivos, envio remoto para outro servidor e inicialização da aplicação utilizando o docker compose.<br> 
3. Para o envio dos arquivos automaticamente de um container que será executado no Gitlab para outro servidor, é necessário criar o par de chaves entre os dois pontos para que não solicite credenciais durante o processo.<br>
4. Realizar a mesma etapa duas vezes, caso possa ocorrer um problema que já é conhecido. "retry: 2".<br>

```
Primeiramente vamos autorizar a comunicação via ssh sem autenticação entre Gitlab > Centos.
> cd install
> vagrant ssh gitlab_srv
    $ sudo docker run -d --name gitlab-runner --restart always -v /srv/gitlab-runner/config:/etc/gitlab-runner -v /var/run/docker.sock:/var/run/docker.sock gitlab/gitlab-runner:latest
    $ sudo docker exec -it gitlab-runner /bin/bash
        # su gitlab-runner
        $ ssh-keygen -t rsa (Enter 3x)
        $ cat ~/.ssh/id_rsa.pub (anotar!)

> cd install
> vagrant ssh centos-srv02
    $ vi ~/.ssh/authorized_keys (Inserir chave pública do gitlab-runner@gitlab_srv)
```
```
Anotar o novo token para este runner (Settings > CI/CD > Runners).
> cd install
> vagrant ssh gitlab_srv
    $ sudo docker exec -it gitlab-runner /bin/bash
        # gitlab-runner register
        Enter the GitLab instance URL (for example, https://gitlab.com/):
            http://192.168.0.220
        Enter the registration token:
            "token"
        Enter a description for the runner:
            [gitlab-srv]: RunnerDeploy
        Enter tags for the runner (comma-separated):
            executor-deploy
        Enter an executor: ssh, docker-ssh+machine, custom, docker, docker-ssh, parallels, shell,virtualbox, docker+machine, kubernetes:
            shell
        # su gitlab-runner
        $ ssh vagrant@192.168.0.221
```
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/160220221337.png">
</kbd>
<br />
<br />

```
> cd install
> vagrant ssh centos_srv02
    $ cd ~/bytebank
    $ vi .gitlab-ci.yml 
```
```
image: docker:stable

stages:
- pre-build
- build
- test
- deploy

build-docker:
  services:
  - docker:dind

  retry: 2

  before_script:
  - docker info
  - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD

  stage: pre-build
  script:
  - docker build -t minha-imagem .
  - docker tag minha-imagem fabiokerber/minha-imagem:latest
  - docker push fabiokerber/minha-imagem:latest

  tags:
  - executor-tarefas

build-project:
  stage: build
  image: fabiokerber/minha-imagem:latest

  retry: 2

  services:
  - docker:dind

  - mysql:5.7
  variables:
    MYSQL_USER: devops_dev
    MYSQL_PASSWORD: mestre
    MYSQL_DATABASE: todo_dev
    MYSQL_ROOT_PASSWORD: senha  
    DB_NAME: 'todo_dev'
    DB_USER: 'devops_dev'
    DB_PASSWORD: 'mestre'
    DB_PORT: '3306'
    DB_HOST: 'mysql'
    SECRET_KEY: 'r*5ltfzw-61ksdm41fuul8+hxs$86yo9%k1%k=(!@=-wv4qtyv'

  dependencies:
  - build-docker

  script:
  - python manage.py makemigrations
  - python manage.py migrate

  tags:
  - executor-tarefas

test-project:
  stage: test
  image: fabiokerber/minha-imagem:latest

  services:
  - docker:dind

  - mysql:5.7
  variables:
    MYSQL_USER: devops_dev
    MYSQL_PASSWORD: mestre
    MYSQL_DATABASE: todo_dev
    MYSQL_ROOT_PASSWORD: senha  
    DB_NAME: 'todo_dev'
    DB_USER: 'devops_dev'
    DB_PASSWORD: 'mestre'
    DB_PORT: '3306'
    DB_HOST: 'mysql'
    SECRET_KEY: 'r*5ltfzw-61ksdm41fuul8+hxs$86yo9%k1%k=(!@=-wv4qtyv'

  dependencies:
  - build-project

  script:
  - python -m unittest setUp

  tags:
  - executor-tarefas

deploy-project:
  stage: deploy

  dependencies:
  - test-project

  script:
  - tar cfz arquivos.tgz *
  - scp arquivos.tgz vagrant@192.168.0.221:/tmp/
  - ssh vagrant@192.168.0.221 'cd /tmp; tar xfz arquivos.tgz; sudo /usr/local/bin/docker-compose up -d'

  tags:
  - executor-deploy
```