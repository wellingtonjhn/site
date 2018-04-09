+++
comments = true
date = 2018-01-04
title = "CI-CD com Travis e AppVeyor usando Cake e .Net Core"
description = "Aprenda a fazer deploys automatizados de seus pacotes Nuget com Cake e .Net Core"
tags = ["cicd", "cake", "dotnetcore", "build", "test", "nuget", "appveyor", "travis"]
categories = ["Desenvolvimento", "CICD"]
nomenu = "main"
image = "https://cdn-images-1.medium.com/max/3536/1*YbCRY0k7lwUDQUfRpyAVjw.png"
+++

No [artigo anterior](https://wellingtonjhn.com/posts/automatizando-tarefas-de-build-teste-e-deploy-de-pacotes-nuget-com-cake-e-.net-core/) usamos o Cake para fazer build, executar testes, gerar pacotes e deploy no repositorio oficial do Nuget através de um script de tarefas. Se você não viu corre lá e depois volte aqui.

Hoje iremos usar esse script em conjunto com duas ferramentas de CI/CD muito populares, o [Travis](https://travis-ci.org/) e [AppVeyor](https://www.appveyor.com/), pois não queremos ficar executando esse script de forma manual em nosso próprio computador.

>  Ambas as ferramentas são gratuitas para projetos open source e oferecem planos pagos para projetos privados. Você vai precisar de uma conta de usuário em cada uma delas.

Agora você deve estar se perguntando o motivo de usarmos duas ferramentas de CI que fazem praticamente as mesmas coisas. Bom, o Travis roda em ambiente Linux e OSX, já o AppVeyor roda apenas em ambiente Windows, dessa forma vamos garantir a portabilidade do código em diferentes plataformas.

### Configurando o Travis

O Travis é uma ferramenta muito popular no mundo JavaScript, Ruby, Python, entre outros, e agora também tem suporte ao .Net Core. 😁

Vamos criar um arquivo de configuração do Travis na raiz do repositório com o nome **.travis.yml**.

![Estrutura do projeto com o arquivo .travis.yml](https://cdn-images-1.medium.com/max/2000/1*BWyEs6U9000D1hLRboCI5g.png)

{{< gist 551579e5398e8a5cfb0e95a905fc5ec1 >}}

Para que o Travis possa executar o script **build.sh**, será necessário dar permissão de execução nesse arquivo em nosso repositório:

![Adicionando permissão de execução ao arquivo build.sh](https://cdn-images-1.medium.com/max/2000/1*pFkwj3Ised40qtDJ0APG8Q.png)

Agora é necessário ativar o repositório no Travis. Para isso acesse sua conta e com isso você verá uma lista de repositórios públicos do seu Github. Ative o repositório que deseja configurar.

![Ativando o repositório no Travis](https://cdn-images-1.medium.com/max/2000/1*NUN3GvKyJ8nbHJC6cfiLWA.png)

Veja que foi disparada uma trigger no Travis que inicia a execução do nosso script **build.sh**.

![Execução no Travis — failed —](https://cdn-images-1.medium.com/max/2132/1*glO2O24IOcmNZlUHY_6glA.png)

Repare que essa execução falhou!! 😓

Verificando os [logs](https://travis-ci.org/wellingtonjhn/csharp-extensions/builds/307368488) podemos perceber que ele passou pelas etapas de build, testes, criação de pacote Nuget, mas falhou ao tentar fazer o deploy do pacote no nuget.org. Essa falha ocorreu pois a chave de api do Nuget que foi configurada no arquivo **build.cake** tinha tempo de expiração para 1 dia e já não é mais válida. Lembre-se, essa chave foi utilizada no artigo anterior.

Calma!! Iremos corrigir isso mais adiante. Nesse momento podemos assumir que o Travis está configurado corretamente pois ele fez o que deveria, executou nosso script.

### Configurando o AppVeyor

O AppVeyor por outro lado é bem popular para quem usa a plataforma .Net pois ele roda em ambiente Windows e sempre suportou o .Net (full) Framework, e agora também o .Net Core.

Assim como fizemos para o Travis, também devemos criar um arquivo de configuração para o AppVeyor na raiz do repositório com o nome **appveyor.yml**.

![Estrutura do projeto com o arquivo appveyor.yml](https://cdn-images-1.medium.com/max/2000/1*ufKbw8R6koE0H2I4bLIYEw.png)

{{< gist 5b9a4fdd487b854adfd7d29dea161926 >}}

Aqui também será necessário ativar o repositório no AppVeyor. Para isso acesse sua conta, clique no menu **Projects** e em seguida no botão **New Project**, conforme abaixo:

![](https://cdn-images-1.medium.com/max/2574/1*SRcC7D7vNxYWHLUiOhZkIA.png)

Agora selecione o repositório na sua lista do Github:

![Ativando o repositório no AppVeyor](https://cdn-images-1.medium.com/max/2000/1*7ZR13YpC-vBqEqjaNMnIwQ.png)

Com isso o AppVeyor já consegue executar nosso script **build.ps1**.

![Execução no AppVeyor — failed —](https://cdn-images-1.medium.com/max/2548/1*_LyxR-CKhoGM0-p2EJCIFw.png)

Nosso build no AppVeyor também falhou pelo mesmo motivo da falha no Travis, a chave de api que estamos usando para o Nuget já não é mais válida para fazer deploy do pacote.

### Criando uma nova chave de Api do Nuget

Para começarmos a corrigir o problema, precisamos criar uma nova chave de Api no [nuget.org](https://www.nuget.org/). Para isso basta acessá-lo com sua conta, clicar no seu nome de usuário no canto superior direito, e clicar na opção **Api Keys**. Preencha as informações necessárias. No meu caso, dessa vez deixei o período máximo para expiração que é de 1 ano.

![Chave de Api para publicação no nuget.org](https://cdn-images-1.medium.com/max/2000/1*D-qOqu8dNMXgww7jR27WnA.png)

Após gerar a nova chave basta copiá-la.

### Protegendo a chave de publicação no AppVeyor

Iremos utilizar o AppVeyor para fazer deploy do nosso pacote Nuget, mas não podemos deixar a chave exposta conforme visto no artigo anterior. Iremos usar as **variáveis de ambiente** criptografadas no AppVeyor.

Para isso AppVeyor conta com a ferramenta [Encrypt Configuration Data](https://ci.appveyor.com/tools/encrypt) que nos permite criptografar qualquer informação em suas váriaveis de ambiente.

Apenas cole a chave de Api gerada no nuget.org, e clique no botão **Encrypt**.

![Criptografando variáveis de ambiente no AppVeyor](https://cdn-images-1.medium.com/max/2508/1*_zrzOY2zTwr_zYnkuothWQ.png)

Copie a nova chave criptografada e cole no arquivo de configuração do AppVeyor, dentro da seção **environment **do arquivo.

![Arquivo de configuração do AppVeyor com a chave de Api do Nuget.org criptografada](https://cdn-images-1.medium.com/max/2000/1*yt8enB8qONRxjMDSTtr4VQ.png)

Assim garantimos que nenhum engraçadinho que tenha acesso ao nosso repositório de código possa usar essa chave para publicar pacotes indevidos em nosso repositório Nuget.

Perceba que adicionamos a nova chave de publicação de pacotes Nuget apenas no arquivo de configuração do AppVeyor, já que não faz sentido as duas ferramentas de CI/CD criarem os mesmos pacotes e fazer deploy no nuget.org.
>  Neste caso escolhi o AppVeyor para criar e fazer deploy dos pacotes apenas por gosto mesmo. Você pode usar o que gostar mais.

### Ajustando o script Cake para rodar em ambiente de CI/CD

Para que os erros anteriores de execução não voltem à ocorrer, será necessário fazer alguns ajustes em nosso script **build.cake**.

{{< gist 60610d9f5d612273abc328b663791922 >}}

Nesse script adicionamos uma task de **Setup** responsável por limpar a pasta de artefatos, essa task é executada automaticamente antes da task Default, além de limpar a pasta de artefatos você pode fazer qualquer ação inicial que desejar. Também adicionamos uma condição nas tasks **Create-Nuget-Package** e **Push-Nuget-Package** para que executem apenas se o build foi disparado pela criação de uma Tag no repositório e se estiver sendo executado no AppVeyor.

Na task **Create-Nuget-Package** usamos o [GitVersion](https://github.com/GitTools/GitVersion) para fazer o [versionamento semântico](https://semver.org/) de nosso pacote, além disso a tarefa **Push-Nuget-Package** também foi alterada para obter a chave de api do Nuget através da variável de ambiente **NUGET_API_KEY** que foi criptografada anteriormente.
>  Observe que o método **ShouldRunRelease** usa o recurso de expression-body do C# 6.0. A versão 0.22 do Cake inclusive já é compatível com C# 7. Você pode conferir no [release notes](https://cakebuild.net/blog/release-notes/) do Cake.

### Deploy de um novo pacote Nuget

Conforme dito anteriormente, apesar de usarmos duas ferramentas de CI/CD apenas o AppVeyor está responsável por fazer o deploy de nossos pacotes Nuget. Para isso basta criar uma tag em nosso repositório.

    git tag v1.0.3
    git push origin --tags

Agora podemos nos preocupar apenas com o desenvolvimento de nosso componente já que as ferramentas de CI vão garantir o build, execução de testes e deploy de nosso pacote Nuget.

Com isso o processo de um novo build deve ser feito com sucesso. Confira os logs de execução do [Travis](https://travis-ci.org/wellingtonjhn/csharp-extensions/builds/313741575) e [AppVeyor](https://ci.appveyor.com/project/wellingtonjhn/csharp-extensions/build/17).

Você também pode verificar no [nuget.org](https://www.nuget.org/packages/CSharpExtensions.Core/) o pacote publicado.

![Execução no Travis com sucesso](https://cdn-images-1.medium.com/max/2040/1*c3fpSMnr2NoGHB62PU2Asw.png)

![Execução no AppVeyor com sucesso](https://cdn-images-1.medium.com/max/2090/1*97Tn6FK8bi9Qn06ZuF6gTw.png)

![Pacote nuget publicado com sucesso](https://cdn-images-1.medium.com/max/2268/1*pAWLeCg2Ewo3qdbSlo-g3w.png)

Espero que tenham gostado, e se ficou alguma dúvida ou tenham sugestões por favor entrem em contato.

Um grande abraço e até a próxima!

### Referências

* [Documentação do Travis CI](https://docs.travis-ci.com/)
* [Documentação do AppVeyor](https://www.appveyor.com/docs/)
* [Documentação do Nuget](https://docs.microsoft.com/en-us/nuget/)
