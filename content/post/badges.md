+++
comments = true
date = 2018-01-05
title = "Badges"
description = "Saiba como adicionar badges em seus arquivos de Readme"
tags = ["badges"]
categories = ["Desenvolvimento"]
nomenu = "main"
image = ""
+++

No [artigo anterior](https://wellingtonjhn.com/posts/ci-cd-com-travis-e-appveyor-usando-cake-e-.net-core/) vimos como configurar o Travis e AppVeyor para fazer todo o processo de build e deploy de um componente Nuget escrito em .Net Core. Agora veremos como podemos configurar **“badges”** em nosso repositório para exibir o status de build do nosso projeto.

![badges](https://cdn-images-1.medium.com/max/2000/1*f9o_MXmOsGNcpPVHWF6-hA.png)

Para isso vamos usar o [shields.io](http://shields.io/) pois ele permite que façamos algumas customizações escolhendo por exemplo um label, formato, tamanho, etc.

Acessando o shields.io, veremos uma lista enorme de badges que podemos usar, escolha a que melhor se adequa ao que você quer exibir e clique para editar. 
Eu escolhi as badges para exibir o status de build do Travis e AppVeyor, e também uma badge para exibir a última versão do pacote Nuget.

Para configurar a badge de status de build é necessário informar o profile e nome do projeto configurado no CI, e para a badge de Nuget basta informar o nome do pacote.

![Editando a badge no shields.io](https://cdn-images-1.medium.com/max/2340/1*83-dan1lqsK3H3P3rkSxMQ.png)

Copie o código Markdown que foi gerado e cole no arquivo **README.md**, conforme exemplo abaixo:

<pre>
  <code>
  [![Travis](https://img.shields.io/travis/wellingtonjhn/csharp-extensions.svg?label=travis)](https://travis-ci.org/wellingtonjhn/csharp-extensions)
  [![AppVeyor](https://img.shields.io/appveyor/ci/wellingtonjhn/csharp-extensions.svg?label=appveyor)](https://ci.appveyor.com/project/wellingtonjhn/csharp-extensions)
  [![NuGet](https://img.shields.io/nuget/v/CSharpExtensions.Core.svg)](https://www.nuget.org/packages/CSharpExtensions.Core)
  </code>
</pre>

Após isso, as badges serão exibidas conforme a seguir.

![Badges configuradas no arquivo README.md](https://cdn-images-1.medium.com/max/2000/1*rZcPpUfpvjF6EpssrTQJnw.png)

Era isso, uma dica bem simples mas que deixa a “página inicial” de nossos repositórios com um aspecto bem legal.

Um grande abraço e até a próxima!
