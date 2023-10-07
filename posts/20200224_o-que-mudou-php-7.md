---
created_at: 2020-02-24
title: O que mudou no PHP7
published: true
description: Um curto e simples artigo sobre as mudanças internas no PHP7
tags: programação, php7, php
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/09zhpiwq9peed7a81u9d.png
---

Como muita gente ainda me pergunta o porquê dessa grande diferença entre o PHP 5 e o 7, resolvi escrever um curto e simples artigo sobre isso (afinal, tem gente que sabe muito mais do que eu e poderia explicar muito melhor).

Apesar de ter saído quase 5 anos atrás (3 de Dezembro de 2015), parece que só agora parte das pessoas começou a perceber o que aconteceu com o PHP7, que chegou fazendo barulho não só na comunidade PHP, mas também chamou a atenção de uma galera que tinha desistido de acompanhar a linguagem desde a versão 4. Afinal, a partir da versão 7 houve um
aumento de quase 40% de performance em geral (em alguns casos, o processamento acaba sendo até 2x mais rápido).
Aliás, vale dizer que MUITAS das críticas que o PHP ainda recebe são relacionadas à versão 4 ou início da 5, de coisas que já foram resolvidas há tempos.

Falando sobre performance: a linguagem recebeu VÁRIAS otimizações internas, baixo-nível mesmo, que deixaram o processamento muito mais rápido e efetivo.
Eu vou listar aqui alguns dos tópicos mas, se você quiser saber um pouquinho mais e conseguir entender bem inglês, vou deixar uns links de referência no final do artigo 🙂

Basicamente, o pessoal que trabalha no código-fonte do PHP buscou melhorar a performance otimizando a forma como a linguagem utiliza a memória, trabalhando nos seguintes pontos:

- Reduzir o número de alocações (alocar e desalocar dados em memória é um processo custoso)
- Reduzir o uso da memória em geral (o acesso à memória RAM é pesado em termos de latência. O ideal seria usar as memórias cache do processador o máximo possível)
- Reduzir a quantidade de ponteiros utilizados (você tem um ponteiro que aponta para um lugar que tem um ponteiro que aponta para onde você tem que ir para conseguir o dado que você precisa)

Esse trabalho todo resultou numa melhor estruturação e utilização dos zvals. Explicando de maneira bem porcaria, um zval seria a estrutura em C que representa um dado sendo utilizado no PHP. Quando você cria uma variável, um objeto ou lida com um valor qualquer, esse dado é transformado em um "zval". É esse zval que é efetivamente manipulado lá no baixo nível.

Os arrays (que no PHP são, na verdade, hash tables) tiveram suas estruturas (do zval) simplificadas, dispensando o uso da lista duplamente ligada que era necessária nas versões anteriores, além de não utilizar mais ponteiros para ligar um dado ao próximo contido no array.

A estruturação de um objeto em zval também foi simplificada, não exigindo mais 2 processos (e ponteiros) intermediários.

Em geral, quase todas as estruturas do PHP tiveram uma diminuição tanto quantidade de alocações em memória necessárias quanto no tamanho em bytes da estrutura em si.

Além disso, o próprio processo de compilação da linguagem foi modificado, adicionando um passo extra entre a tokenização e a compilação: o parsing passou a ser feito separadamente. Isso abriu espaço para implementação de muitas funcionalidades que antes não eram possíveis por conta do processo "simplista" entre a fase de tokenização e compilação.

Sobre as novas funcionalidades, bom... não vou falar delas aqui porque já existem muitos bons artigos em português sobre, como esse post da Caelum aqui: <https://blog.caelum.com.br/novidades-do-php-7/> ou essa ótima palestra do Duodraco <https://www.youtube.com/watch?v=r260ueWFq2M&list=PLnYXGcDMOP_FarvCEYFKKOC5KoIjw8gIU> (valeu, Marcel, pela dica) :)

Espero que esse texto tenha ajudado a esclarecer um pouco das dúvidas em relação a esse aumento bruto de performance.
Qualquer coisa, me manda um salve lá no twitter ou instagram: @dianaarnos

Referências:

- Uma ótima palestra do Nikita Popov sobre as mudanças internas feitas no PHP7:
  <https://www.youtube.com/watch?v=zekEqhaPmag>
- Sobre zvals: <http://www.phpinternalsbook.com/php5/zvals/basic_structure.html>