# Semana 1

## Introdução

Neste segundo artigo, eu tenho a intenção de explorar recursos iniciais de desenvolvimento na .NET.
A linguagem que estou estudando é C#. Uma linguagem moderna, orientada a objetos e fortemente
tipada. Durante o estudo e os próximos artigos, mais detalhes sobres essa linguagem serão abordados.

Este artigo aborda os seguintes tópicos:

- Visão geral da CLI (Command Line Interface) .NET
- Estrutura de comandos e gerenciamento de pacotes
- Aplicativo de Console e Entry Point
- Testes unitários com xUnit e FluentAssertions

No primeiro artigo, abordei alguns conceitos introdutórios sobre a plataforma, seus compnentes e
funcionamento. Vou considerar que já conhecemos o basicamente o que é e para que serve o .NET e que
já temos o SDK instalado. Então pularei essa parte. Acredito que a melhor maneira de aprender alguma
coisa é colocando logo a mão na massa então vamos começar:

## CLI .NET

Meu SO é um Arch Linux e estou usando Visual Estúdio Code, então darei um foco no trabalho com a CLI
porque não tenho, no momento como usar a IDE Visual Studio.

**Visão Geral**:

O que é, e para quê serve? A CLI .NET é uma ferramenta multiplataforma para desenvolvimento,
criação, execução e publicação de aplicativos .NET.

**Estrutura de Comando**

Como a maioria dos CLIs, a do .NET consiste no driver .NET (dotnet = binário de execução) suas
opções e argumentos de comando. Driver (dotnet): Tem duas responsabilidades: a) executar um
aplicativo dependente da estrutura ou executar um comando.

### Executando um aplicativo dependente da estrutura

```bash
$ dotnet /path/to/my_app.dll
```

Se estiver no diretório do aplicativo basta apenas executar $ dotnet /path/to/my_app.dll

```bash
 $ dotnet my_app.dll
```

Quando passamos um comando para o executável (dotnet) inicia-se a execução daquele comando:

```bash
 $ dotnet build
```

Primeiro, o driver determina a versão do SDK a ser usada e, se não houver nenhum arquivo global.JSON
(que é um arquivo que permite definir qual versão do SDK pode ser usada), ele usará a versão
disponível no momento. Por último, após essa determinação, o driver executa o comando.

**Comando:**

O comando executa uma ação. Por exemplo, dotnet build compila o código. dotnet publish, publica o
código.

**Argumentos:**

Argumentos que você passa na linha de comando são aqueles do comando invocado. Por exemplo, quando
você executa dotnet publish my_app.csproj, o argumento my_app.csproj indica que o projeto a ser
publicado é passado para o comando publish.

**Opções:**

As opções são aquelas do comando invocado. Por exemplo: dotnet publish --output /build_output, é a
opção que indica o caminho da saída do aplicativo a ser publicado e esse valor é passado para o
comando publish . É importante ter em mente os comandos do CLI .NET que se dividem nas seguintes
categorias:

**Gerenciar dependências:** abrange instalação, remoção, limpeza após a instalação de pacotes e
atualização dos pacotes.

**Executar programas:** abrange o fluxo de desenvolvimento, isto é, execução de testes, compilação
do código e comandos de migração para atualizar projetos.

**Criar e publicar pacotes:** abrange tarefas como criar um pacote compactado e efetuar um push do
pacote para um registro, por exemplo. Uma lista dos comandos padrão do CLI .NET e seus detalhamentos
pode ser consultada [aqui]().

CLI do .NET Iniciando um projeto .NET Antes de iniciar um projeto, é importante saber que, ao
instalar o SDK do .NET, recebemos dezenas de modelos internos para criar projetos e arquivos,
incluindo aplicativos de console, bibliotecas de classes, projetos de testes unitários, aplicativos
ASP.NET Core (incluindo projetos Angular e React), além de arquivos de configuração. Para listar
esses modelos disponíveis, executamos o comando dotnet new com a opção -l ou --list . Mais detalhes
de cada um desses modelos podem ser acessados aqui: Modelos padrão do .NET para dotnet new - .NET
CLI Hello, World É comum iniciar os estudos em .NET iniciando um aplicativo de console (terminal).
Para iniciar, você pode executar: $ dotnet new console

