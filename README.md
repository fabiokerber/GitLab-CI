# GitLab-CI

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