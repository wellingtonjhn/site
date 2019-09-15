---
title: CQS - Command Query Separation
date: '2019-09-15T16:05:31+01:00'
categories:
  - Desenvolvimento
tags:
  - cqs
  - design patterns
autoThumbnailImage: false
thumbnailImagePosition: left
coverImage: ''
---
O [CQS (Command Query Separation)](https://martinfowler.com/bliki/CommandQuerySeparation.html) é um pattern introduzido por **Bertrand Meyer** no livro **[Object Oriented Software Construction](https://www.amazon.com/gp/product/0136291554)**, com a primeira edição publicada em 1988, e a segunda, revisada e expandida em 1997.

A idéia central é que os métodos de uma aplicação podem ser **commands** (comandos) ou **queries** (consultas), mas nunca ambos.

Temos então:

* Commands: métodos que alteram o estado (mudança de valores) do objeto que o define sem retornar nenhum valor (deve ser “void”), e causam efeitos colaterais no sistema.

* Queries: métodos que retornam valores, mas não alteram o estado do objeto que o define.

![](https://cdn-images-1.medium.com/max/2000/1*ISZtRPbcJbGb1A4R0l7Oag.png)

>  No exemplo acima, desconsidere questões como injeção de dependências e abstrações, é apenas um exemplo didático, ok?! ;)

Repare que as queries podem ser chamadas várias vezes em sequência produzindo sempre o mesmo resultado, e sem causar qualquer mudança de estado no sistema (sem efeitos colaterais). São consideradas **funções puras**. 

>  O conceito de [Pure Functions](https://en.wikipedia.org/wiki/Pure_function) vem da programação funcional e são análogas à funções matemáticas, onde o valor de retorno será determinado pelos parâmetros de entrada, ou seja, se você passar sempre os mesmos valores de entrada terá sempre o mesmo resultado.

O uso adequado do CQS vai totalmente de encontro com o [Princípio de Responsabilidade Única (SRP)](https://robsoncastilho.com.br/2013/02/06/principios-solid-principio-da-responsabilidade-unica-srp/) do SOLID. Seus métodos tendem a ficar mais limpos e coesos.

Você verá que nem sempre é possível aplicar o CQS. Um bom exemplo disso está na clase **[Stack](https://docs.microsoft.com/en-us/dotnet/api/system.collections.stack?view=netframework-4.8)** do .Net, o método **[Pop](https://docs.microsoft.com/en-us/dotnet/api/system.collections.stack.pop?view=netframework-4.8#System_Collections_Stack_Pop)** é ao mesmo tempo um Command e uma Query, pois ele remove o objeto da pilha (altera estado) e ao mesmo tempo retorna o objeto para quem chamou o método (consulta). Ele claramente viola o CQS, mas também não faz muito sentido separar essas responsabilidades em dois métodos diferentes devido sua natureza.

![](https://cdn-images-1.medium.com/max/2000/1*CdquqgxmnYn5dDCrdZdfvw.png)

### Conclusão

Em minha opinião, é uma boa idéia considerar o CQS como uma prática de programação que devemos seguir sempre, e abrir uma exceção à ele somente e quando for necessário, conforme vimos acima.

Apenas para salientar, devemos levar em consideração que CQS não é CQRS (Command Query Responsibility Segregation), existe uma certa confusão entre os dois patterns hoje em dia, pretendo escrever mais sobre CQRS futuramente.

Espero que tenham gostado da dica de hoje, e se possível deixem suas críticas ou sugestões.

Abraços!

### Referências

* [https://martinfowler.com/bliki/CommandQuerySeparation.html](https://martinfowler.com/bliki/CommandQuerySeparation.html)

* [https://robsoncastilho.com.br/2013/02/06/principios-solid-principio-da-responsabilidade-unica-srp/](https://robsoncastilho.com.br/2013/02/06/principios-solid-principio-da-responsabilidade-unica-srp/)

* [https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/refactoring-into-pure-functions](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/refactoring-into-pure-functions)
