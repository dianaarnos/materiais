---
created_at: 2018-06-13
title: WordPress no Google App Engine
tags: programação, wordpress, google app engine
description: Recentemente eu precisei disponibilizar uma instalação WordPress no Google App Engine e deu um pouco de trabalho pra achar materiais de consulta atuais, então resolvi disponibilizar um passo-a-passo pra facilitar a vida de quem precisar fazer isso ou tiver a curiosidade de tentar :)
published: true
---
Recentemente eu precisei disponibilizar uma instalação <a href="https://wordpress.org/" target="_blank">WordPress</a> no <a href="https://cloud.google.com/appengine/" target="_blank">Google App Engine</a> e deu um pouco de trabalho pra achar materiais de consulta atuais, então resolvi disponibilizar um passo-a-passo pra facilitar a vida de quem precisar fazer isso ou tiver a curiosidade de tentar :)

1. Primeiro, você precisa configurar o Google Cloud SDK com a sua conta e o ID do projeto:
~~~
gcloud init
~~~
Caso você ainda não saiba configurar o SDK, a documentação oficial é fácil de seguir: https://cloud.google.com/sdk/docs/ e pra criar um novo projeto no Cloud Console: https://cloud.google.com/resource-manager/docs/creating-managing-projects
2. Criar uma aplicação do App Engine:
~~~
gcloud app create
~~~
3. Configurar o bucket padrão do Google Storage.

Você pode fazer isso tanto pelo Cloud Console quanto pela linha de comando. Vamos usar o terminal, já que ele já tá aqui na nossa cara:
~~~
gsutil defacl ch -u AllUsers:R gs://IDDOSEUPROJETO.appspot.com
~~~
O que é um bucket? É o "espaço" que sua aplicação vai usar pra armazenar, editar e criar os arquivos que ela precisa. Já que o App Engine em si não dá permissão de escrita pra aplicação, você usa um "reservatório" (bucket) externo.

Ok, até aqui nós preparamos o terreno pra subir o código do WordPress e armazenar os arquivos dele. Agora precisamos de um banco de dados!
4. Vamos criar uma instância do "Cloud SQL for MySQL Second Generation":
~~~
gcloud sql instances create NOMEDAINSTANCIA \
  --activation-policy=ALWAYS \
    --tier=db-n1-standard-1
~~~
Obs.: com esse comando a instância vai ser criada na região padrão: us-central1
5. Vamos mudar a senha do usuário root do seu banco de dados (POR FAVOR, NÉ?)
~~~
gcloud sql users set-password root % \
  --instance NOMEDAINSTANCIA --password=ASUANOVASENHADEROOT
~~~
Agora nós precisamos fazer a configuração inicial do banco pro WordPress (criar o schema, basicamente). Pra isso, obviamente, vamos ter que acessar remotamente a instância de banco que acabamos de criar.
6. Acessando nosso banco de dados

Acontece que, por questões de segurança (obrigada, Google) você só consegue acessar uma instância de banco de dados através de um proxy que usa as suas credenciais do Google Cloud.

Antes de subirmos esse proxy (relaxa, tem uma ferramenta de linha de comando da própria Google pra isso) precisamos gerar as nossas credenciais de acesso ao projeto. Então…

6.1. Acesse a sessão de credenciais do seu projeto: https://console.cloud.google.com/apis/credentials/

6.2. Clique em "Criar credenciais" e então em "Chave de conta de serviço"

6.3. Escolha "JSON" e clique em "Criar" (favor salvar esse arquivo com segurança, não seja ingênuo)

6.4. Agora sim, suba o proxy pra acessar o seu banco:
~~~
cloud_sql_proxy \
  -dir /tmp/cloudsql \    -instances=IDDOSEUPROJETO:REGIÃODAINSTÂNCIA:NOMEDASUAINSTÂNCIA=tcp:PORTAQUEVOCÊESCOLHER -credential_file=SEUARQUIVOJSON
~~~
Se você ainda não tem o sql proxy na sua máquina, bora dar uma geral na documentação, é fácil de seguir: https://cloud.google.com/sql/docs/mysql/connect-external-app#proxy

Agora é só acessar a instância e criar o banco que será usado com seu WordPress:
~~~
mysql -h 127.0.0.1 -u root -p
mysql> create database NOMEDOBANCO;
mysql> create user 'NOMEDOUSUÁRIODOBANCO'@'%' identified by 'SENHADOUSUÁRIO';
mysql> grant all on NOMEDOBANCO.* to 'NOMEDOUSUÁRIO'@'%';
mysql> exit
~~~
Basicamente, o ambiente está pronto \o/

