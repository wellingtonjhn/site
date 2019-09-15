+++
comments = true
date = 2018-05-12
title = "Obtendo o Usuário Logado em APIs ASP.Net Core"
description = "Veja como trabalhar com o usuário logado em uma API ASP.Net Core"
tags = ["aspnet", "aspnetcore", "jwt", "token", "auth", "json web token"]
categories = ["Desenvolvimento", "ASP.Net"]
nomenu = "main"
image = "https://cdn-images-1.medium.com/max/9000/1*ctkWx-g2uUsUhQOCmUWq5Q.jpeg"
+++

No [artigo anterior](https://www.wellingtonjhn.com/posts/autentica%C3%A7%C3%A3o-em-apis-asp.net-core-com-jwt/) eu mostrei como criar uma API de autenticação em ASP.Net Core com JWT. Hoje iremos ver como podemos obter o usuário autenticado, extraindo os dados do token de uma forma muito simples.

Para isso o ASP.Net Core oferece uma biblioteca de abstrações HTTP (pacote nuget [Microsoft.AspNetCore.Http.Abstractions](https://www.nuget.org/packages/Microsoft.AspNetCore.Http.Abstractions/2.1.0-rc1-final)) que contém a interface **IHttpContextAccessor**.

A classe **HttpContextAccessor** implementa tal interface, e possui uma propriedade onde podemos obter o **HttpContext** da requisição e com ele a identidade do usuário logado. Ela deve ser registrada no contêiner de DI como **Singleton**.

Para simplificar as coisas, podemos criar uma classe que representa nosso usuário logado na aplicação, em meu caso ela se chama **AuthenticatedUser**, e recebe em seu construtor uma instância de **IHttpContextAccessor**.

{{< gist 17b2972d1e3de5d6324602267c7cdf1f >}} 

Também devemos fazer o registro dessa classe no contêiner de DI, e ela deve ser utilizada onde for necessário obter o usuário logado.

{{< gist c1d158376291764a6efa82553f33aa0a >}} 

> O HttpContext será recriado à cada nova requisição HTTP na aplicação.

Com isso podemos injetar um **AuthenticatedUser** em qualquer classe que precisarmos dele.

![](https://cdn-images-1.medium.com/max/2000/1*ezhfYijWeoaRVXFB0rWHQA.png)

### Conclusão

Fazendo uso do **HttpContextAccessor** em conjunto com a injeção de dependências, temos um jeito fácil de obter o usuário autenticado e utilizá-lo em qualquer camada de nossa aplicação conforme a necessidade. Em versões anteriores do ASP.Net isso também era possível, porém um pouco mais trabalhoso. Com isso temos á nossa disposição um recurso muito poderoso que nos dá muita flexibilidade ao se trabalhar com o contexto da requisição HTTP em nossas aplicações.

Espero que tenham gostado e se ficou alguma dúvida, ou tenham críticas e sugestões entrem em contato.

Abraços!

Foto por [Nicole Harrington](https://unsplash.com/photos/gMJ3tFOLvnA?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) em [Unsplash](https://unsplash.com/search/photos/document?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