Esse comando cria um projeto .NET com base no modelo pré-configurado para uma aplicação desse tipo.
Você pode usar a opção --help tanto após new quanto após console para verificar as opções
disponíveis para cada. Se fizer isso, verá que existe uma opção que já direciona a saída desse
comando para um diretório específico: -o Portanto, se executar: $ dotnet new console -o MyAwesomeApp

Verá que um diretório será criado com o valor passado para opção -o , no exemplo MyAwesomeApp. Crie
um projeto console chamado HelloWorld e dê uma espiada no que existe dentro dele: $ dotnet new
console -o HelloWorld O modelo "Aplicativo do Console" foi criado com êxito.

Processando ações pós-criação... Executando 'dotnet restore' em
/home/tiago/HelloWord/HelloWord.csproj... Determining projects to restore... Restored
/home/tiago/HelloWord/HelloWord.csproj (in 263 ms). A restauração foi bem-sucedida.

$ tree HelloWorld HelloWord ├── HelloWorld.csproj ├── obj │ ├── HelloWorld.csproj.nuget.dgspec.json
│ ├── HelloWorld.csproj.nuget.g.props │ ├── HelloWorld.csproj.nuget.g.targets │ ├──
project.assets.json │ └── project.nuget.cache └── Program.cs

Program.cs Esse arquivo representa o ponto de partida do projeto. HelloWorld.csproj O arquivo com
extensão .csproj representa a base para configuração do projeto. Ele interpreta todas as
dependências do projeto, além de informações de configuração como requisitos da plataforma, controle
de versão, definições para configuração de servidor e muito mais. Aqui ele tem o nome “HelloWorld”,
mas ele terá o nome do projeto criado. Diretório obj: Este diretório é criado quando para receber as
dependências. Quando um modelo é criado, ele inclui algumas dependências padrão para sua
funcionalidade. Note a descrição na saída do comando de criação do projeto HelloWorld. Perceba que o
comando dotnet restore foi executado implicitamente. Este comando restaura as dependências e as
ferramentas de um projeto. Os registros e configurações dessas dependências ficam armazenadas no
diretório obj , que, também, pode ser chamado de repositório de dependências. O que é uma
dependência e como administrá-las? O ecossistema .NET usa muito a palavra dependência. Uma
dependência é uma biblioteca de terceiros, ou seja, um conjunto de código reutilizável que realiza
alguma funcionalidade dentro do contexto em que o aplicativo está inserido. Em outras palavras, é um
código desenvolvido por outras pessoas que funciona para um determinado contexto e, cujo qual o seu
aplicativo depende. O .NET tem muitas bibliotecas principais que cuidam de praticamente tudo, desde
gerenciamento de arquivos até HTTP e compactação de arquivos. E, esses pacotes (ou bibliotecas, ou
dependências) são gerenciados pelo NuGet, que define como os pacotes .NET são criados, hospedados e
consumidos e fornece as ferramentas para cada uma dessas funções. Bibliotecas, no geral, nos ajudam
a obter um código melhor, mais conciso, o que muitas vezes nos poupa tempo e facilita a manutenção
do programa. Contudo, é preciso levar em consideração o tamanho da biblioteca e a quantidade delas.
O número de dependências pode gerar um grande volume, o que pode ser um ponto de preocupação durante
o desenvolvimento caso você tenha recursos limitados de hardware, por exemplo. Também é preciso
verificar se a licença concedida à biblioteca abrange o uso pretendido, seja comercial, pessoal ou
acadêmico. E, por fim, mas não menos importante, se a manutenção da biblioteca está ativa. Existem
bibliotecas preteridas ou que não são atualizadas há muito tempo. Antes de usarmos a primeira
biblioteca de terceiros, vamos ver um exemplo de uso de uma das bibliotecas padrão (standard ou
bibliotecas de classes) do .NET: System.IO.File . Vamos fugir um pouco do padrão e gerar uma
mensagem “Hello World!” de uma maneira um pouco diferente. Dentro do diretório do projeto
HelloWorld, abra o arquivo Program.cs no seu editor favorito. Digite ou copie e cole o código a
seguir: using System; using System.IO;

