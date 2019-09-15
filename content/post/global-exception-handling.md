---
comments : true
date : 2018-11-05
title : "Tratamento Global de Exceptions no ASP.Net Core"
description : "Aprenda a capturar todas as Exceptions de sua aplicação"
tags : ["aspnet","aspnetcore", "global-exception-handling", "exceptions"]
categories : ["Desenvolvimento", "ASP.Net"]
nomenu : "main"
image : "https://cdn-images-1.medium.com/max/2000/0*f8VaTl-csq_uHv4b.png"
---

Desde as primeiras versões do C# temos à nossa disposição o famoso bloco **try…catch**, onde podemos capturar exceções e tratá-las da melhor forma possível, seja gravando um log, adicionando uma mensagem amigável para o usuário, etc. Porém, muitas vezes não conseguimos prever todos os possíveis erros que possam acontecer e muitas exceções acabam “explodindo” na tela para o usuário, ou pior, causam a queda de nossos sistemas e até mesmo prejuízos financeiros para nossos clientes.

Uma forma de prevenir que ocorram exceptions não tratadas é fazendo o seu gerenciamento de forma global, assim podemos ter um local centralizado onde todas as exceções são capturadas, facilitando a manutenção e deixando nossas classes mais limpas e legíveis, sem a necessidade de usar o bloco **try…catch** em cada método de seu sistema.
>  O uso exagerado do bloco **try…catch** pode tornar o código mais verboso e sua leitura difícil, tenha bom senso, você pode usá-lo onde realmente irá tratar a exception, gerar um log específico, tomar uma ação efetiva, etc… caso contrário, deixe a exception subir a stack e ser capturada pelo filtro global.

Existem diversas formas de se tratar as exceptions de forma global, seja usando Middlewares, Action Filters, ferramentas específicas de AOP (Aspect-Oriented Programming), entre outros. Hoje veremos como fazer isso com Middlewares.

