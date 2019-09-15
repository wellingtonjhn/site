---
comments : true
date : 2019-01-19
title : "Padronização de Respostas de Erro em APIs com Problem Details"
description : "Veja como implementar a RFC 7808 em APIs ASP.Net Core"
tags : ["aspnet","aspnetcore", "problem details pattern", "design patterns"]
categories : ["Desenvolvimento", "ASP.Net"]
nomenu : "main"
image : "https://cdn-images-1.medium.com/max/11976/1*a5N4vokpfnlmUnnJMou9zw.jpeg"
---

Como sabemos existem centenas de milhares de APIs no mundo, e a cada dia outras milhares surgem. Cada uma delas tem o seu próprio padrão de mensagens de erro, não existindo um consenso comum.

Você pode conferir um pouco desse problema verificando as APIs do Facebook, Google e Twitter por exemplo. Cada uma delas possui seu próprio formato para mensagens de erro.

* [Google Standard Error Response](https://developers.google.com/search-ads/v2/standard-error-responses)
* [Facebook Graph API Error Response](https://developers.facebook.com/docs/graph-api/using-graph-api/error-handling)
* [Twitter API Error Response](https://developer.twitter.com/en/docs/ads/general/guides/response-codes.html)

Diante disso foi criada a **RFC 7807**, que é uma especificação que visa padronizar os formatos de mensagens de erro em APIs HTTP, para assim evitar que novos formatos sejam criados.

Basicamente ela especifica que:

* Devem ser usados os códigos de Status HTTP entre os ranges 400 e 500 para representar mensagens de erro.

* O header **Content-Type** deve ser do tipo **application/problem**, incluindo o formato de serialização da mensagem, json ou xml:

```
    application/problem+json
    application/problem+xml
```
* O payload das respostas de erro devem conter as seguintes propriedades:

```
    Title -> um breve resumo do tipo de problema. Não deve mudar para ocorrências do mesmo tipo, 
    exceto para fins de localização;

    Detail -> descrição detalhada do problema;

    Type -> uma URL para um documento que descreva o tipo do problema;

    Status -> o status HTTP gerado pelo servidor de origem. Normalmente deve ser o mesmo status 
    HTTP da resposta, e pode servir de referência para casos onde um 
    servidor proxy altera o status da resposta;
    
    Instance -> propriedade opcional, com um URI exclusivo para o erro específico, 
    que geralmente aponta para um log de erros para essa resposta.
```

Com isso em mente, hoje veremos como implementar esse padrão no ASP.Net Core e para isso irei usar um projeto já existente que faz uso de um middleware para tratamento global de exceptions, o qual foi descrito em um [artigo anterior](https://medium.com/@wellingtonjhn/tratamento-global-de-exceptions-1ad613f58dbd).

Como de costume você pode conferir o código fonte no meu [Github](https://github.com/wellingtonjhn/DemoGlobalExceptionHandling).

### Implementação

O ASP.Net Core passou a suportar essa especificação na versão 2.1 do framework, através da classe **ProblemDetails** que faz parte do assembly **Microsoft.AspNetCore.Mvc.Core**.

A classe **ProblemDetails** é uma classe muito simples que contém somente as propriedades descritas acima.

Abaixo veremos como podemos usá-la em duas situações, no tratamento global de exceptions e também na validação de input dos usuários sobrescrevendo o comportamento padrão de validação do Model State.

### a) Tratamento Global de Exceptions

Podemos usar o método de extensão nativo **UseExceptionHandler()** para capturar todas as exceptions não tratadas, conforme expliquei no artigo mencionado acima. Nesse método, basta retornar um objeto do tipo **ProblemDetails.**

Todo o código necessário foi encapsulado no método **UseProblemDetailsExceptionHandler()**, conforme podemos ver a seguir:

{{< gist fe1d2791b8aa358e11f6a785388b7777 >}}

Simples assim! Ao ser interceptada a exception terá seus detalhes encapsulados na classe ProblemDetails e retornado para o client.

Nesse ponto você pode escolher qual nível de detalhe quer exibir, pois conforme expliquei no artigo mencionado anteriormente, é preciso ter cuidado ao expor os detalhes das exceções. Para isso você pode verificar em qual ambiente seu código está sendo executado e só exibir os detalhes em ambiente de desenvolvimento por exemplo.

Além disso, basta fazer a chamada ao método **app.UseProblemDetailsExceptionHandler()** na classe **Startup** de sua API.

Em nossa API de demonstração temos um endpoint do tipo GET que retorna um valor numérico aleatório em caso de sucesso, e também pode disparar uma exception de forma aleatória para representar um erro.

Ao fazer uma chamada para essa API, podemos ver que o formato da resposta de erro está no padrão esperado:

![](https://cdn-images-1.medium.com/max/2390/1*m8bA3w9IUg-QTP4sWJfAiQ.png)

Também podemos verificar o header **Content-Type**, que agora está com o valor **application/problem+json:**

![](https://cdn-images-1.medium.com/max/3010/1*2VnGVZXcmGR384u_0wnGxQ.png)

**b) Validação do Model State**

Também podemos usar a classe **ProblemDetails** para padronizar os erros de validação do Model State, quando o usuário faz uma requisição com dados de entrada inválidos.

{{< gist 767a151c36bd021bc0451c4ac851d528 >}}

Usamos o método **services.Configure()** para alterar o comportamento padrão de nossa API, nesse caso, mais especificamente a resposta dos erros do Model State.

Para isso vamos configurar a propriedade **InvalidModelStateResponseFactory** da classe **ApiBehaviorOptions** para customizar a resposta do Model State, assim podemos retornar um **ProblemDetails** com as respectivas mensagens de erro das validações.

Lembre-se que você deve usar o atributo **ApiController** em suas controllers. Esse atributo adiciona vários recursos úteis em suas APIs, como por exemplo validação automática do Model State, inferência automática do Model Binding, não sendo mais necessário usar os atributos **FromBody** por exemplo, entre outros.

Além disso, também é necessário fazer a chamada do método **services.ConfigureProblemDetailsModelState()** na classe **Startup** após a chamada ao **services.AddMvc()**. Isso é importante pois se você chamar o método antes do AddMvc, o comportamento que você configurou será substituído para o comportamento padrão que implementa **IConfigureOptions\<ApiBehaviorOptions\>.**

Como demonstração temos um endpoint do tipo POST que recebe um objeto do tipo **Item** que contém algumas validações feitas com **Data Annotations**. Veja que não é necessário verificar manualmente se o Model State está válido, devido ao uso do atributo **ApiController** descrito acima.

Ao fazer a chamada para esse endpoint as validações serão feitas, e caso os valores de entrada não satisfaçam as condições esperadas, as mensagens de erro serão retornadas para o client no padrão esperado.

![](https://cdn-images-1.medium.com/max/2076/1*MyFE6BUcdKOjFaEvZfCWjA.png)

Vale mencionar que além do recurso nativo do ASP.Net Core visto aqui, também existe uma implementação da RFC 7807 feita por Kristian Hellang que está disponível através do pacote nuget [Hellang.Middleware.ProblemDetails](https://www.nuget.org/packages/Hellang.Middleware.ProblemDetails).

### Conclusão 

O padrão Problem Details apesar de simples é muito flexível e poderoso e eu acredito que em um futuro próximo seu uso será cada vez mais comum, principalmente com a adoção por parte dos grandes players como é o caso da Microsoft, e ao meu ver isso é muito bom pois dessa forma teremos APIs cada vez mais padronizadas, facilitando dessa forma a integração entre sistemas e melhorando a vida dos desenvolvedores que consomem tais APIs.

Espero que tenham gostado da dica de hoje, e se possível deixem suas críticas ou sugestões. 

Abraços!

### Referências

* [RFC 7807](https://tools.ietf.org/html/rfc7807)

* [Documentação oficial da classe ProblemDetails](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.problemdetails?view=aspnetcore-2.2)

* [Documentação oficial do atributo ApiController](https://docs.microsoft.com/en-us/aspnet/core/web-api/index?view=aspnetcore-2.2#annotation-with-apicontroller-attribute)