class HelloWorld { static void Main() { string path = @"./message.txt"; if (!File.Exists(path)) { //
Crie um arquivo e grave nele. using (StreamWriter message = File.CreateText(path)) {
message.WriteLine("Hello, World!"); } } // Leia o conteúdo do arquivo criado. using (StreamReader
message = File.OpenText(path)) { string content; while ((content = message.ReadLine()) != null) {
Console.WriteLine(content); } }

    }

}

Este código cria um arquivo no diretório atual chamado message.txt e grava nele a mensagem “Hello,
Word!”, depois lê o arquivo criado enquanto existir algo nele, quando isso se torna falso, o
programa encerra. Muita coisa pode ser dita a respeito desse exemplo. Coisas que ainda estou
estudando e coisas que fogem do escopo deste artigo no momento. Por enquanto, vou me preocupar com o
próximo passo: executar esse código. Ainda no diretório do projeto, execute os comandos: $ dotnet
build

e $ dotnet run

Se tudo correr bem, a saída de primeiro comando demonstrará que a compilação foi bem sucedida. $
HelloWorld dotnet build Microsoft(R) Build Engine versão 17.0.0+c9eb9dd64 para .NET Copyright (C)
Microsoft Corporation. Todos os direitos reservados.

Determining projects to restore... All projects are up-to-date for restore.

Compilação com êxito.

    0 Aviso(s)
    0 Erro(s)

Tempo Decorrido 00:00:05.21

E a do segundo, retornará a mensagem do arquivo criado: $ HelloWorld dotnet run Hello, World!

Note que o arquivo .txt foi criado na pasta atual: $ HelloWorld tree -L 1 . ├── bin ├──
Hello2.csproj ├── message.txt ├── obj └── Program.cs

Como C# Funciona Tivemos que executar dois comandos para obter a saída esperada. O primeiro, dotnet
build compilou o código, e dotnet run que o executou. No caso das linguagens interpretadas, uma
máquina virtual traduz o código-fonte conforme ele é escrito. Em outras palavras, um software roda
junto com o código, executando linha por linha para a máquina “entender” aquilo que está definido.
Por isso você consegue executar diretamente código Javascript no REPL (Read Eval Print Loop) do
nodeJs. Não só isso: você consegue mudar valores e entradas sem precisar executar novamente o
código-fonte. Houve um tempo em que as linguagens interpretadas foram significativamente mais lentas
do que as linguagens compiladas. Mas com o desenvolvimento da compilação just-in-time, essa lacuna
está diminuindo. As linguagens compiladas, como C#, precisam de uma etapa de “construção”, ou seja,
é preciso reconstruir o programa toda vez que há alterações. Essa característica permite que o
programa execute exatamente o que o processador pode executar. Como resultado, elas tendem a ser
mais rápidas e eficientes de executar do que as linguagens interpretadas. Também dão ao
desenvolvedor mais controle sobre aspectos de hardware, como gerenciamento de memória e uso de CPU.
Contudo, a maioria das linguagens de programação podem ter implementações compiladas ou
interpretadas, isto é, a linguagem em si não é necessariamente compilada ou interpretada, mas são
normalmente referidas dessa forma por uma questão de simplificação. Python, por exemplo, pode ser
executado como um programa compilado ou como uma linguagem interpretada no modo interativo. Testes
unitários Testes unitários, além de todos os benefícios inerentes, seguram minha ansiedade de pular
etapas. Então, antes da semântica e sintaxe de uma linguagem, eu prefiro começar entendendo como
realizamos testes, pois, de quebra, acabo conhecendo os outros dois. Por isso acho importante
começar a entendê-los logo cara. O ecossistema .NET possui algumas ferramentas para desenvolver
testes de todos os tipos: unitários, de integração e de carga. Uma descrição rápida de cada um
desses tipos de testes e suas ferramentas podem ser consultadas aqui: Teste no .NET - .NET Nesse
momento, meus estudos são endereçados para testes unitários. Uma ferramenta popular para testes
unitários e focada na comunidade .NET é o xUnit. Segundo a documentação, “é a tecnologia mais
recente para testes de unidade em aplicativos .NET”. Portanto, vamos utilizá-la. Antes de iniciar,
eu pensei em criar uma calculadora simples. Uma classe MyCalculator que possui um método para cada
operação básica: Soma, Subtração, Multiplicação e Divisão. Quão difícil deve ser testar isso? Vamos
ver: Primeiro vamos criar um projeto do tipo console: $ dotnet new console -o MyCalculator

