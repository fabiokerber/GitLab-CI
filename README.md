# GitLab-CI

Pré requisito:

|Tool    |Link|
|-------------|-----------|
|`Vagrant`| https://releases.hashicorp.com/vagrant/2.2.19/vagrant_2.2.19_x86_64.msi
|`VirtualBox`| https://download.virtualbox.org/virtualbox/6.1.30/VirtualBox-6.1.30-148432-Win.exe


# Pipeline
"Pipeline é basicamente onde todos os processos acontecem..."<br>
Composta por jobs, são exatamente as etapas que a pipeline deve executar.<br>

```
> cd install
> edit files/gitlab.rb (external_url 'http://192.168.0.220')
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