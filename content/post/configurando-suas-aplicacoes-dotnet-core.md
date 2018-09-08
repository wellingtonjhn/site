+++
comments = true
date = 2018-07-30
title = "Configurando suas aplicações .Net Core"
description = "Veja algumas dicas para configurar suas aplicações .Net Core"
tags = ["dotnetcore","aspnetcore", "configuration", "options pattern"]
categories = ["Desenvolvimento", "ASP.Net"]
nomenu = "main"
image = "https://cdn-images-1.medium.com/max/12032/1*Zv5KXdAW988QNEeyKlB83A.jpeg"
+++

Antigamente tínhamos os arquivos Web.Config e App.Config em nossas aplicações .Net Framework que na verdade eram arquivos XML onde podíamos colocar as configurações de nossas aplicações, e em conjunto à eles usávamos a classe **ConfigurationManager** para acessá-los. Muitas vezes esses arquivos se tornavam verdadeiros monstros e sua manutenção traumatizante para muitos desenvolvedores.

Com o lançamento do .Net Core e ASP.Net Core há alguns anos, ganhamos um novo mecanismo de configuração ainda mais poderoso, flexível e simples.

Esse é um artigo introdutório, e imagino que alguns de vocês irão pensar que estou escrevendo ele fora de época ou que é muito simplista, e é pra ser mesmo, mas o que me motivou é que ainda vejo muitos desenvolvedores com alguma dificuldade ao trabalhar com o modelo de configuração do .Net Core. 

O que pretendo aqui é mostrar alguns pontos que irão ajudar a entender como esse novo mecanismo de configuração funciona.

### 1. Várias Fontes de Dados

Além de arquivos JSON como o famoso **appsettings.json**, a nova API de configuração do .Net Core nos permite ter uma maior flexibilidade ao escolher a fonte de nossas configurações. Podemos ter diversas, como:

* Arquivos JSON; 
* Arquivos XML;
* Arquivos INI;
* Variáveis de ambiente;
* Coleções em memória;
* Argumentos de linha de comando;
* Fontes externas como banco de dados.

Além das fontes mencionadas acima, você pode criar seu próprio provider de configuração para ler outras fontes de dados usando as interfaces **IConfigurationSource** e **IConfigurationProvider**.

### 2. Vários Arquivos de Configuração

Assim como nos antigos Web/App.Config, nossa lista de configurações é formada por um conjunto de chave/valor, e podem ser distribuídos em arquivos organizados de forma hierárquica separados por ambiente.

Por exemplo, podemos ter um arquivo para cada ambiente específico do nosso fluxo de desenvolvimento.