Se rodarmos o comando dotnet new -l veremos que é possível criar um modelo xUnit Test Project. Vamos
gerá-lo: $ cd MyCalculator $ MyCalculator dotnet new xunit -o MyCalculator.Test

Se dermos uma espiada no primeiro pacote, teremos: $ MyCalculator tree -L 1 . ├──
MyCalculator.csproj ├── MyCalculator.Test ├── obj └── Program.cs

E no pacote de testes: $ MyCalculator tree -L 1 MyCalculator.Test MyCalculator.Test ├──
MyCalculator.Test.csproj ├── obj └── UnitTest1.cs

O comando criou um arquivo UnitTest1.cs com a estrutura base de um teste unitário: $ MyCalculator
cat MyCalculator.Test/UnitTest1.cs using Xunit;

namespace MyCalculator.Test;

public class UnitTest1 { [Fact] public void Test1() {

    }

}

Teremos um momento para estudar mais sobre as instruções using e namespace no futuro. Por enquanto,
vemos que temos uma classe UnitTest1 com um data annotation e um bloco Test1 onde realizaremos os
testes. Um data annotation é um atributo usado para identificar os testes, descrevê-los ou passar
parâmetros. Eles existem pois o xUnit suporta dois tipos diferentes de teste de unidade: os que são
Fatos ou [Fact] e as Teorias ou [Theory]. O primeiro se refere aos testes que são sempre
verdadeiros. Eles testam condições invariáveis. Já o segundo se refere a testes que só são
verdadeiros para um conjunto de dados determinado. Antes de entender como isso se dá, existe um
último passo fundamental: a partir do diretório de testes, crie uma referência ao projeto base. Sem
isso, seu teste não saberá onde encontrar o que testar: $ dotnet add reference
../MyCalculator/Mycalculator.csproj

Criando os primeiros testes Definir os casos de teste:

A classe MyCalculator possuirá quatro métodos que realizarão uma operação matemática básica: Soma,
Subtração Multiplicação e Divisão;

Deve ser verificado se Soma, Subtração e Multiplicação efetuam as operações retornando o resultado
correto;

O método Divisão deve, além de retornar o resultado correto, lançar uma mensagem de erro caso o
divisor seja igual a 0.

Disclaimer

Imagino que muitos casos de teste podem ser criados para esse exercício. Mas eu ainda estou
aprendendo, então, vamos aos poucos.

Remova o arquivo de UnitTest1.cs e crie um arquivo MyCalculatorTest.cs com a seguinte estrutura:

using Xunit;

namespace MyCalculator.Test;

public class MyCalculatorTest { [Fact] public void MyCalculatorTestSumTwoTerms() {

    }

}

Eu me sinto mais confortável usando um padrão de escrita de testes conhecido como AAA (Arrange, Act
e Assert). Basicamente, Arrange representa o momento de preparação dos dados do teste, Act é o
momento da execução da funcionalidade que será testada e Assert é o momento de comparação do
resultado da execução com o resultado esperado. Dito isto, começarei instanciando uma classe
MyCalculator (Se, por acaso, você não está habituado com termos da programação orientada a objetos,
não se preocupe, ainda vamos estudar isso. Por enquanto, tente entender isso como uma chamada de
função… talvez ajude).

... [Fact] public void MyCalculatorTestSumTwoTerms() { var sut = new MyCalculator();

    }

Se, por acaso, você está se perguntando o motivo da variável se chamar sut, da perspectiva de Testes
Unitários, System Under Test (SUT) representa todos os atores, isto é, uma ou mais classes em um
teste que não são mocks e/ou stubs. Vou me aprofundar em mocks e stubs no contexto de C# em breve.

Como o nome do nosso teste indica, a MyCalculator tem um método Sum que recebe dois termos. Vamos
executar isso:

... [Fact] public void MyCalculatorTestSumTwoTerms() { var sut = new MyCalculator(); var result =
sut.Sum(10, 10);

    }

Por fim, vamos imaginar que 10 + 10 resulta em 20 e esperar que Sum esteja fazendo seu trabalho:

