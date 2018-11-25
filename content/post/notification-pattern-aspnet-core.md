+++
comments = true
date = 2018-11-25
title = "Não lance Exceptions em seu Domínio… Use Notifications!"
description = "Veja como deixar de usar Exceptions em seu domínio com Notification Pattern"
tags = ["aspnet","aspnetcore", "notification pattern", "design patterns"]
categories = ["Desenvolvimento", "ASP.Net"]
nomenu = "main"
image = "https://cdn-images-1.medium.com/max/6400/1*1JRIBJZtMRT8U6NfoDhbcA.jpeg"
+++

É muito comum encontrar nos sistemas corporativos o lançamento de Exceptions ao realizar validações de regras de negócio, afinal de contas é necessário informar ao usuário que algo deu errado.

{{< gist 0dfa933aca1684e6fde0592619b215c6 >}}

Porém, o que muitos desenvolvedores esquecem é que exceptions são inesperadas e elas indicam uma falha, uma exceção ao funcionamento normal do sistema.

Uma validação de regra de negócio é esperada e normal em praticamente todos os sistemas, acontecem à todo momento, não são falhas para serem tratadas como exceptions.

O lançamento de exceptions no domínio traz alguns problemas:

* Interrupção do fluxo de execução para cada inconsistência encontrada durante as verificações de regras de negócio;
* São onerosas para o processador;
* São deselegantes.

Uma abordagem superior seria usar **Notifications**!

