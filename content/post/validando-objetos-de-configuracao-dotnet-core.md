---
comments : true
date : 2018-09-08
title : "Validando Objetos de Configuração no .Net Core"
description : "Veja como validar objetos de configuração no .Net Core"
tags : ["dotnetcore","aspnetcore", "configuration", "options pattern"]
categories : ["Desenvolvimento", "ASP.Net"]
nomenu : "main"
image : "https://cdn-images-1.medium.com/max/8512/1*e7O7A5rM79ng6b8Dc3EbEg.jpeg"
---

No [artigo anterior](https://www.wellingtonjhn.com/posts/configurando-suas-aplica%C3%A7%C3%B5es-.net-core/) eu falei sobre alguns conceitos básicos do modelo de configuração do .Net Core, e um dos itens abordados é a possibilidade de podermos usar objetos POCO para representar grupos de configuração relacionadas entre si usando [Options Pattern](https://docs.microsoft.com/pt-br/aspnet/core/fundamentals/configuration/options?view=aspnetcore-2.1).

Nesse artigo irei mostrar como podemos validar esses objetos de configuração de forma a garantir que eles foram instanciados corretamente, uma vez que estamos totalmente suscetíveis a erros no momento em que definimos as configurações de nossa aplicação, e podemos deixar algum valor importante em branco por exemplo.

> Os projetos de exemplo estão no meu Github e os links estão no fim do artigo.

### Definindo as validações

A primeira coisa a fazer é definir as validações em nossa classe de configuração. Você pode usar algo como o [FluentValidation ](https://github.com/JeremySkinner/FluentValidation)ou qualquer outra biblioteca de validação. No meu caso vou usar o próprio mecanismo de Data Annotations nativo do .Net.

{{< gist 51d566ae7de2790e7a37787298fa498a >}}

Veja que eu criei uma classe abstrata chamada **ConfigurationSettings** para que as classes de configuração possam herdá-la. Nela, existe apenas a implementação de um método chamado **Validate** que irá verificar nosso objeto usando os Data Annotations, e uma propriedade chamada **ValidationResult** que irá acumular as mensagens de erro.

É importante notar que esses objetos de configuração são baseados na interface **IOptions\<T\>**, que tem como princípio o uso de Lazy Load ao efetuar a leitura de seus valores, sendo assim, o objeto é instanciado somente durante a primeira chamada de sua propriedade **Value**, e não quando o objeto é configurado na classe Startup usando o método **Configure\<TOptions\>**.

### Validando seu objeto de configuração durante o fluxo normal da aplicação

Agora precisamos de um *interceptor* que será responsável por invocar o método **Validate** mencionado acima, pois não quero ficar fazendo essa chamada em cada classe que precisar ler as configurações da aplicação.

Existem algumas formas de se fazer isso, uma delas é usando a interface **IPostConfigureOptions\<TOptions\>** e criar uma classe que será invocada automaticamente ao fazer uso do objeto de configuração.
>  Lendo a documentação da interface IPostConfigureOptions, fica claro que seu objetivo principal é nos dar a possibilidade de fazer algo depois de os valores serem carregados do arquivo appsettings.json, dando a oportunidade de alterá-los em tempo de execução, definir valores default, invalidação de cache, etc. Mas nada impede de usarmos para validar nosso objeto. =)

{{< gist 1f240e4f718364da83862a0fd4c79ff1 >}}

Veja que a classe **MySettingsValidator** implementa a interface mencionada acima com o método PostConfigure.

Para que ela funcione, você deve fazer sua configuração na classe **Startup** usando o método **services.ConfigureOptions\<TConfigureOptions\>**.

Quando o objeto de configuração for utilizado pela primeira vez, irá invocar o método PostConfigure que irá verificar se o objeto é válido, lançando uma exception caso algum valor não obedeça aos critérios de validação que foram definidos. Caso ache necessário, você também pode gerar um log de erro na classe **MySettingsValidator** para registrar o motivo da falha.

![](https://cdn-images-1.medium.com/max/2130/1*b-C4CgRiVPV2YQhaSSHXjA.png)

![](https://cdn-images-1.medium.com/max/3840/1*kfOHNyAijsodorMap9ECjQ.png)

Caso você não queira implementar a interface IPostConfigureOptions, poderá usar o método de extensão **PostConfigure** da interface IServicesCollection diretamente em seu arquivo Startup.

{{< gist 58116c42791ed61db9bffdca9a867367 >}}

Um ponto muito importante é que esse tipo de validação é feita durante o fluxo de execução normal da aplicação, sendo assim, você ainda deve tratar a exception que é lançada. Nesse momento você deve estar pensando que não vale a pena fazer a validação se uma exception é disparada de qualquer forma. Bom, em um cenário sem a validação, ao ler a propriedade “ApplicationName”, será retornado seu valor atual, ou seja, vazio, fazendo deste modo com que a aplicação possa ter um comportamento estranho caso não esteja preparada para isso, e acredite, aplicações não preparadas para valores vazios ou nulos são a coisa mais comum que você irá encontrar por aí.

### Validando seu objeto de configuração durante a inicialização da aplicação

Para evitar o problema de lançamento de exceptions descrito acima, você pode validar seus objetos de configuração durante a inicialização do seu sistema. Existem diversas formas de se fazer isso, uma delas é criando um filtro de inicialização, mais especificamente usando a interface **IStartupFilter**.

Essa solução não é muito diferente da anterior. Com a classe de configuração e suas validações criadas, vamos criar o filtro e registrar no pipeline do sistema.

{{< gist 861c887e87e00a3b5f6cd98c0cb2ed8b >}}

Veja a classe **ConfigurationSettingValidationStartupFilter**, que implementa a interface IStartupFilter. Ela recebe em seu construtor uma coleção de **ConfigurationSettings**, que é o tipo base para todos os objetos de configuração que a aplicação possa ter. Em seu método **Configure** é onde executamos a validação de nosso objeto.

Além dela, criamos um método de extensão para auxiliar o registro dos objetos de configuração, através do método **RegisterSettings\<TOptions\>** que está na classe estática **ConfigureOptionsExtensions**.

Agora basta configurar o filtro de validação e o objeto de configuração na classe Startup.

{{< gist cd475a9c36d09fd7d95385f26a79bd23 >}}

Ao executar a aplicação, caso os critérios de validação não sejam satisfeitos um log é gerado e uma exception é disparada, exatamente como no modelo anterior. A diferença é que essa classe é invocada durante a inicialização da aplicação, dessa forma impedindo que seu sistema inicie com as configurações erradas.

![](https://cdn-images-1.medium.com/max/2276/1*xmM8Mcu1Aq2cu-C2rlIf6w.png)

Um ponto importante dessa solução, é que a validação só ocorre uma única vez durante a inicialização do sistema. Se você estiver usando **IOptionsSnapshot\<TOptions\>** para recarregar suas configurações, essa abordagem não funcionará para você, e em algum momento seus objetos podem ficar inválidos.

### Conclusão

Normalmente erros de configuração são percebidos somente quando nosso sistema já está em execução no ambiente de produção, e isso é muito grave, já que irá impactar diretamente os usuários do sistema causando comportamentos estranhos na aplicação.

Parece bobo imaginar que iremos configurar nossas aplicações de forma errada, esquecer algum valor obrigatório ou algo assim, pois geralmente somos nós mesmos que a configuramos, mas lembre-se que as aplicações podem crescer de tamanho e um dia se faça necessário usar um servidor de configuração externo, seja um banco de dados ou um Key Vault qualquer. Outra equipe pode ter a responsabilidade de alterar as configurações dos sistemas, ou até nós mesmos podemos esquecer algum valor, isso já aconteceu comigo e certamente pode acontecer com você, então esteja preparado.

Se ficou qualquer dúvida, não deixe de entrar em contato.

Abraços!

### Referências

* [Documentação IPostConfigureOptions](https://docs.microsoft.com/en-gb/aspnet/core/fundamentals/configuration/options?view=aspnetcore-2.1#ipostconfigureoptions)
* [Documentação Data Annotations](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations?view=netframework-4.7.2)
* [Documentação Startup Filter](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/startup?view=aspnetcore-2.1)
* [Projeto de exemplo usando IPostConfigureOptions](https://github.com/wellingtonjhn/DemoSettingsValidationPostConfigure)
* [Projeto de exemplo usando IStartupFilter](https://github.com/wellingtonjhn/DemoSettingsValidationStartup)