![](https://cdn-images-1.medium.com/max/2000/1*IlnbFgx5eONp5fRqiA_ISw.png)

O que define qual ambiente rodar é a variável de ambiente **ASPNETCORE_ENVIRONMENT**. O valor dessa variável é que define qual ambiente estamos executando.

Em ambiente de desenvolvimento, na máquina do desenvolvedor, ela pode ser encontrada no arquivo de **launchSettings.json**, ou nas propriedades de seu projeto. Já em um sistema rodando em ambiente de produção, essa variável de ambiente deverá ser registrada no sistema operacional.

![Arquivo launchSettings.json com a variável de ambiente ASPNETCORE_ENVIRONMENT](https://cdn-images-1.medium.com/max/2360/1*0kH4BIARgZTse6EeK32pNQ.png)

### 3. Sobrescrita de Valores

Normalmente, os valores que não mudam podem ser armazenados no arquivo appsettings.json, e nos demais arquivos você pode colocar apenas os valores que mudam de acordo com o ambiente em que estiver executando pois eles são sobrescritos em memória durante a execução de sua aplicação.

![Arquivo appsettings.json](https://cdn-images-1.medium.com/max/2000/1*prx4BGYWCf0XJg4bCDSxlg.png)

![Arquivo appsettings.Development.json — com Log Level Default sobrescrito](https://cdn-images-1.medium.com/max/2000/1*nIaD-aXaccdcCHdQYd2WpA.png)

No exemplo acima, ao executar a aplicação no ambiente Development, o valor de **Logging:LogLevel:Default** será **Debug**, pois o valor original que antes era **Warning** foi sobrescrito pelo valor do arquivo appsettings.Development.json. 

### 4. Inicialização do Mecanismo de Configuração

É na classe **ConfigurationBuilder** onde toda a mágica começa a acontecer. Ela está disponível no pacote Nuget **Microsoft.Extensions.Configuration**.

Antes de mais nada, devemos criar uma nova instância de ConfigurationBuilder informando quais providers queremos usar como fonte de configuração. No exemplo abaixo um arquivo JSON é adicionado como provider através do método AddJsonFile, caso queira usar outro tipo de provider use o método correspondente, como AddXmlFile, AddIniFile, etc…

{{< gist 200bf1246af0c1d1e5e4a8cd94ef0aec >}}

Muitos desenvolvedores não conseguem acessar o arquivo de configuração **appsettings.json** em uma aplicação do tipo **Console** no .Net Core, isso se deve ao fato de que o trecho de código acima não é feito automaticamente nesse tipo de aplicação, você deve inseri-lo manualmente. 

Em aplicações ASP.Net Core 2.x não é necessário inserir esse trecho de código, pois isso já é feito automaticamente através da chamada ao método **WebHost.CreateDefaultBuilder**, que você pode ver na classe Program de sua aplicação, e então uma instância de **IConfiguration** é passada no construtor da classe **Startup**. Em versões anteriores do ASP.Net Core essa configuração era feita diretamente no construtor da classe Startup.

>  Recomendo que você registre essa instância de **IConfiguration** em modo Singleton no seu container de injeção de dependências. Isso irá facilitar a leitura de suas configurações posteriormente.

### 5. Lendo Valores de Configuração 

Como é de se imaginar, devemos usar as **chaves** descritas em nossos arquivos de configuração para ler seus valores. Para isso, podemos usar a interface **IConfiguration** mencionada anteriormente, que está disponível no pacote Nuget **Microsoft.Extensions.Configuration.Abstractions**.

Ela pode ser injetada em sua classe através do construtor e com isso podemos ler os valores através de seus métodos **GetValue\<T\>** informando a chave da qual desejamos obter o valor.

{{< gist 89f4444f64535d7b977bcb0794a23cdb >}}

Veja que a chave informada obedece a hierarquia que está no arquivo de configuração, com os nós separados por dois pontos **“:”**.

Os valores sempre são armazenados como string por default, entretanto, veja que eu informo o tipo de dado apropriado para o retorno do valor para que seja feita a conversão apropriada, nesse caso, se você estiver lendo valores numéricos, pode fazer a chamada como **configuration.GetValue\<int\>(“chave”)** por exemplo.

Além dos valores de configuração armazenados em arquivos, você pode ler as variáveis de ambiente do seu sistema operacional.

    set LOGGING__ENABLED=True 
    set LOGGING__LEVEL=Debug

Recomendo que você use **“__”** para separar as palavras no identificador de suas variáveis de ambiente, pois o carácter **“:”** não é muito adequado para esse uso.

Não se preocupe em usar caixa alta na definição das variáveis de ambiente, o acesso à elas é indiferente entre maiúsculas e minusculas pois é **case-insensitive**.

### 6. Lendo Valores de Configuração com Objetos Fortemente Tipados

Além de ler os valores como string usando o método GetValue\<T\>, você pode fazer o parse para objetos definidos como classes POCO (Plain Old CLR Object). Hoje essa é uma das formas mais comuns ao se trabalhar com valores de configuração no .Net Core.

Para isso usaremos o que é conhecido como [Options Pattern](https://docs.microsoft.com/pt-br/aspnet/core/fundamentals/configuration/options?view=aspnetcore-2.1). Ele é útil para agruparmos configurações relacionadas entre si em objetos bem definidos.

O primeiro passo é criar uma classe simples com as propriedades que compõem nosso objeto de configuração.

{{< gist 7648aae72c1ed7d5eb737aae0f323c41 >}}

Em seguida, na classe Startup, você deve ler uma seção de seu arquivo de configuração usando o método **configuration.GetSection**, e então registrá-lo no container de DI através do método **services.Configure\<TOptions\>**, sendo **TOptions** a sua classe POCO. 

Ao registrar seu objeto no container de DI, na verdade ele será definido como uma instância de **IOptions**.

{{< gist fbfb83845b738cea6a1767b4a6d7bc29 >}}

>  Em muitas literaturas você poderá ver a chamada ao método **services.AddOptions** sendo feito antes da chamada ao método **services.Configure**. Essa chamada era obrigatória para registrar os serviços necessários para usar o Options Pattern em versões anteriores do .Net Core. A partir das versões 2.x não é mais necessário fazer essa chamada explicitamente, pois o método **services.Configure** irá executá-lo internamente.

Para usar seu objeto de configuração, você deve simplesmente receber um **IOptions\<T\>** no construtor da classe onde deseja usá-lo, sendo **T** o tipo do objeto. No exemplo abaixo faço uso em uma Controller do ASP.Net.

Você deverá perceber que o objeto de configuração está armazenado na propriedade **Value** do Options que você recebeu no construtor.

{{< gist 569b5d0386215268f84005b8b21b7f63 >}}

![](https://cdn-images-1.medium.com/max/2000/1*QbMdgooUu2g632L_S2cSZg.png)

Além do IOptions, existe a interface **IOptionsSnapshot**, que te dá a possibilidade de recarregar as configurações em tempo de execução após elas serem alteradas no arquivo de configuração. Elas são recalculadas em cada requisição e são armazenadas em cache durante o período de duração da requisição.

Lembrando que os únicos providers que permitem que sejam recarregados em tempo de execução são àqueles baseados em arquivos JSON, XML e INI. Os demais não permitem que um evento de mudança seja disparado para a aplicação. Nesse caso você deverá reiniciar sua aplicação para que as mudanças tenham efeito.

### 7. Lendo Strings de Conexão

Muitos desenvolvedores ao iniciar os estudos em .Net Core usam o recurso de Options Pattern para ler uma string de conexão de acesso à base de dados, dessa forma criando classes desnecessárias para esse fim.

A interface IConfiguration já possui um método de extensão exclusivo para ler strings de conexão que se chama **GetConnectionString** onde é necessário apenas informar o nome da conexão desejada. Na verdade esse método é um atalho para o uso de **GetSection(“ConnectionStrings”)[name]**.

![Arquivo appsettings.jon](https://cdn-images-1.medium.com/max/3242/1*rr1G5fi7CwssbGtS3PFo8g.png)

{{< gist 5f96c45ce7f8ea621e87c13640c83533 >}}

### Conclusão

Como vimos, o novo mecanismo de configuração do .Net Core é muito flexível e poderoso. Aprendendo a usá-lo corretamente você terá um ótimo aliado em seus novos projetos .Net Core.

Alguns recursos vistos aqui já existiam no mecanismo anterior, porém era um pouco trabalhoso de serem usados e a manutenção não era tão trivial assim.

Nos próximos artigos irei mostrar como usar alguns recursos mais avançados de configuração.

Espero que tenham gostado, e se ficou alguma dúvida não deixem de entrar em contato.

Abraços!

### Referências

* [Documentação oficial](https://docs.microsoft.com/pt-br/aspnet/core/fundamentals/configuration/index?view=aspnetcore-2.1&tabs=basicconfiguration)
* [Options Pattern](https://docs.microsoft.com/pt-br/aspnet/core/fundamentals/configuration/options?view=aspnetcore-2.1)