Lembrando que o código fonte de demonstração está em meu [Github](https://github.com/wellingtonjhn/DemoGlobalExceptionHandling).

### O que são Middlewares?

Antes de mais nada, é preciso entender o que são os famosos Middlewares do ASP.Net Core.

Os Middlewares basicamente são componentes que **definem o pipeline de execução dos requests HTTP**, ou seja, todos os passos que seu request faz dentro da aplicação desde a sua recepção até a resposta. Eles são encadeados, então, um middleware pode executar outro através de delegates repassando o mesmo contexto da requisição. Quando uma resposta é gerada em algum passo dentro do pipeline, a execução volta para o  passo anterior e assim em diante. Você pode criar trechos de código que executam antes ou depois do próximo passo do pipeline. Eles são configurados em ordem no método Configure da classe Startup, através da interface **IApplicationBuilder**.

![](https://cdn-images-1.medium.com/max/2000/0*3Flzz4VWl30if5Q5)

Agora que você já sabe o que são Middlewares, podemos criar nosso componente de tratamento centralizado de exceptions.

Para mais informações sobre a criação de Middlewares no ASP.Net Core recomendo a leitura da [documentação oficial](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/?view=aspnetcore-2.1).

### Middleware Nativo do ASP.Net Core

O ASP.Net Core possui um Middleware nativo para tratamento de exceptions, que pode ser configurado através do método **UseExceptionHandler**.

{{< gist c18b2922b886b75ba6326e2c2a52b756 >}}

Nesse exemplo, o que fizemos foi extender o comportamento padrão do método capturando a exception gerada, então gravamos um log de erro e retornamos uma mensagem para o usuário com HTTP Status Code 500 (Internal Server Error) em formato JSON. Veja que recebemos uma instância de **ILoggerFactory** via parâmetro do método como dependência para geração do log de erros.
>  Parte da mensagem que será retornada para o client é a própria exception através do campo Detailed. Eu recomendo não retornar os detalhes da exception em uma aplicação real de produção, pois um usuário mal intencionado pode ver esses detalhes e descobrir brechas para atacar seu sistema. Em ambiente de desenvolvimento não vejo nada de errado em mostrar os detalhes das exceptions, até mesmo para ajudar a resolver os possíveis problemas mais rapidamente. Você pode facilmente verificar em qual ambiente sua aplicação está rodando usando a interface **IHostingEnvironment**.

Para que seu método de extensão tenha efeito, será necessário chamá-lo na classe Startup dentro do método Configure.

### Middleware Customizado

Com a interface **IMiddleware**, você pode facilmente criar seu próprio Middleware customizado. O uso dessa interface implica que seu middleware será do tipo **Factory-based**, sendo necessário também fazer sua configuração no container de DI, conforme veremos mais a seguir. 

{{< gist 39f5391a5dcf0cf3e7087765b9a806c1 >}}

Nosso middleware recebe em seu construtor uma dependência de **ILogger** para a geração de logs de erro, diferentemente do exemplo anterior, que recebia a dependência de log via parâmetro no método UseGlobalExceptionHandler. 

O método **InvokeAsync** será chamado automaticamente, nele existe uma chamada ao método next(context) que irá executar o próximo passo do pipeline, ele está contido em um bloco **try…catch**. Caso alguma exception seja lançada ela será capturada e enviada para o método **HandleExceptionAsync**, onde retornamos uma mensagem JSON para o client, exatamente como no exemplo anterior.

Algumas alterações também devem ser feitas na classe Startup. Para simplificar criei alguns métodos de extensão na classe **GlobalExceptionHandlerMiddlewareExtensions**.

Como nosso middleware é ativado através da classe **MiddlewareFactory**, ele deve ser registrado no container de DI através do método **AddGlobalExceptionHandlerMiddleware**. Em seguida, basta fazer a chamada ao método **UseGlobalExceptionHandlerMiddleware** para adicioná-lo ao pipeline do ASP.Net Core. Lembre-se que o pipeline é executado na ordem em que foi definido no método Configure, então ele deve ficar antes da definição de uso do MVC.

### Controllers

Para ambos os casos, as Controllers de nossa API ficam limpas, não sendo necessário fazer uso de blocos **try…catch** conforme eu mencionei anteriormente. Todas as exceções não tratadas serão capturadas pelo mecanismo de tratamento global.

{{< gist 621adfa4f79034d71c2c7f19051d87da >}}

### Testes

Nossa API de testes contém um único endpoint chamado “ api/values” que irá retornar um valor numérico aleatório. Para simulação, eu criei uma classe auxiliar que irá gerar exceptions também de forma aleatória, então algumas chamadas à API irão funcioar e outras irão disparar uma exception.

A chamada a seguir retorna um valor numérico, com HTTP Status Code 200 indicando o sucesso da requisição.

![Chamada com sucesso — HTTP Status Cde 200](https://cdn-images-1.medium.com/max/2522/1*gaaSIisdsXRa5xeVW4VxJw.png)

O próximo exemplo mostra um objeto JSON que contém o erro ocorrido, bem como os detalhes da exception, com HTTP Status Code 500 indicando a falha na requisição.

![](https://cdn-images-1.medium.com/max/3314/1*afsmG73BfkbslD5TLw0M_w.png)

### Menção honrosa

Eu gostaria de indicar aqui o pacote nuget **[GlobalExceptionHandler](https://www.nuget.org/packages/GlobalExceptionHandler)**, criado por [Joseph Woodward](http://josephwoodward.co.uk), que também resolve o problema de tratamento global de exceções usando métodos de extensão ao middleware nativo (convention-based). A diferença é que ele possui alguns recursos bem legais de customização, como por exemplo mapeamento de exceptions para determinado HTTP Status Code, mensagens diferentes para cada exception, negociação de conteúdo, entre outros.

### Conclusão

A captura de exceções é uma parte crucial em todas as aplicações e o correto tratamento delas pode nos ajudar no rápido troubleshooting dos problemas e suas respectivas correções. Ignorar que elas existem ou não fazer um tratamento adequado delas é um problema sério que existe em muitas aplicações corporativas hoje em dia. Não pense que sua aplicação estará livre delas, pois como seu nome sugere, exceções ao comportamento esperado do seu sistema podem e irão ocorrer em algum momento, esteja preparado para isso e seja feliz.

### Referências

* [Documentação Oficial Microsoft — Error Handling](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/error-handling?view=aspnetcore-2.1)
* [Documentação Oficial Microsoft — Middlewares](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/?view=aspnetcore-2.1)

### UPDATE — 16/11/2018

Existe uma especificação formal que basicamente define um formato padrão de mensagens de erro nas respostas de APIs HTTP. Esse padrão é conhecido como **Problem Details** e foi implementado no ASP.Net Core 2.1.

Pretendo cobrí-lo em um artigo posterior como um complemento à este, já que a implementação é bem parecida ao que fizemos aqui.

[RFC7807 — Problem Details for HTTP APIs](https://tools.ietf.org/html/rfc7807)

### UPDATE — 19/01/2019

Conforme prometi, escrevi um artigo que trata da implementação do padrão Problem Details (RFC 7807).

[Padronização de Respostas de Erro em APIs com Problem Details](https://www.wellingtonjhn.com/posts/padroniza%C3%A7%C3%A3o-de-respostas-de-erro-em-apis-com-problem-details/)