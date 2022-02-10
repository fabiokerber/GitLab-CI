# GitLab-CI

# Pipeline
"Pipeline é basicamente onde todos os processos acontecem..."<br>
Composta por jobs, são exatamente as etapas que a pipeline deve executar.<br>

# Instalação inicial GitLab
```
> cd install
!! edit files/gitlab.rb (external_url 'http://192.168.0.220')
> vagrant up
> vagrant ssh -c 'sudo cat /etc/gitlab/initial_root_password | grep Password:' (anotar!)

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
Criar novo projeto.<br>
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/100220221010.jpg">
</kbd>
<br />
<br />
Configurar o runner.<br>
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/100220221040.jpg">
</kbd>
<br />
<br />
Anotar token!
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/100220221101.jpg">
</kbd>
<br />
<br />
```
Acessar servidor gitlab e executar o registro do Runner.
> cd install
> vagrant ssh gitlab_srv
    $ sudo gitlab-runner register
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
        ruby:2.6
```

# Hello World com Pipeline
Criar .gitlab-ci.yml<br>
Irá conter todos os jobs e serviços que a pipeline irá executar.
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/100220221021.jpg">
</kbd>
<br />
<br />
Conteúdo .gitlab-ci.yml
```
job1: (primeiro serviço/etapa)
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
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/100220221108.png">
</kbd>
<br />
<br />
CI Lint para validação manual de código.
<kbd>
    <img src="https://github.com/fabiokerber/GitLab-CI/blob/main/img/100220221127.png">
</kbd>
<br />
<br />