---
comments : true
date : 2018-11-25
title : "Não lance Exceptions em seu Domínio… Use Notifications!"
description : "Aprenda a usar o padrão Notifications em seu domínio"
tags : ["aspnet", "aspnetcore", "notification pattern", "design patterns"]
categories : ["Desenvolvimento", "ASP.Net"]
nomenu : "main"
image : "https://cdn-images-1.medium.com/max/6400/1*1JRIBJZtMRT8U6NfoDhbcA.jpeg"
---

É muito comum encontrar nos sistemas corporativos o lançamento de Exceptions ao realizar validações de regras de negócio ou entrada de dados de formulários, afinal de contas é necessário informar ao usuário que algo deu errado.

{{< gist 0dfa933aca1684e6fde0592619b215c6 >}}

Porém, o que nós geralmente esquecemos é que Exceptions são inesperadas e elas indicam um erro, uma exceção ao funcionamento normal do sistema, algo que não era para ter acontecido, mas aconteceu.

Uma validação de regra de negócio que potencialmente crie uma mensagem de erro faz parte do funcionamento normal do sistema e é perfeitamente esperado que elas ocorram, e ainda, os usuários podem e certamente vão inserir dados inválidos em seus formulários, então, para mim não faz sentido lançar Exceptions nesses casos.

>  If a failure is expected behavior, then you shouldn’t be using exceptions. (Martin Fowler)

O problema com esse modelo de validações, é que ele indica apenas a primeira inconsistência encontrada. Imagine isso em um cenário onde o usuário deve preencher vários campos em um formulário, sendo que para cada campo existe uma validação com uma Exception sendo lançada e interrompendo as validações seguintes, provavelmente o usuário deverá fazer vários requests até resolver todos os erros de validação do formulário. É mais recomendável mostrar todos os erros de validação uma única vez e assim o usuário pode corrigir tudo de uma vez antes de submeter o próximo request.

