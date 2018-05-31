+++
comments = true
date = 2018-03-30
title = "MediatR com ASP.Net Core"
description = "Aprenda a usar o MediatR em uma aplicação ASP.Net Core"
tags = ["aspnet","aspnetcore", "mediatr", "design patterns"]
categories = ["Desenvolvimento", "ASP.Net"]
nomenu = "main"
image = "https://cdn-images-1.medium.com/max/2000/1*KzQbnYDWP1qScTRtfBeOSQ.jpeg"
+++

O Mediator é um padrão de projeto Comportamental criado pelo GoF, que nos ajuda a garantir um baixo acoplamento entre os objetos de nossa aplicação. Ele permite que um objeto se comunique com outros sem saber suas estruturas. Para isso devemos definir um ponto central que irá encapsular como os objetos irão se comunicar uns com os outros.

Neste artigo iremos ver como podemos criar uma API ASP.Net Core que faz uso desse padrão de projeto usando a biblioteca [MediatR](https://github.com/jbogard/MediatR).

### MediatR

O MediatR foi criado por [Jimmy Bogard](https://jimmybogard.com/), o mesmo criador do famoso [AutoMapper](https://github.com/jbogard/AutoMapper). Ele implementa o padrão de projeto Mediator de uma forma bem simples.

Para instalar em seu projeto execute o seguinte comando:

    dotnet add package MediatR

Basicamente temos dois componentes principais chamados de Request e Handler, que implementamos através das interfaces **IRequest** e **IRequestHandler<TRequest>** respectivamente.

* Request → mensagem que será processada.

* Handler → responsável por processar determinada(s) mensagen(s).

> Não confunda o Request do MediatR com um request HTTP. Request é o nome usado pelo MediatR para descrever uma mensagem que será processada por um Handler. Além disso, algumas literaturas usam o termo Command para descrever essas mensagens, eu mesmo ainda uso esse termo de vez em quando.

{{< gist 65d3a95bab6f6e29d41b508f7b085d0c >}}

> Veja que a implementação acima apenas executa determinada tarefa e não tem nenhum retorno.

Podemos ter um Request que irá devolver uma resposta para quem o invocou. No caso devemos implementar um Request que tem uma resposta associada à ele usando a interface **IRequest<TResponse>**.

{{< gist 9f901e1a931e9e99050688673f75ee09 >}}

Um Request normalmente contém propriedades que são usadas para fazer o input dos dados para os Handlers.

Esses dois componentes não fazem nada sozinhos. Precisamos de um intermediador, que será responsável por receber um Request e invocar o Handler associado á ele. Para isso temos um componente chamado **Mediator** que implementa a interface **IMediator**, por onde deveremos interagir com as demais classes.

> A classe Mediator já está implementada e não precisamos nos preocupar com ela.

Usando a interface **IMediator** nossas classes não irão saber quem ou quais componentes irão realizar determinada ação. Apenas enviamos para o Mediator e ele irá se encarregar de chamar a classe que irá executar o que precisamos. Simples assim! ;)

Com isso temos um baixo acoplamento e fácil manutenção. Cada Handler normalmente irá tratar um único Request e assim podemos ter classes menores e mais simples. Entretanto, nada impede de você definir mais de um Request para um Handler, basta implementar mais uma interface **IRequestHandler** e fazer a correta injeção de dependências para isso.

As primeiras versões do MediatR eram totalmente assíncronas, mas nas novas versões temos a possibilidade de omitir o CancelationToken caso não seja necessário, ou ainda, criar Handlers que são processados de forma síncrona. Também podemos fazer a publicação de mensagens que devem ser processadas por vários Handlers simultâneamente, esse recurso leva o nome de Notifications. Além disso, o MediatR também tem um mecanismo para interceptar a execução de determinado Handler chamado Pipeline Behavior, que irei abordar em um próximo artigo.

Podemos perceber que o MediatR é bem completo, e é uma ferramenta poderosa que pode nos ajudar muito na criação de aplicações modernas, e pode ser usado em conjunto com padrões como o CQRS (Command Query Responsibility Segregation).

Para mais detalhes, não deixe de conferir a [documentação oficial](https://github.com/jbogard/MediatR/wiki) que está disponível no [repositório do MediatR](https://github.com/jbogard/MediatR).

### Usando Mediator em Controllers do ASP.Net Core

Eu gosto muito de usar o Mediator para manter minhas Controllers limpas. Basicamente elas recebem os requests HTTP e executam o Mediator. Normalmente já recebo o objeto de Request do MediatR como parâmetro das Actions em minhas Controllers, dessa forma o próprio ASP.Net irá fazer o [Model Binding](https://docs.microsoft.com/en-us/aspnet/core/mvc/models/model-binding) apropriado para meu objeto de Request do MediatR.

{{< gist 3686dc5c03b2eefc97a2b30129de3f80 >}}

### Injeção de Dependências

Para que tudo isso funcione, você deve fazer a correta injeção de dependências do MediatR em sua aplicação. Neste ponto, vamos fazer uso do pacote Nuget **MediatR.Extensions.Microsoft.DependencyInjection**.

    dotnet add package MediatR.Extensions.Microsoft.DependencyInjection

Com o pacote devidamente instalado em seu projeto basta fazer a injeção de dependências. Neste exemplo eu uso o injetor de dependências nativo do ASP.Net Core.

{{< gist ada390eb9185741b2892f77ea3206aca >}}

No primeiro exemplo, apenas executamos o método de extensão AddMediatR, que foi disponibilizado com a instalação do pacote acima. Esse método irá injetar todas as dependências necessárias para que o MediatR funcione corretamente, inclusive os Handlers.

O segundo exemplo faz a mesma coisa que o primeiro, com a diferença que podemos informar em qual assembly estão os Handlers.

Em um próximo artigo irei mostrar como usar o recurso de Pipeline Behavior do MediatR.

Caso tenham quaisquer dúvidas, sugestões ou críticas, não deixem de entrar em contato. Ficarei feliz em responder.

Abraços!

### Referências

* [Repositório oficial do MediatR no Github](https://github.com/jbogard/MediatR)
* [Pacote Nuget do MediatR](https://www.nuget.org/packages/MediatR)
* [Pacote Nuget MediatR.Extensions.Microsoft.DependencyInjection](https://www.nuget.org/packages/MediatR.Extensions.Microsoft.DependencyInjection/)
* [Mediator Design Pattern](http://www.dofactory.com/net/mediator-design-pattern)
* [GoF Design Patterns](http://www.dofactory.com/net/design-patterns)
* [Livro Design Patterns: Elements of Reusable Object-Oriented Software](https://www.amazon.com.br/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612)
