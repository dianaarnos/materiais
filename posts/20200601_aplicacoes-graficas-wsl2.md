---
created_at: 2020-06-01
title: Aplicações gráficas no WSL2
published: true
description: Como rodar aplicações gráficas direto do Linux no seu Windows 10
tags: tech, windows, wsl2, x11
cover_image: https://d33wubrfki0l68.cloudfront.net/e321a98c8c2aa029fb212a633bc6e0332f00fb3c/9e16a/images/posts/aplicacoes_graficas_wsl2/wsl2.png
---

*Artigo publicado originalmente em [dianaarnos.com](https://www.dianaarnos.com/)*

Antes de começar, vale lembrar que as features mais recentes estão disponíveis apenas para quem está no Fast Ring do Windows Insider Program.
No momento de publicação desse artigo, eu estou na build 19619.1000 e usando o Ubuntu 20.04 como distro.

Para conseguir rodar aplicações gráficas do Linux no seu Windows 10 você precisa fazer forwarding do X11 de forma que seu W10 possa receber os sinais e executar as instruções gráficas.

#### Mas o que é X11?

De maneira simplista e rasa, vou dizer que é um protocolo de comunicação que permite que aplicações possam criar objetos como janelas e usar primitivas básicas de desenho.
Simplificando ainda mais, podemos assumir que X11 é o responsável pelo ambiente gráfico do Linux (na maioria das distros, mas nem todas usam).
Essa não é a explicação mais correta, mas é o suficiente pra entender o que vamos fazer aqui.

Resumidamente: você vai iniciar uma aplicação gráfica no Linux. A partir daí, as instruções gráficas dessa execução precisam ser encaminhadas para o Windows 10, de forma que possam ser interpretadas e executadas. Isso é feito por servidor X11 instalado no Windows, que então reproduz a janela da aplicação na sua área de trabalho.

Agora, ao passo-a-passo simples:

### 1. SSH

Garanta que a distro tenha um servidor ssh instalado (o openssh-server é "mais que bom") e edite o arquivo de configuração `/etc/ssh/sshd_config` para que contenha essas linhas:

```bash
Port 222
X11Forwarding yes
X11DisplayOffset 10
```

Dessa forma você garante que a distro possa enviar os dados do X11 via ssh, permitindo que um servidor X11 rodando no seu host Windows receba esses dados.

### 2. Display
Faça export da variável de ambiente `DISPLAY` do Linux, adicionando a seguinte linha ao arquivo `.bashrc`:

```
export DISPLAY=<IP do host Windows>:0.0
```

![](https://d33wubrfki0l68.cloudfront.net/be591201110fc28ba38f11470cadb914fcf3397d/d7045/images/posts/aplicacoes_graficas_wsl2/display.png)

Em poucas e porcas palavras, essa linha diz para a distro enviar os dados gráficos para o IP especificado.

Mas qual IP usar?

Você consegue visualizar os IPs necessários digitando `ipconfig` no *Windows PowerShell*.  
Se você estiver usando internet via cabo, você vai usar o IPv4 exibido em *Adaptador Ethernet vEthernet (WSL)*.  
Se você estivar usando internet via wi-fi, é o IPv4 do *Adaptador de Rede sem Fio Wi-Fi*.  
Algo assim:

![](https://d33wubrfki0l68.cloudfront.net/290909bd5040b101b0766d5ec5412dd8e55000b9/12c17/images/posts/aplicacoes_graficas_wsl2/ip.png)

### 3. Instale um servidor X11 no Windows e execute-o

Eu uso o **Xming**, mas existem outros compatíveis com Windows 10. Escolha seu preferido :)

Ao executar o servidor, configure para usar o display de número 0 (aquele que foi exportado no seu .bashrc da distro) e marque a opção *"No access control"*. Assim você não precisa se preocupar com configurações de firewall ou coisas do gênero (você está rodando tudo local mesmo ¯\\\_(ツ)\_/¯ ).

Seguem prints de como faço isso no Xming:

![](https://d33wubrfki0l68.cloudfront.net/eb0f02f085e6c25e435cfc4e812715169e864601/13bd5/images/posts/aplicacoes_graficas_wsl2/xming1.png)

![](https://d33wubrfki0l68.cloudfront.net/b5249d7d1cb3437ea53711ad936d952883aa854f/86d8c/images/posts/aplicacoes_graficas_wsl2/xming2.png)

![](https://d33wubrfki0l68.cloudfront.net/c9ee6c667ca25a4d8926ebb22d2ee61ef503140a/eb815/images/posts/aplicacoes_graficas_wsl2/xming3.png)

Depois de concluir, o servidor vai ficar ativo e disponível lá na barra de tarefas:

![](https://d33wubrfki0l68.cloudfront.net/d63b17ad0edbeaf5f9f28d7d080836d983f2c3b4/63b1a/images/posts/aplicacoes_graficas_wsl2/xming4.png)

#### 4. E agora é só executar uma aplicação gráfica no Linux:

{% youtube zArM8gBsJsA %}

Se achou útil, compartilhe 🙂  
Se tiver alguma dúvida, deixa nos comentários ou me manda mensagem nas redes sociais. Ficarei feliz em ajudar :D