+++
comments = true
date = 2018-05-31
title = "Autorização em APIs ASP.Net Core"
description = "Aprenda a autorizar o acesso à suas APIs"
tags = ["aspnet", "aspnetcore", "jwt", "auth", "authorization"]
categories = ["Desenvolvimento", "ASP.Net"]
nomenu = "main"
image = "https://cdn-images-1.medium.com/max/12000/1*bfOAJYIfkbxZ5SmrMLyJig.jpeg"
+++



Em um [artigo anterior](https://www.wellingtonjhn.com/posts/autentica%C3%A7%C3%A3o-em-apis-asp.net-core-com-jwt) eu falei sobre a criação de uma API de Autenticação utilizando ASP.Net Core. Hoje quero falar um pouco sobre como podemos fazer a Autorização de nossos usuários dentro das APIs.

Sendo que no artigo citado acima eu fiz uma breve introdução sobre os conceitos de **Autenticação** e **Autorização**, então, se você não viu corre lá e depois volte aqui.

Os recursos de autorização descritos neste artigo foram implementados a titulo de demonstração nesse projeto que está no meu [Github](https://github.com/wellingtonjhn/DemoJwt).

### Autorização baseada em Roles

Quando uma identidade de usuário é criada, ela pode ter Roles (papéis) associadas ao usuário. Por exemplo, um usuário pode ter o “papel” de **Administrador**, enquanto outro pode ter o “papel” de **Usuário**, sendo que cada um possui permissões de acesso diferentes dentro do sistema.

Usamos as Roles para autorizar o acesso do usuário à determinados recursos dentro da aplicação. Sendo essa a forma mais comum de autorização.

Para autorizar o acesso dos usuários em nossas APIs, seja usando Roles ou Policies, conforme veremos mais adiante, devemos fazer uso de um atributo chamado **AuthorizeAttribute**. Para isso basta adicionar esse atributo na Controller ou Action que queremos proteger.

{{< gist d6c9370eddb18e238fd4c8f9f639ea26 >}} 

No exemplo acima, o acesso à essa Controller será permitido apenas aos usuários que possuem a role *Administrator*. Usuários que não possuem essa role receberão um erro (403 Forbidden) ao tentar acessar esse recurso.

Abaixo temos o exemplo do payload de um token JWT com uma role User.

![JWT com a role User](https://cdn-images-1.medium.com/max/2000/1*5cd-oFidieqQ_5HNWbN4oQ.png)

Ao chamar a API no Postman passando esse token JWT no header *Authorization*, o acesso é negado, pois o usuário não possui a role Administrator que é requerida para acessar essa Controller.

![Acesso negado à API com HTTP Status Code 403 Forbidden](https://cdn-images-1.medium.com/max/2252/1*dCi4ZmdhAea4XxRWMOBmaA.png)

Já neste exemplo, temos o payload de um usuário com role de administração, no caso Administrator.

![JWT com a role Administrator](https://cdn-images-1.medium.com/max/2000/1*6AV9lEmCkgpdUDlwrLnA_Q.png)

Ao chamar a mesma API com esse token JWT, o acesso é permitido e uma listagem dos usúarios cadastrados é exibida.

![Acesso concedido à API com HTTP Status Code 200 OK](https://cdn-images-1.medium.com/max/2232/1*HTCzVLiYyEbp4FIfgIyBzQ.png)

Veja que o atributo Authorize deve ser colocado em cada classe que queremos autorizar o acesso, mesmo que não informemos nenhuma Role específica. Nesse caso, quando nenhuma Role é informada, apenas uma identidade de usuário válida já basta para autorizar o acesso.

Temos que concordar que adicionar esse atributo em cada Controller ou Action que queremos proteger pode se tornar algo trabalhoso e chato dependendo do tamanho da aplicação. Mas existe uma forma de aplicar a autorização de forma global para toda a API, para isso devemos fazer uso de [Filtros](https://docs.microsoft.com/pt-br/aspnet/core/mvc/controllers/filters) do ASP.Net MVC, mais precisamente do filtro chamado **AuthorizeFilter**.

{{< gist bda46b2ed9615ebba69091bb31f42b49 >}} 

Com isso, toda a API irá requerer ao menos um token válido. Nas Controllers ou Actions que você desejar acesso anônimo, use o atributo **AllowAnonymous**.

{{< gist dced5b34732220ec240e5480aecbbddd >}} 

### Autorização baseada em Policies

Outra forma de autorizar o acesso à determinados recursos é fazendo uso de **Policies**.

Uma **Policy** pode ser composta por um ou mais requirements que uma identidade de usuário deve satisfazer.

As Policies são mais flexíveis quando comparadas à Roles, e com elas podemos elaborar melhor o acesso aos recursos.

Você precisa criar um Requirement que deverá ser satisfeito durante a autorização do usuário, usando a interface **IAuthorizationRequirement**. No meu exemplo criei uma classe que representa o requirement necessário para permitir a exclusão de um usuário na aplicação.

{{< gist a0c820a1a2242c82d66034058b04f03e >}} 

Após criar o requirement, precisamos criar um handler que será responsável por validar se ele foi satisfeito, para isso fazemos uso da classe **AuthorizationHandler\<TRequirement\>** informando em sua definição o requirement que ela irá tratar.

{{< gist 74b7405703e3f55c29b716cf672fbe52 >}} 

Nesse exemplo hipotético, essa classe irá verificar se o usuário possui a role *Administrator*, e também se ele possui a permissão para exclusão de um usuário, nesse caso ele deverá possuir a claim *Permissions* com o valor *CanDeleteUser*, que será passado através da propriedade RequiredPermission do requirement conforme veremos mais adiante.

Para que tudo isso funcione você deve configurar a Policy utilizando o método **AddAuthorization** da interface **IServiceCollection**.

{{< gist 818144a4625bcad602d110c5e3dfb351 >}} 

Você deve informar um nome para a policy, aqui ela se chama **DeleteUserPolicy**, e então adicionar o requirement que quer validar, em meu exemplo temos o **DeleteUserRequirement**. Também deve ser informado o valor esperado para esse requirement que é **CanDeleteUser**, nesse caso o usuário precisa ter esse valor na claim **Permissions** caso contrário o requirement não será satisfeito.

Além disso, também é necessário injetar o handler **DeleteUserRequirementHandler** no containder de DI.

Com tudo isso pronto, basta utilizar novamente o atributo *Authorize*, informando o nome da Policy que queremos utilizar, nesse caso **DeleteUserPolicy**.

{{< gist 23be96bf91632833090e7dac2f519bbd >}} 

Somente usuários que possuem a role *Administrator*, e que possuem a permissão adequada, neste caso *CanDeleteUser*, conseguirão acessar a action **DeleteAccount**.

Abaixo temos o exemplo de um payload JWT em que o usuário possui a role *Administrator*, e a permissão *CanDeleteUser*.

![JWT com role Administrator e claim Permissions com valor CanDeleteUser](https://cdn-images-1.medium.com/max/2000/1*QD5C4jPy-HIgX5CofZI8xw.png)

### Authorization Service

Além do atributo **Authorize** que vimos anteriormente, podemos usar a interface **IAuthorizationService** manualmente. Ela é utilizada internamente pelos filtros de autorização do ASP.Net Core.

Você pode injetar essa interface no construtor da classe onde quer autorizar o acesso e fazer uso do método **AuthorizeAsync**. Após ser chamado, esse método irá disparar o handler associado ao requirement informado, da mesma forma como o atributo **Authorize** faria.

{{< gist 0c25e94c515a10e6929a60777b947916 >}} 

### Conclusão

Roles e Policies representam a forma mais comum e segura de autorizar o acesso à determinado recurso em nossas aplicações, e fazendo o correto uso deles podemos customizar o acesso à nossas APIs de forma simples.

Espero que tenham gostado e se tiverem dúvidas, críticas ou sugestões entrem em contato.

Abraços!

### Referências

* [Autorização no ASP.Net Core](https://docs.microsoft.com/en-us/aspnet/core/security/authorization/?view=aspnetcore-2.1)
* [Filtros do ASP.Net MVC](https://docs.microsoft.com/pt-br/aspnet/core/mvc/controllers/filters)

Foto da capa por [Chris Barbalis](https://unsplash.com/photos/bchywV6UEbE?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) em [Unsplash](https://unsplash.com/search/photos/lock?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)