Mas tem alguns truques a serem feitos pra "ensinar" o WordPress a fazer todo o processo de escrita de arquivos no bucket e usar algumas bibliotecas do App Engine que vão permitir que o PHP se comunique com os serviços da Google Cloud sem problemas.

Sim, isso pode dar um certo trabalho, MAS… A linda da Google já deixou tudo pronto pra gente :D

Existe um repositório <a href="https://github.com/GoogleCloudPlatform" target="_blank">no GitHub da própria Google Cloud Platform</a> que chama "php-docs-samples". Ele tem MUITOS exemplos de código PHP pra usar cada um dos serviços de Cloud (sério, cada um mesmo).

Entre tantos scripts úteis, o que vamos usar é um que faz o download do WordPress, configura com as bibliotecas da Google e já prepara a configuração para o deploy no App Engine.
7. Clone o repositório com os scripts da Google
~~~
git clone git@github.com:GoogleCloudPlatform/php-docs-samples.git
~~~
8. Copie o conteúdo da pasta "appengine/wordpress" onde você deseja deixar seu projeto
9. Instale as dependências via composer
~~~
composer install
~~~
10. Execute o script preparado pelo pessoal da Google
~~~
php wordpress-helper.php setup
~~~
Obs.: você pode rodar com a opção "-vv", que é um verbose bem detalhado. Eu prefiro assim, pra ajudar a entender o que está acontecendo.

Durante a execução, o script vai perguntar coisas pra ajudar a configurar seu projeto e já sua instância do App Engine (diretório, ambiente, banco de dados). E aqui temos um grande porém.

Tanto no script quanto no README do repositório é dito que podemos escolher entre o App Engine Standard e o App Engine Flexible (prometo explicar a diferença entre os 2 num post futuro), mas você precisa escolher o Flexible.

Por quê?

Por limitações de configuração do próprio modelo Standard o WordPress não consegue evoluir a instalação do step 1 para o step 2 graças a um timeout na execução do script. ¯\\_(ツ)_/¯

Eu criei <a href="https://github.com/GoogleCloudPlatform/php-docs-samples/issues/618" target="_blank">uma issue reportando isso</a>, mas infelizmente ainda não consegui reservar um tempo pra trabalhar nela.

Bom… finalizada a instalação/configuração de instância, hora do deploy \o/
11. Deploy time!

Acesse o diretório que você indicou durante a execução do "wordpress-helper.php" quando ele perguntou
~~~
    Please enter a directory path for the new project (defaults to my-wordpress-project)
~~~
E rode o seguinte comando:
~~~
gcloud app deploy \
    --promote --stop-previous-version app.yaml cron.yaml
~~~
Isso vai iniciar o processo de reserva da instância no ambiente da Google Cloud Platform, subir seu código (o WordPress, no caso) e realizar as configurações necessárias. Como estamos subindo um ambiente Flexible, o processo pode levar alguns minutos. Hora de pegar um café.
12. Finalizando a instalação

Com o deploy finalizado, é só acessar o endereço do seu projeto e seguir com a instalação padrão do WordPress. A url padrão para o seu projeto é https://IDDOSEUPROJETO.appspot.com/

Depois de passar pelo processo padrão, você precisa habilitar 2 plugins no seu dashboard, que já foram instalados previamente pelo script da Google:
~~~
    Batcache Manager
    GCS media plugin
~~~
Eles vão permitir que sua instalação use os recursos da Google Cloud Platform (nesse caso, o cache e o bucket pra armazenar arquivos).

Pronto! WordPress rodando liso ;)
Instalando e atualizando temas, plugins e o próprio WordPress

Como o App Engine não dá permissão de escrita pra aplicação, plugins e temas precisam ser instalados localmente e colocados no ar via deploy.

Para atualizar plugins, temas e a própria versão do WordPress é só usar a ferramenta de linha de comando (lembre-se de deixar o sql proxy rodando pra você poder acessar sua instância de banco).

Atualizando o WordPress em si:
~~~
vendor/bin/wp core update --path=DIRETORIODOWORDPRESS
~~~
Atualizando todos os plugins:
~~~
vendor/bin/wp plugin update --all --path=DIRETORIODOWORDPRESS
~~~
Atualizando todos os temas:
~~~
vendor/bin/wp theme update --all --path=DIRETORIODOWORDPRESS
~~~
E é isso. A parte realmente interessante é perceber que a lógica de pensamento pra disponibilizarmos uma aplicação usando um PaaS (Platform as a Service) é diferente da que usamos pra disponibilizar alguma coisa usando IaaS (Infrasctructure as a Service), já que temos que pensar em conectar diversos serviços diferentes.

Se tiver dúvidas ou quiser discutir comigo sobre, só me chamar lá no twitter :)