Além disso, as Exceptions tem um alto custo para serem lançadas e por isso recomenda-se que não sejam usadas para controle de fluxo ou validação de entrada de dados do usuário, conforme podemos ver na [documentação oficial](https://docs.microsoft.com/en-us/dotnet/api/system.exception?view=netframework-4.8#performance-considerations).

>  Throwing or handling an exception consumes a significant amount of system resources and execution time. **Throw exceptions only to handle truly extraordinary conditions, not to handle predictable events or flow control.** For example, in some cases, such as **when you’re developing a class library, it’s reasonable to throw an exception if a method argument is invalid, because you expect your method to be called with valid parameters**. An invalid method argument, if it is not the result of a usage error, means that something extraordinary has occurred. Conversely, **do not throw an exception if user input is invalid, because you can expect users to occasionally enter invalid data**. Instead, provide a retry mechanism so users can enter valid input.

Com isso, uma melhor abordagem para notificar mensagens da camada de domínio para o usuário seria usar **Notification Pattern**!

### Mas antes, um breve disclaimer…

Em uma primeira versão desse mesmo artigo, por descuido meu, acabei esquecendo esse importante disclaimer, além de alguns pontos do texto não terem ficado bem contextualizados segundo foi relatado, o que possívelmente abriu margem para um entendimento errado do que eu queria realmente dizer, vou tomar mais cuidado com isso nos próximos posts. Se você já leu esse artigo anteriormente e sentiu isso, peço sinceras desculpas pela interpretação errônea que meu texto passou, e convido você a refazer essa leitura.

Vale lembrar que não tenho nada contra o uso de Exceptions, também não estou tentando “endemonizar” seu uso, ou “vender um conceito errado para a comunidade” como fui “acusado” recentemente.

As **Exceptions são recursos extremamente valiosos e úteis**, são parte fundamental de várias plataformas de desenvolvimento, e além disso, eu não sou ninguém para dizer que você não deve usá-las. Pelo contrário, eu também faço uso desse importante recurso como qualquer outro desenvolvedor e você também deveria usar.

Apenas avalie cada caso, veja se realmente faz sentido lançar uma Exception para tudo que você queira dizer que está “errado”.

Além disso, eu não criei o Notification Pattern ou as boas práticas sobre o uso de Exceptions, então **te convido à ler as referências no final desse artigo**, algumas delas são bem longas, mas vale muito a pena.

Lembrando que esse artigo tem a intenção de mostrar uma das diversas formas de como podemos notificar mensagens de validações do domínio sem fazer o lançamento de Exceptions, mas isso não quer dizer que elas devem ser descartadas de sua base de código, com certeza elas são necessárias em diversos pontos do seu software.

Também não vou entrar no mérito se devemos ou não lançar Exceptions na entrada de parâmetros de métodos, afinal sabemos que é uma boa prática de programação defensiva validar os argumentos com **[Guard Clauses](http://www.stavroskasidis.com/blog/2017/tips-and-tricks-1-guard-clauses/)**, principalmente se você desenvolve bibliotecas de código.

Se você discordar de mim em qualquer ponto, sem problemas, fique à vontade em abrir uma discussão aqui nos comentários, terei o maior prazer em conversarmos e ver os prós e contras de cada abordagem.

Sim eu sei, esse disclaimer ficou longo demais, mas era preciso dizer. =)

### Notification Pattern

Para capturar as mensagens das validações de domínio podemos usar o Notification Pattern ou Domain Notifications como também é conhecido, que foi descrito por **Martin Fowler** em um [artigo](https://martinfowler.com/eaaDev/Notification.html) de 2004, onde ele também mostra um modelo de implementação em C# sem fazer lançamento de Exceptions.

Basicamente esse pattern nos ajuda a levar mensagens de domínio para a camada de apresentação, como por exemplo erros de validação ou qualquer outra mensagem que seja necessário mostrar ao usuário, já que normalmente a camada de apresentação não possui nenhum acesso direto à camada de domínio.

>  An object that collects together information about errors and other information in the domain layer and communicates it to the presentation. (Martin Fowler)

![Arquitetura em camadas padrão aplicada ao DDD (Domain Driven Design)](https://cdn-images-1.medium.com/max/2774/0*7KfYRkjst3l6Di1p.jpg)

Existem variações ao desenho de camadas mostrado acima, eu mesmo uso modelos diferentes dependendo da aplicação, mas normalmente em nenhuma delas a camada de apresentação acessa o domínio diretamente.

>  [Lembrando que o DDD não é sobre camadas, e separação em camadas não é DDD... ;)](https://www.eduardopires.net.br/2016/08/ddd-nao-e-arquitetura-em-camadas/)

### Implementando Notification Pattern no ASP.Net Core

Existem formas diferentes de implementação, mas basicamente uma notificação pode ser representada como **um objeto que encapsula uma mensagem gerada pelo domínio**, mas também pode ser representada como algo mais simples, uma coleção de strings por exemplo.

Em nosso cenário iremos representar as notificações como uma coleção de **Notification** na classe **NotificationContext**, conforme veremos a seguir.

{{< gist 427bf809fd05b0703927bbdb6808f452 >}}

Além da classe **Notification**, que tem uma estrutura bem simples e serve apenas para representar uma mensagem, também temos a classe **NotificationContext**, que é responsável por armazenar as notificações através da propriedade **Notifications** e a propriedade **HasNotifications** que serve apenas para informar se existem notificações no contexto.

Além dessas propriedades também temos alguns métodos auxiliares que servem para adicionar mensagens ao contexto de notificações.

Como cenário, vamos fazer a criação de um novo **Customer** recebendo apenas um nome e email, que posteriormente deverá ser gravado em uma base de dados, simples assim.

No domínio temos uma classe base chamada **Entity** que possui um método de validação genérico e algumas propriedades que indicam o estado da entidade.

{{< gist bd7d786b975b09b0e258607f1b0f50f5 >}}

Como já sabemos, em um domínio rico as próprias entidades possuem a responsabilidade de alterar seu estado (valor de suas propriedades), e portanto fazer suas próprias validações. Então, no construtor da classe **Customer** o método **Validate** da classe base é chamado onde passamos como parâmetro uma instância da própria entidade e sua definição de validações.

>  Lembrando que para o propósito desse artigo, não é necessário a criação de demais métodos de alteração de estado na entidade, já que apenas com o construtor conseguimos aplicar as validações necessárias para o cenário proposto e ver o Notification Pattern em funcionamento, que é o propósito desse artigo. Mas tenha em mente que em um domínio rico de verdade, além das propriedades também temos métodos de alteração de estado, onde novas validações podem ser feitas e notificações criadas, portanto o princípio é o mesmo.

Essa entidade expõe as mensagens de erro das validações através da propriedade **ValidationResult** da classe base.

>  Nesse exemplo eu estou usando a biblioteca **[FluentValidation](https://github.com/JeremySkinner/FluentValidation)**para fazer as validações, mas você pode usar o que achar melhor.

Normalmente, na camada de Application também criamos classes que orquestram a interação entre apresentação, dominio e infraestrutura, as famosas **Application Services**, mas como vocês já sabem eu gosto de usar o **MediatR** em meus projetos e acabo não criando tais classes, conforme já falei em um [artigo](https://medium.com/@wellingotnjhn/mediatr-com-asp-net-core-7b98ba0ca640) anterior. =)

Nesse caso as Application Services são substituídas por Handlers do MediatR que servem ao mesmo propósito. Com isso recebemos um request/command e fazemos toda a orquestração para um “use case" específico.

{{< gist 3be2dd6b2b4a6ffade6180a36ec762fd >}}

No método Handle da classe CreateCustomerHandler, após instanciar a entidade Customer ela irá armazenar o resultado de suas validações na propriedade **ValidationResult**.

Nesse ponto, podemos ter dois estados para a entidade, ela pode estar válida ou inválida, pois podem existir erros de validação. É muito importante ficar atento à isso já que não queremos persistir uma entidade inválida.

Nesse modelo, podemos chamar todos os métodos de alteração de estados necessários para atender ao use-case que estamos trabalhando, e em seguida **obrigatóriamente** devemos verificar seu estado final, e é exatamente isso que fazemos na linha 18 do Handler, ao verificar a integridade da entidade com a condição **“if (customer.Invalid)”** antes de persistir os dados na base de dados.

>  Eu vi esse modelo de validação onde podemos ter uma instância da entidade com estado válido ou inválido nos [cursos do MVP André Baltieri](https://balta.io) e achei bem interessante. Se você não gostar disso, não tem problema, pode fazer da forma que preferir.

Ao constatar que a entidade está inválida, podemos adicionar suas mensagens de validação no contexto de notificações que foi injetado no construtor do Handler. Isso nos permite acumular as falhas e exibir todas ao mesmo tempo para o usuário. Com a entidade inválida não faz sentido avançar, então podemos interromper o fluxo de execução com um simples **return**.

Perceba que interrompemos o fluxo de execução apenas uma vez, após todas as validações serem feitas e apenas para notificar as mensagens ao usuário. Caso as validações fossem feitas com o lançamento de exceptions, para cada inconsistência uma interrupção no fluxo seria feita para indicar essa falha específica.

>  Existem desenvolvedores que preferem lançar uma Exception após realizar as validações para bloquear o fluxo de execução com as mensagens que foram acumuladas e assim levar as mensagens para cima na stack e apresentar ao usuário. Em nosso cenário, eu acredito que um simples “return” no Handler resolve bem o problema e para mim nenhuma Exception é necessária nesse caso. De qualquer forma você deve avaliar o que for melhor para o seu cenário.

Ao fazer essa interrupção o fluxo volta para a Controller que originou o request ao Handler.

{{< gist 8bbda8cfc60c7dee8f4ccfd49efbe6a8 >}}

Veja que a action está limpa, ela só envia o request para o MediatR e devolve o resultado para o client. Simples assim!

Mas e as notificações?

Elas são capturadas por um filtro global chamado **NotificationFilter** que implementa a interface **IAsyncResultFilter**. Isso quer dizer que esse filtro será invocado automaticamente, após a action da controller gerar um resultado válido, e antes do retorno para o client. Com isso podemos interceptar o resultado da action e formatá-lo conforme necessário.

Para mais informações sobre como os filtros funcionam, veja a [documentação oficial](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/filters?view=aspnetcore-2.1).

{{< gist 5d2598eb835964571769a6b06956c01f >}}

Verificamos se existem notificações através da propriedade **HasNotifications** do **NotificationContext**. Se existirem mensagens mudamos o Http Status Code para **Bad Request (400)** e retornamos a lista de notificações em formato Json.

>  When the presentation receives a response from the validation, **it needs to check the Notification to determine if there are errors**. If so it can pull information out of the Notification to display these errors to the user. (Martin Fowler)

Você também deve fazer a configuração no seu container de injeção de depêndencias predileto. Aqui estou usando o container nativo do ASP.Net Core.

{{< gist 07ca4d995516c0a305d39fe6e1991258 >}}

### Testes

Ao tentar fazer uma chamada para a nossa API com os dados do Customer inválidos, as mensagens de validação são exibidas todas de uma vez.

![](https://cdn-images-1.medium.com/max/2000/1*NDqC7zArtayss-ZNXe2QQg.png)


### Conclusão

O padrão Notifications nos ajuda a diminuir e até eliminar o uso de Exceptions em validações na camada de domínio de nossas aplicações, nos permitindo acumular mensagens e exibí-las todas de uma vez para o usuário, e com o uso de bibliotecas como FluentValidation podemos criar validações de forma simples.

Existem diversas formas de implementar o Notification Pattern e a que vimos aqui é uma delas. A biblioteca de validações FluentValidation também não é a única que nos ajuda nesse cenário, outra que gosto bastante é o [Flunt](https://github.com/andrebaltieri/flunt), idelizada e mantida pelo Microsoft MVP André Baltieri em conjunto com a comunidade.

E novamente, não estou dizendo para você abolir o uso de Exceptions em seus sistemas, não faça isso! Apenas apresentei uma abordagem que você pode usar e adaptar para seu cenário se achar que faz sentido, ok?! =)

Perceba também que podem sim existir situações em que o lançamento de Exceptions nas regras de negócio poderia ser mais adequado, talvez em processos que executam em background onde não existe nenhuma interface de apresentação com o usuário, etc. Como eu disse anteriormente, cada caso é um caso e vale uma avaliação.

Espero que tenham gostado desse artigo. Caso tenham dúvidas, sugestões ou críticas, deixem um comentário.

Ahh! Você pode conferir os exemplos de código em meu [Github](https://github.com/wellingtonjhn/DemoNotifications).

Abraços!

### Referências 

* [Notifications - Martin Fowler](https://martinfowler.com/eaaDev/Notification.html)
* [Replacing Throwing Exceptions with Notification in Validations - Martin Fowler](https://martinfowler.com/articles/replaceThrowWithNotification.html)
* [Exception VS Domain Notification - André Baltieri](https://medium.com/balta-io/exception-vs-domain-notification-4fe8734a039f)
* [ASP.Net Core Filters - Documentação Oficial](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/filters?view=aspnetcore-2.1)
* [Exception Class - Documentação Oficial](https://docs.microsoft.com/en-us/dotnet/api/system.exception?view=netframework-4.8)
* [Best Practices for Exceptions - Documentação Oficial](https://docs.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions)
* [Exception Throwing - Documentação Oficial](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/exception-throwing)
* [Why Exceptions should be Exceptional - Matt Warren](https://mattwarren.org/2016/12/20/Why-Exceptions-should-be-Exceptional/)