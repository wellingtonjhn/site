+++
comments = true
date = 2018-04-09
title = "Fail-fast Validations com Pipeline Behavior no MediatR e ASP.Net Core"
description = "Veja como implementar Fail-Fast Validations com MediatR em uma aplicação ASP.Net Core"
tags = ["aspnet","aspnetcore", "mediatr", "design patterns"]
categories = ["Desenvolvimento", "ASP.Net"]
nomenu = "main"
image = ""
+++

Sabemos que o MediatR simplifica muito o design de nossas aplicações tornando nosso código mais simples e com baixo acoplamento, conforme eu mostrei no [artigo anterior](https://www.wellingtonjhn.com/posts/mediatr-com-asp.net-core/). Se você não viu corre lá e depois volte aqui.

Um dos recursos mais legais dele é a possibilidade de executarmos determinadas ações antes ou depois de um Handler ser disparado. Esse recurso leva o nome de **Pipeline Behavior**.

Você pode usar o Pipeline Behavior para, por exemplo, validar os parâmetros de entrada dos Requests, autorizar o acesso à determinado Handler, gravar log dos Requests, etc.

> Lembre-se do artigo anterior, quando eu me refiro à Request no MediatR não estou falando de uma requisição HTTP, mas sim da “mensagem” que é enviada para o MediatR, e que está associada a determinado Handler.

### Como funciona?

O Pipeline Behavior é bem parecido com o pipeline de execução do ASP.Net Core, onde temos middlewares sendo executados sequencialmente.

![](https://cdn-images-1.medium.com/max/2000/1*9QeDOotCpo5eboOd0GryhA.png)

Para usar esse recurso do MediatR você deve criar uma classe que implementa a interface **IPipelineBehavior\<TRequest, TResponse\>**. Nessa classe iremos definir o comportamento que desejamos.

{{< gist e7c07f8116b47247592c88cb9d831b38 >}}

Veja que um dos parâmetros do Pipeline Behavior é um Request. Além disso, ele usa o método **next()** para disparar o próximo passo do pipeline, que normalmente é a execução do respectivo Handler.

Após implementar a classe você deve registrá-la no container de DI de sua preferência. No meu caso irei usar o container nativo do ASP.Net Core mais adiante.

Quando um Request for enviado para o MediatR, os Behaviors serão executados na sequência em que foram definidos no container de DI.

### Fail-Fast Validations

Em nosso exemplo iremos implementar um Behavior para validar os parâmetros de entrada dos Requests antes de seus respectivos Handlers serem disparados, dessa forma antecipamos a falha no Request e poupamos nossa aplicação de processar um Handler para descobrir que os parâmetros não são válidos.

> Normalmente a validação dos Requests é feita na entrada do Handler ou nas Actions dentro das Controllers da API, com chamadas para validar os inputs de dados. Isso torna o código um pouco sujo na minha opinião. A implementação do Behavior torna essa tarefa automática.

Para implementar esse Behavior usaremos o MediatR em conjunto com a biblioteca [FluentValidation](https://github.com/JeremySkinner/FluentValidation) que acredito ser uma das mais conhecidas e utilizadas mundialmente quando falamos de validações. Caso você queira usar outra biblioteca de Notification Pattern nas validações fique à vontade, as alterações seriam mínimas. Recomendo você dar uma olhada no [Flunt](https://github.com/andrebaltieri/flunt).

### Implementação

A classe a seguir representa nosso Behavior responsável por interceptar todos os Requests.

Veja que injetamos em nosso Behavior todos os validadores do **FluentValidation**. No método **Handle** executamos os validadores e caso existam falhas apontadas por eles, interrompemos o pipeline e então retornamos essas mensagens de erro.

{{< gist eaa780f2b3d0683bba5671602cf0a279 >}}

A seguir temos um Request e seu respectivo Validator que irá garantir que os parâmetros do Request estejam em conformidade.

{{< gist 8c303fd24c792fc23e8ea77ead15c3dc >}}

Na classe **Startup** de nossa API temos o método **AddMediatr**, que é responsável por fazer a devida injeção de dependências do MediatR além dos validadores de Requests.

{{< gist df75023e50bf4d3f1b6c5122a5bba3b4 >}}

Na controller apenas enviamos o Request para o MediatR e então recuperamos seu retorno. Caso ele retorne mensagens de erro, elas são exibidas com o Http Status Code 400 (Bad Request).

{{< gist 0dec296770cf381b803e5c48852f3f72 >}}

Podemos fazer os testes de execução da API com o [Postman](https://www.getpostman.com/).

![Chamada com todas as mensagens de validação do Request](https://cdn-images-1.medium.com/max/2000/1*ftS5Jizsaoamyp04bDZ76w.png)

![Mensagem de e-mail inválido](https://cdn-images-1.medium.com/max/2000/1*RsfOYEBlRY_0HZ3LZkBBjw.png)

![Chamada de Sucesso na API](https://cdn-images-1.medium.com/max/2000/1*zW_4IZX1ylmP8dAkAFLWog.png)

### Conclusão

Você deve ter percebido que as mensagens são cumulativas, então podemos validar todos os parâmetros e retornar as falhas de uma única vez.

Muitas APIs por aí fazem a validação campo à campo, disparando uma Exception para cada propriedade inválida. Essa abordagem é muito ruim na minha opinião, pois além de interromper o fluxo para cada propriedade inválida ainda existe o lançamento de Exception que é muito oneroso para a aplicação.

Usando uma abordagem de *Fail-fast com Notification Pattern*, nosso código fica mais limpo e as validações dos Requests, caso existam, serão executadas automaticamente dentro do pipeline do MediatR antes que seus respectivos Handlers sejam disparados.

Quaisquer dúvidas ou sugestões, por favor entrem em contato.

Abraços!!

> Não deixe de conferir o código fonte para essa demo que está no meu [Github](https://github.com/wellingtonjhn/DemoMediatR).

### Referências

* [Documentação do Pipeline Behavior do MediatR](https://github.com/jbogard/MediatR/wiki/Behaviors)
* [Documentação do Fluent Validation](https://github.com/JeremySkinner/FluentValidation/wiki)
* [Notification Pattern](https://martinfowler.com/eaaDev/Notification.html)