>  Você pode conferir os exemplos de código em meu [Github](https://github.com/wellingtonjhn/DemoNotifications).

### Notification Pattern

Para capturar as mensagens das validações de domínio podemos usar o pattern Notification ou Domain Notification como também é conhecido, que foi descrito por Martin Folwer em um [artigo](https://martinfowler.com/eaaDev/Notification.html) de 2004, que também mostra um modelo de implementação em C#.

Basicamente esse pattern nos ajuda a levar mensagens de domínio para a camada de apresentação, como por exemplo erros ou mensagens de validação de negócio, já que normalmente a camada de apresentação não possui nenhum acesso direto à camada de domínio.

![Arquitetura em camadas padrão aplicada ao DDD (Domain Driven Design)](https://cdn-images-1.medium.com/max/2774/0*7KfYRkjst3l6Di1p.jpg)

>  Existem variações ao desenho de camadas mostrado acima, eu mesmo uso modelos diferentes dependendo da aplicação, mas normalmente em nenhuma delas a camada de apresentação acessa o domínio diretamente, se sua aplicação faz isso, reveja seu desenho arquitetural.

### Implementando Notification Pattern no ASP.Net Core

Existem variações em sua implementação, mas basicamente uma notificação pode ser representada como um objeto que encapsula uma mensagem gerada pelo domínio, mas também pode ser representada como algo mais simples, uma coleção de strings por exemplo.

Em nosso cenário iremos representar as notificações como uma coleção de **Notification** na classe **NotificationContext**, conforme veremos a seguir.

{{< gist 427bf809fd05b0703927bbdb6808f452 >}}

Além da classe **Notification**, que é uma estrutura bem simples e serve apenas pare representar uma mensagem, também temos a classe **NotificationContext**, que é responsável por armazenar as notificações através da propriedade **Notifications**. Além da coleção de notificações, temos a propriedade **HasNotifications** que serve apenas para informar se existem notificações no contexto.

Além dessas propriedades também temos alguns métodos auxiliares que servem para adicionar mensagens ao contexto de notificações.

No domínio temos uma classe base chamada **Entity** que possui um método de valiação genérico e algumas propriedades que indicam o estado da entidade.

{{< gist bd7d786b975b09b0e258607f1b0f50f5 >}}

Como exemplo, criei uma entidade **Customer** que recebe alguns parâmetros em seu construtor. 

Como já sabemos, em um domínio rico as próprias entidades possuem a responsabilidade de se validarem, então, no construtor o método **Validate** da classe base é chamado onde passamos como parâmetro uma instancia da própria entidade e sua definição de validações feita com o **FluentValidation**.

Essa entidade expõe as mensagens de erro das validações através da propriedade **ValidationResult** da classe base.

>  Nesse exemplo de domínio eu estou usando a biblioteca [FluentValidation](https://github.com/JeremySkinner/FluentValidation) para fazer as validações, mas você pode usar o que achar melhor.

Geralmente também criamos um **ApplicationService** que será responsável por orquestrar as interações entre a apresentação, domínio e repositório, mas como vocês já sabem, eu gosto de usar o MediatR em meus projetos e acabo não criando classes de serviço na camada de Application conforme já falei em um [artigo](https://medium.com/@wellingotnjhn/mediatr-com-asp-net-core-7b98ba0ca640) anterior. =)

Nesse caso os ApplicationServices são substituídos por Handlers do MediatR. Com isso recebemos um request/command e fazemos toda a orquestração para um “use case" específico.

{{< gist 3be2dd6b2b4a6ffade6180a36ec762fd >}}

No método Handle, após instanciar a entidade Customer ela irá armazenar o resultado de suas validações na propriedade **ValidationResult**.

Ao verificar que a entidade está inválida podemos adicionar suas mensagens de validação no contexto de notificações que foi injetado no construtor do Handler. Isso nos permite acumular as falhas e exibir todas ao mesmo tempo para o usuário. Com a entidade inválida não faz sentido avançar, então podemos interromper o fluxo de execução com um simples **return**.

>  Perceba que interrompemos o fluxo de execução apenas uma vez, após todas a validações terem sido feitas e apenas para notificar as mensagens ao usuário.

Ao fazer essa interrupção o fluxo volta para a controller que originou o request para o Handler.

{{< gist 8bbda8cfc60c7dee8f4ccfd49efbe6a8 >}}

Veja que a action está limpa, ela só envia o request para o MediatR e devolve o resultado para o client. Simples assim!

Mas e as notificações?

Elas são capturadas por um filtro global chamado **NotificationFilter** que implementa a interface **IAsyncResultFilter**. Isso quer dizer que esse filtro será invocado automaticamente, após a action da controller gerar um resultado válido, e antes do retorno para o client. Com isso podemos interceptar o resultado da action e formatá-lo conforme o necessário.

Para mais informações sobre como os filtros funcionam, veja a [documentação oficial](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/filters?view=aspnetcore-2.1).

{{< gist 5d2598eb835964571769a6b06956c01f >}}

Basicamente verificamos se existem notificações através da propriedade **HasNotifications** do **NotificationContext**. Se existirem mensagens mudamos o Http Status Code para **Bad Request (400)** e retornamos a lista de notificações em formato Json.

Se você não quiser usar filtros, pode criar um objeto que encapsula os resultados da sua camada Application adicionando as mensagens de notificação ou o resultado da operação nele. Entretanto, será necessário verificar se existem notificações na controller e gerar o Bad Request em cada action.

### Testes

Ao tentar fazer uma chamada para a nossa API com os dados do Customer inválidos, as mensagens de validação são exibidas todas de uma vez.

![](https://cdn-images-1.medium.com/max/2000/1*NDqC7zArtayss-ZNXe2QQg.png)

### Conclusão 

O uso de padrões como Domain Notifications traz uma certa flexibilidade aos nossos sistemas e ajuda a manter nosso código mais simples e elegante, além de não impactar a performance da aplicação com o lançamento de exceptions desnecessárias.

Existem diversas formas de implementar o Notification Pattern, sendo a que vimos aqui uma delas. Outra implementação que gosto bastante é o [Flunt](https://github.com/andrebaltieri/flunt), uma biblioteca idelizada e mantida pelo Microsoft MVP André Baltieri em conjunto com a comunidade.

Espero que tenham gostado desse artigo. Não deixem de deixar seus comentários, dúvidas ou sugestões.

Abraços!

### Referências 

* [Notifications by Martin Fowler](https://martinfowler.com/eaaDev/Notification.html)
* [ASP.Net Core Filters](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/filters?view=aspnetcore-2.1)