... [Fact] public void MyCalculatorTestSumTwoTerms() { var sut = new MyCalculator(); var result =
sut.Sum(10, 10); Assert.Equal(result, 20); }

Nesse ponto você já deve ter percebido que esse código não compila. Vamos executar e ver ele falhar
com o seguinte comando: $ dotnet test ... error CS0118: "MyCalculator" é um namespace, mas é usado
como um tipo [/home/tiago/Desktop/calculator/MyCalculator.Test/MyCalculator.Test.csproj]

Deixemos de lado por mais um pouco a palavra namespace . O que essa mensagem indica é que estamos
usando algo que não é o que queremos. Nem existe a implementação da classe MyCalculator nem método
Sum . Vamos resolver: Em MyCalculator/Program.cs, vamos implementar o método Sum da calculadora:
namespace MyCalculator;

public class Program { static void Main(string[] args) { } }

public class MyCalculator { public int Sum(int term1, int term2 ) => term1 + term2; }

Parece que está tudo pronto para rodar o teste. Execute novamente dotnet test . Starting test
execution, please wait... A total of 1 test files matched the specified pattern.

Passed! - Failed: 0, Passed: 2, Skipped: 0, Total: 2, Duration: 5 ms -
/home/tiago/calculator/MyCalculator.Test/bin/Debug/net6.0/MyCalculator.Test.dll (net6.0)

Para os métodos de subtração e multiplicação, os passos podem ser os mesmos. Recomendo que você
volte ao arquivo de teste e pense na implementação do teste para depois implementar os métodos no
projeto base. Vamos pular para o método de divisão. Este exige um item a mais: Como testar uma
exeção com xUnit? A divisão tem uma regra simples: Uma divisão nunca é realizada quando o valor do
dividendo é igual a 0. Queremos lançar uma exceção avisando o usuário que dividir por 0 é possível.
[Fact] public void MyCalculator_TestMethodDiv() { var message = "Não é possível realizar uma divisão
quando o divisor é igual a 0"; } }

O C# possui tipos de exceções que auxiliam no tratamento de erros. Um deles serve como uma luva para
o nosso caso que é o DivideByZeroException . Para testar se uma exceção foi lançada na chamada de um
método, o xUnit nos fornece Assert.Throws<T>(), que recebe um tipo de exceção e executa o método
verificando se ocorre o lançamento. Assert.Throws<Exception>(() => SomethingThatThrowsAnException()

Se o método SomethingThatThrowsAnException() lança a exceção que a asserção passa, se não lançar uma
exceção, a asserção falhará e, portanto, o teste falhará. Simples. Porém, também queremos testar a
mensagem da exceção. Para isso, basta adicionar mais uma asserção, dessa vez, passando a mensagem e
comparado com a mensagem lançada pela exceção: [Fact] public void MyCalculator_TestMethodDiv() { var
dividend = 10; var message = "Não é possível realizar uma divisão quando o divisor é igual a 0"; var
sut = new MyCalculator(); var expected = Assert.Throws<DivideByZeroException>(() =>
sut.Div(dividend, 0)); Assert.Equal(message, expected.Message); } }

Se rodarmos rodamos o teste, vemos ele falhar e partimos para solução: ... public int Div(int
dividend, int divisor) { try { return dividend / divisor;

        }
        catch (DivideByZeroException e)
        {
            throw new DivideByZeroException("Não é possível realizar uma divisão quando o divisor é igual a 0", e);
        }
    }

Rodamos o teste novamente e agora ele deve passar. Nesse artigo, eu comentei superficialmente, sobre
o uso de bibliotecas de terceiros, mas não chegamos a usar uma. Agora é a hora. Podemos melhorar a
legibilidade dos nossos testes com uma biblioteca chamada FluetAssertions. Essa biblioteca possui
diversos métodos que tornam os testes mais fáceis de entender, modificar, além de melhorar as
mensagens de erro. Para instalar uma dependência usamos o seguinte comando (no diretório de testes):
$ dotnet add package FluentAssertions --version 6.5.1

Note que passamos a versão do pacote. As versões e documentação podem ser acessadas aqui: Fluent
Assertions Execute dotnet restore para confirmar se o pacote foi instalado pelo NuGet. Depois,
defina a dependência no arquivo de testes através da diretiva using FluentAssertions. Vamos
refatorar o teste de Sun com os recursos oferecidos pela dependência: using Xunit; using System;
using FluentAssertions;

... ...

[Fact] public void MyCalculator_Test_Method_Sum() { var expected = 20; var sut = new MyCalculator();
var result = sut.Sum(10, 10); result.Should().Be(expected); // Assert.Equal(result, expected);

    }

A base continua a mesma, no entanto, temos uma melhor legibilidade no momento da asserção: “O
resultado deve ser o esperado”. No método Div() podemos testar o lançamento da exceção da seguinte
forma: ... public void MyCalculator_Test_Method_Div() { var dividend = 10; var message = "Não é
possível realizar uma divisão quando o divisor é igual a 0"; var sut = new MyCalculator(); var
expected = () => sut.Div(dividend, 0);
expected.Should().Throw<DivideByZeroException>().Which.Message.Should().Be(message); // var expected
= Assert.Throws<DivideByZeroException>(() => sut.Div(dividend, 0)); // Assert.Equal(message,
expected.Message); }

Mais verboso? Talvez. Mais legível? Certeza. Eu prefiro assim, mas é uma questão de gosto. Usando
Teorias Vimos que xUnit suporta dois tipos de testes unitários: os fatos e as Teorias. Usamos os
fatos com o atributo [Fact]. Agora usaremos as teorias com o atributo [Theory] . A ideia por trás de
uma [Theory], além de indicar que se trata de um teste, é tornar o mesmo mais generalista,
fornecendo argumentos para o método testado a fim de verificarmos seu comportamento em diferentes
cenários. Vamos imaginar que nossa calculadora, tem um método especialista em calcular números
ímpares e, caso receba um argumento diferente disso, lança uma exceção. Para essa teoria \*\*vamos
passar um conjunto de valores que irão testar essa feature: [Theory(DisplayName = "Verifica se o
argumento retornado é ímpar e lança uma exceção caso contrário")] [InlineData(15)] [InlineData(25)]
[InlineData(30)] public void MyCalculator_Test_Method_IsOdd(int value) { var message = $"IsOdd deve
receber um argumento cujo valor seja ímpar"; var sut = new MyCalculator(); var result = () =>
sut.IsOdd(value); result.Should().Throw<ArgumentException>().Which.Message.Should().Be(message); }

Antes de tudo, perceba o atributo DisplayName . É comum que um software tenha centenas ou milhares
de testes de unidade. Para tornar a identificação do teste mais eficiente, podemos dar um nome para
o teste, uma rápida descrição do que o teste realiza. Você pode fazer isso com [Fact] também.
[Theory] geralmente é utilizado com outro data annotation: [InlineData] . Este último também indica
que o método é um teste, além de permitir passar os valores que serão usados como argumentos para o
método. Note que passamos um conjunto de 3 valores cujo último é um valor que fere o escopo que
fechamos para o método IsOdd() . Prevendo essa possibilidade, testamos se uma exceção é lançada caso
o argumento não corresponda ao que definimos. Como das outras vezes, veremos o teste falhar e
implementamos o método logo em seguida: ... public bool IsOdd(int value) { try { return value % 2 ==
1;

        }
        catch (ArgumentException e)
        {
            throw new ArgumentException($"IsOdd deve receber um argumento cujo valor seja ímpar", e);
        }

    }

Concluíndo Esse foi um resumo da semana 1 dos meus estudos de .NET e C#. Quero ressaltar que estou
totalmente aberto para feedbacks e novas informações! Apesar de bem simples, nesse resumo vimos
alguns conceitos chave para iniciar com confiança esse mergulho. Tivemos uma visão geral do ambiente
.NET, conhecemos as estruturas da CLI do .NET, introduzimos ao longo do estudo os conceitos básicos
para inicialização e gerenciamento de pacotes, vimos a diferenças entre linguagem compilada e
interpretada, na raça criamos um exemplo guiados por testes, tema que nos aprofundamos na estrutura
básica, e tivemos um primeiro contato com a sintaxe do C# . Na próxima semana veremos mais sobre a
linguagem e seus tipos primitivos. Até!
