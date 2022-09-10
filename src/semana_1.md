# Semana 1

## Introdução

Neste segundo artigo, minha intenção é iniciar uma exploração dos recursos
iniciais de desenvolvimento .NET. A linguagem que estou estudando é C#. Uma
linguagem moderna, orientada a objetos e fortemente tipada. Durante o estudo (e
nos próximos artigos), mais detalhes sobres essa linguagem serão abordados.

[No artigo anterior](https://dev.to/dvths/me-tornando-um-desenvolvedor-net-0-1n15),
abordei alguns conceitos introdutórios sobre a plataforma, seus componentes e
funcionamento. Eu recomendo que você o leia para poder compreender alguns pontos
que irei abordar mais para frente. Vou considerar que já conhecemos basicamente
o que é e para que serve o .NET, e que já temos o SDK (Software Development Kit)
instalado (porque hoje tem código).

Meu ambiente de desenvolvimento é Linux, a distribuição é Arch Linux. Isso é
importante destacar por duas razões: Primeiro porque o IDE (Itegrated
Development Environment) Visual Studio, indicado para desenvolvimento .NET, não
tem suporte para Linux. Apesar de uma IDE possuir diversos recursos que
facilitam o desenvolvimento, atualmente o Visual Studio Code dá conta do recado
nesse momento de aprendizado. Particularmente, eu uso Neovim 0.7 com
[Language Server Protocol](https://docs.microsoft.com/en-us/visualstudio/extensibility/language-server-protocol?view=vs-2022)
configurado.

Segundo: eu começarei estudando a CLI (Command Line Interface) do dotnet, algo
que um usuário do IDE usará poucas vezes no dia a dia. Portanto, alguns desses
usuários pode considerar irrelevante parte desse conteúdo.

No fim do artigo, você terá percebido que alguns comandos podem ser bem longos e
repetitivos. Eu acho isso um pouco desconfortável. Por isso, escrevi um script
em bash que auxilia na criação de projetos sem que fiquemos digitando muito no
terminal. O script e suas instruções de uso estão neste
[repo](https://github.com/dvths/cs-dotnet). Nessa primeira versão, ele apenas
ajuda na criação de um aplicativo de console com testes unitários e uma
estrutura para construção de uma APIWeb seguindo os padrões SOLID/DDD (veremos
mais sobre esses termos no futuro). Conforme a necessidade irei atualizando.

Por fim, este artigo aborda os seguintes tópicos:

- Visão geral da CLI (Command Line Interface) .NET
- Aplicativos de Console e Entry Point
- Estrutura de comandos
- Gerenciamento de pacotes
- Referencia entre projetos
- Testes unitários com xUnit e FluentAssertions

---

## CLI .NET -- Visão Geral

A Interface de Linha de Comando do .NET é uma ferramenta multiplataforma para
desenvolvimento, criação, execução e publicação de aplicações .NET e vem como
parte do conjunto de recursos do SDK.

Como a maioria das CLIs, a do .NET consiste no driver -- um arquivo executável
-- opções e argumentos de comando. O driver (dotnet) tem basicamente duas
responsabilidades: a) executar um aplicativo e b) fornece uma série de comandos
para trabalhar com projetos .NET.

## Estrutura de comando

```bash
$ dotnet <COMANDO> [opções-do-comando] [argumentos]
```

Quando passamos um comando para o executável (dotnet) inicia-se a execução
daquele comando, por exemplo:

```bash
 $ dotnet build
```

Esse comando compila um projeto e suas dependências no diretório de trabalho.

```bash
$ dotnet new --list
```

Listas uma séries de opções de modelos pré preparados que você pode criar.

Cada comando define suas próprias opções e argumentos e também possuem uma opção
`--help` muito útil quando se sentir perdido.

Por tanto temos:

**Comando:**

Executa uma ação, compila o código ou publica o código (veremos mais para
frente).

**Argumentos:**

Argumentos que você passa na linha de comando são aqueles do comando invocado.
Por exemplo, quando você executa `dotnet publish my_app.csproj`, o argumento
`my_app.csproj` indica que o projeto a ser publicado é passado para o comando
publish.

**Opções:**

As opções são aquelas do comando invocado. Por exemplo: `dotnet publish --output
/build_output``, é a opção que indica o caminho da saída do aplicativo a ser
publicado e esse valor é passado para o comando publish .

É importante ter em mente, principalmente se você não está habituado a usar a
linha de comando, que os comandos do CLI .NET se dividem nas seguintes
categorias:

**Gerenciar dependências:** abrange instalação, remoção, limpeza após a
instalação de pacotes e atualização dos pacotes.

**Executar programas:** abrange o fluxo de desenvolvimento, isto é, execução de
testes, compilação do código e comandos de migração para atualizar projetos.

**Criar e publicar pacotes:** abrange tarefas como criar um pacote compactado e
efetuar um push do pacote para um registro, por exemplo.

Uma lista dos comandos do CLI .NET e seus detalhes pode ser consultada
[aqui](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet?source=recommendations#dotnet-commands),
mas lembre-se sempre da opção `--help`.

## Iniciando um Aplicativo console

Antes de iniciar um projeto, é importante saber que, ao instalar o SDK do .NET,
recebemos dezenas de modelos internos para criar projetos e arquivos como
aplicativos de console, bibliotecas de classes, projetos de testes unitários,
aplicativos ASP.NET, ect. Para listar esses modelos disponíveis, como vimos
acima, executamos o comando `dotnet new` com a opção `-l` ou `--list`.

Como estamos falando sobre aplicações de linha de comando, vamos criar um
aplicativo de console. Para iniciar, você pode executar: `$ dotnet new console`.
Contudo, você pode usar a opção `--help` tanto após `new` quanto após console
para verificar as opções disponíveis para cada. Se fizer isso após o comando
`new`, verá que existem duas opções (`-n` e `-o`) que já direcionam a saída
argumento passado para um diretório específico. Portanto, se executar:

```bash
$ dotnet new console -o HelloWord
```

Verá que um diretório será criado com o valor passado para opção `-o` , no caso,
`HelloWord`. Crie um projeto console chamado HelloWorld e dê uma espiada no que
existe dentro dele:

```shell

$ tree HelloWord
HelloWord
├── HelloWord.csproj
├── obj
│   ├── HelloWord.csproj.nuget.dgspec.json
│   ├── HelloWord.csproj.nuget.g.props
│   ├── HelloWord.csproj.nuget.g.targets
│   ├── project.assets.json
│   └── project.nuget.cache
└── Program.cs

1 directory, 7 files

```

Essa é a estrutura básica e, durante o artigo, veremos o que são esses arquivos.
Se abrirmos o arquivo `Program.cs`, teremos a instrução clássica _Hello, World_.
Não é nenhuma novidade que se executarmos esse código a mensagem será impressa
no console. Para fugir um pouco do padrão, vamos tentar deixar um pouco mais
empolgante.

A maioria dos programas processa alguma entrada para produzir uma saída;
basicamente essa é a definição de computação. Um programa obtêm dados de entrada
de maneiras diversas, mas, com frequência, a entrada vem de uma fonte externa:
um arquivo, uma conexão de rede, os dados da saída de outro programa ou de um
usuário em um teclado. Como iniciamos falando de argumentos de linha de comando,
vamos criar um programa que recebe argumentos e nos exibe o que foi passado;
algo como o comando `echo` do UNIX.

Você pode criar um novo aplicativo de console com o nome Echo. Dentro do
diretório criado, abra o arquivo `Program.cs` e remova o código que estiver lá e
cole este:

```c

namespace Echo;

/// Exibe seus argumentos da linha de comando
class Program
{
    public static void Main()
    {
        string[] args = Environment.GetCommandLineArgs();
        string sep = string.Empty;
        string s = string.Empty;

        for (int i = 1; i < args.Length; i++)
        {
            s += sep + args[i];
            sep = " ";
        }
        Console.WriteLine(s);
    }
}
```

Tudo bem se, por acaso, você não conhece a sintaxe do C#, eu também estou
aprendendo. Mas, o que esse programa faz é montar uma string concatenando
repetidamente um novo texto ao final. A string `s` _nasce_ vazia, ou seja, igual
a `""`, e cada passagem pelo loop acrescenta um texto a ela; após a primeira
iteração, um espaço é inserido (`sep`) de modo que, quando o loop acabar, haverá
um espaço entre cada argumento. Esses argumentos são obtidos e armazenados em
uma matriz de `strings` pelo método `GetCommandLineArgs()`, da classe
`Environment`.

A posição 0 da matriz é o driver (ou o executável) do programa, para não
exibi-lo, começamos o loop da posição 1, que representa, nesse caso, o primeiro
argumento passado.

Não ligue para os detalhes da implementação. A ideia aqui é entender o que vai
acontecer agora: Execute `dotnet build` e note que um novo diretório foi
adicionado a raiz do projeto: `/bin`.

```bash
bin
└── Debug
    └── net6.0
        ├── Echo
        ├── Echo.deps.json
        ├── Echo.dll
        ├── Echo.pdb
        ├── Echo.runtimeconfig.json
        └── ref
            └── Echo.dll

```

Este é o produto resultante da compilação. Aqui, temos o diretório `Debug` que
armazena um conjunto de arquivos binários. Esses binários incluem:

- o código do projeto em IL (Intermediate Language - Vimos um pouco sobre isso
  no artigo anterior) com extensão `.dll`.

- arquivos de símbolo usados para depuração com uma extensão `.pdb`
  (entenderemos melhor isso mais para frente).

- um arquivo `.deps.json`, que lista as dependências do aplicativo (ou
  biblioteca, se fosse o caso).

- um arquivo `.runtimeconfig.json`, que especifica o tempo de execução
  compartilhado e sua versão em um aplicativo.

- pode conter também outras bibliotecas das quais o projeto depende (por meio de
  referências de projeto ou referências de pacote do NuGet que ainda veremos).

- por último, temos o executável do programa. Vamos ver esse programa
  funcionando:

```bash
# ./bin/Debug

./Echo Olá, mundo!

```

A saída será `Olá, mundo!`. Mas, você não precisa navegar todas as vezes para o
diretório do executável para executar o codigo-fonte. Ao invés disso, execute
`dotnet run` e ele irá compilar e executar o programa para você. No caso do
nosso exemplo, será preciso fornecer os argumentos: `dotnet run Hello, World!`.

Nesse ponto, você deve estar se perguntando: _Ok, agora posso passar o meu
programa para os meus amigos "echoarem" o que eles passam na linha de comando?_
Não. Porque o produto não está pronto para ser transferido para outro computador
para execução. Se você leu o artigo anterior, deve recordar que os executáveis
não são multiplataforma, mas específicos do sistema operacional em que foram
desenvolvidos. Isto significa que o executável Echo criado aqui na minha máquina
não rodará em um SO ou arquitetura de CPU diferente. Para criar uma versão do
aplicativo que pode ser distribuída (implantada), você precisa publicá-lo (por
exemplo, com o comando `dotnet publish`).

Antes de estudarmos o comando `publish` e ver como podemos tornar esse programa
multiplataforma, precisamos refletir sobre a existência dos diretórios `obj` e
`bin`. Por que precisamos delas para o processo de compilação?

Bom, creio que você já deve ter ouvido a expressão "vamos compilar e ver no que
dá", ou alguém já perguntou: "o seu código está compilando?". O uso coloquial da
palavra pode gerar certa confusão para programadores mais novos (experiência
própria). Compilar não necessariamente o mesmo que criar um arquivo executável.
Em vez disso é um processo de várias estágios divididos em dois componentes:
**compilação** e **linking**. 

**Compilação**
Compilação se refere ao processamento do código-fonte e a criação de um arquivo
"objeto". Ele cria um executável? Sim. Mas o compilador apenas produz aquilo
para sua arquitetura de CPU. Inclusive, você pode ter vários arquivos compilados



------> TODO Program.cs Esse arquivo representa o ponto de partida do projeto.
HelloWorld.csproj O arquivo com extensão .csproj representa a base para
configuração do projeto. Ele interpreta todas as dependências do projeto, além
de informações de configuração como requisitos da plataforma, controle de
versão, definições para configuração de servidor e muito mais. Aqui ele tem o
nome “HelloWorld”, mas ele terá o nome do projeto criado. Diretório obj: Este
diretório é criado quando para receber as dependências. Quando um modelo é
criado, ele inclui algumas dependências padrão para sua funcionalidade. Note a
descrição na saída do comando de criação do projeto HelloWorld. Perceba que o
comando dotnet restore foi executado implicitamente. Este comando restaura as
dependências e as ferramentas de um projeto. Os registros e configurações dessas
dependências ficam armazenadas no diretório obj , que, também, pode ser chamado
de repositório de dependências. O que é uma dependência e como administrá-las? O
ecossistema .NET usa muito a palavra dependência. Uma dependência é uma
biblioteca de terceiros, ou seja, um conjunto de código reutilizável que realiza
alguma funcionalidade dentro do contexto em que o aplicativo está inserido. Em
outras palavras, é um código desenvolvido por outras pessoas que funciona para
um determinado contexto e, cujo qual o seu aplicativo depende. O .NET tem muitas
bibliotecas principais que cuidam de praticamente tudo, desde gerenciamento de
arquivos até HTTP e compactação de arquivos. E, esses pacotes (ou bibliotecas,
ou dependências) são gerenciados pelo NuGet, que define como os pacotes .NET são
criados, hospedados e consumidos e fornece as ferramentas para cada uma dessas
funções. Bibliotecas, no geral, nos ajudam a obter um código melhor, mais
conciso, o que muitas vezes nos poupa tempo e facilita a manutenção do programa.
Contudo, é preciso levar em consideração o tamanho da biblioteca e a quantidade
delas. O número de dependências pode gerar um grande volume, o que pode ser um
ponto de preocupação durante o desenvolvimento caso você tenha recursos
limitados de hardware, por exemplo. Também é preciso verificar se a licença
concedida à biblioteca abrange o uso pretendido, seja comercial, pessoal ou
acadêmico. E, por fim, mas não menos importante, se a manutenção da biblioteca
está ativa. Existem bibliotecas preteridas ou que não são atualizadas há muito
tempo. Antes de usarmos a primeira biblioteca de terceiros, vamos ver um exemplo
de uso de uma das bibliotecas padrão (standard ou bibliotecas de classes) do
.NET: System.IO.File . Vamos fugir um pouco do padrão e gerar uma mensagem
“Hello World!” de uma maneira um pouco diferente. Dentro do diretório do projeto
HelloWorld, abra o arquivo Program.cs no seu editor favorito. Digite ou copie e
cole o código a seguir: using System; using System.IO;

class HelloWorld { static void Main() { string path = @"./message.txt"; if
(!File.Exists(path)) { // Crie um arquivo e grave nele. using (StreamWriter
message = File.CreateText(path)) { message.WriteLine("Hello, World!"); } } //
Leia o conteúdo do arquivo criado. using (StreamReader message =
File.OpenText(path)) { string content; while ((content = message.ReadLine()) !=
null) { Console.WriteLine(content); } }

    }

}

Este código cria um arquivo no diretório atual chamado message.txt e grava nele
a mensagem “Hello, Word!”, depois lê o arquivo criado enquanto existir algo
nele, quando isso se torna falso, o programa encerra. Muita coisa pode ser dita
a respeito desse exemplo. Coisas que ainda estou estudando e coisas que fogem do
escopo deste artigo no momento. Por enquanto, vou me preocupar com o próximo
passo: executar esse código. Ainda no diretório do projeto, execute os comandos:
$ dotnet build

e $ dotnet run

Se tudo correr bem, a saída de primeiro comando demonstrará que a compilação foi
bem sucedida. $ HelloWorld dotnet build Microsoft(R) Build Engine versão
17.0.0+c9eb9dd64 para .NET Copyright (C) Microsoft Corporation. Todos os
direitos reservados.

Determining projects to restore... All projects are up-to-date for restore.

Compilação com êxito.

    0 Aviso(s)
    0 Erro(s)

Tempo Decorrido 00:00:05.21

E a do segundo, retornará a mensagem do arquivo criado: $ HelloWorld dotnet run
Hello, World!

Note que o arquivo .txt foi criado na pasta atual: $ HelloWorld tree -L 1 . ├──
bin ├── Hello2.csproj ├── message.txt ├── obj └── Program.cs

Como C# Funciona Tivemos que executar dois comandos para obter a saída esperada.
O primeiro, dotnet build compilou o código, e dotnet run que o executou. No caso
das linguagens interpretadas, uma máquina virtual traduz o código-fonte conforme
ele é escrito. Em outras palavras, um software roda junto com o código,
executando linha por linha para a máquina “entender” aquilo que está definido.
Por isso você consegue executar diretamente código Javascript no REPL (Read Eval
Print Loop) do nodeJs. Não só isso: você consegue mudar valores e entradas sem
precisar executar novamente o código-fonte. Houve um tempo em que as linguagens
interpretadas foram significativamente mais lentas do que as linguagens
compiladas. Mas com o desenvolvimento da compilação just-in-time, essa lacuna
está diminuindo. As linguagens compiladas, como C#, precisam de uma etapa de
“construção”, ou seja, é preciso reconstruir o programa toda vez que há
alterações. Essa característica permite que o programa execute exatamente o que
o processador pode executar. Como resultado, elas tendem a ser mais rápidas e
eficientes de executar do que as linguagens interpretadas. Também dão ao
desenvolvedor mais controle sobre aspectos de hardware, como gerenciamento de
memória e uso de CPU. Contudo, a maioria das linguagens de programação podem ter
implementações compiladas ou interpretadas, isto é, a linguagem em si não é
necessariamente compilada ou interpretada, mas são normalmente referidas dessa
forma por uma questão de simplificação. Python, por exemplo, pode ser executado
como um programa compilado ou como uma linguagem interpretada no modo
interativo. Testes unitários Testes unitários, além de todos os benefícios
inerentes, seguram minha ansiedade de pular etapas. Então, antes da semântica e
sintaxe de uma linguagem, eu prefiro começar entendendo como realizamos testes,
pois, de quebra, acabo conhecendo os outros dois. Por isso acho importante
começar a entendê-los logo cara. O ecossistema .NET possui algumas ferramentas
para desenvolver testes de todos os tipos: unitários, de integração e de carga.
Uma descrição rápida de cada um desses tipos de testes e suas ferramentas podem
ser consultadas aqui: Teste no .NET - .NET Nesse momento, meus estudos são
endereçados para testes unitários. Uma ferramenta popular para testes unitários
e focada na comunidade .NET é o xUnit. Segundo a documentação, “é a tecnologia
mais recente para testes de unidade em aplicativos .NET”. Portanto, vamos
utilizá-la. Antes de iniciar, eu pensei em criar uma calculadora simples. Uma
classe MyCalculator que possui um método para cada operação básica: Soma,
Subtração, Multiplicação e Divisão. Quão difícil deve ser testar isso? Vamos
ver: Primeiro vamos criar um projeto do tipo console: $ dotnet new console -o
MyCalculator

Se rodarmos o comando dotnet new -l veremos que é possível criar um modelo xUnit
Test Project. Vamos gerá-lo: $ cd MyCalculator $ MyCalculator dotnet new xunit
-o MyCalculator.Test

Se dermos uma espiada no primeiro pacote, teremos: $ MyCalculator tree -L 1 .
├── MyCalculator.csproj ├── MyCalculator.Test ├── obj └── Program.cs

E no pacote de testes: $ MyCalculator tree -L 1 MyCalculator.Test
MyCalculator.Test ├── MyCalculator.Test.csproj ├── obj └── UnitTest1.cs

O comando criou um arquivo UnitTest1.cs com a estrutura base de um teste
unitário: $ MyCalculator cat MyCalculator.Test/UnitTest1.cs using Xunit;

namespace MyCalculator.Test;

public class UnitTest1 { [Fact] public void Test1() {

    }

}

Teremos um momento para estudar mais sobre as instruções using e namespace no
futuro. Por enquanto, vemos que temos uma classe UnitTest1 com um data
annotation e um bloco Test1 onde realizaremos os testes. Um data annotation é um
atributo usado para identificar os testes, descrevê-los ou passar parâmetros.
Eles existem pois o xUnit suporta dois tipos diferentes de teste de unidade: os
que são Fatos ou [Fact] e as Teorias ou [Theory]. O primeiro se refere aos
testes que são sempre verdadeiros. Eles testam condições invariáveis. Já o
segundo se refere a testes que só são verdadeiros para um conjunto de dados
determinado. Antes de entender como isso se dá, existe um último passo
fundamental: a partir do diretório de testes, crie uma referência ao projeto
base. Sem isso, seu teste não saberá onde encontrar o que testar: $ dotnet add
reference ../MyCalculator/Mycalculator.csproj

Criando os primeiros testes Definir os casos de teste:

A classe MyCalculator possuirá quatro métodos que realizarão uma operação
matemática básica: Soma, Subtração Multiplicação e Divisão;

Deve ser verificado se Soma, Subtração e Multiplicação efetuam as operações
retornando o resultado correto;

O método Divisão deve, além de retornar o resultado correto, lançar uma mensagem
de erro caso o divisor seja igual a 0.

Disclaimer

Imagino que muitos casos de teste podem ser criados para esse exercício. Mas eu
ainda estou aprendendo, então, vamos aos poucos.

Remova o arquivo de UnitTest1.cs e crie um arquivo MyCalculatorTest.cs com a
seguinte estrutura:

using Xunit;

namespace MyCalculator.Test;

public class MyCalculatorTest { [Fact] public void MyCalculatorTestSumTwoTerms()
{

    }

}

Eu me sinto mais confortável usando um padrão de escrita de testes conhecido
como AAA (Arrange, Act e Assert). Basicamente, Arrange representa o momento de
preparação dos dados do teste, Act é o momento da execução da funcionalidade que
será testada e Assert é o momento de comparação do resultado da execução com o
resultado esperado. Dito isto, começarei instanciando uma classe MyCalculator
(Se, por acaso, você não está habituado com termos da programação orientada a
objetos, não se preocupe, ainda vamos estudar isso. Por enquanto, tente entender
isso como uma chamada de função… talvez ajude).

... [Fact] public void MyCalculatorTestSumTwoTerms() { var sut = new
MyCalculator();

    }

Se, por acaso, você está se perguntando o motivo da variável se chamar sut, da
perspectiva de Testes Unitários, System Under Test (SUT) representa todos os
atores, isto é, uma ou mais classes em um teste que não são mocks e/ou stubs.
Vou me aprofundar em mocks e stubs no contexto de C# em breve.

Como o nome do nosso teste indica, a MyCalculator tem um método Sum que recebe
dois termos. Vamos executar isso:

... [Fact] public void MyCalculatorTestSumTwoTerms() { var sut = new
MyCalculator(); var result = sut.Sum(10, 10);

    }

Por fim, vamos imaginar que 10 + 10 resulta em 20 e esperar que Sum esteja
fazendo seu trabalho:

... [Fact] public void MyCalculatorTestSumTwoTerms() { var sut = new
MyCalculator(); var result = sut.Sum(10, 10); Assert.Equal(result, 20); }

Nesse ponto você já deve ter percebido que esse código não compila. Vamos
executar e ver ele falhar com o seguinte comando: $ dotnet test ... error
CS0118: "MyCalculator" é um namespace, mas é usado como um tipo
[/home/tiago/Desktop/calculator/MyCalculator.Test/MyCalculator.Test.csproj]

Deixemos de lado por mais um pouco a palavra namespace . O que essa mensagem
indica é que estamos usando algo que não é o que queremos. Nem existe a
implementação da classe MyCalculator nem método Sum . Vamos resolver: Em
MyCalculator/Program.cs, vamos implementar o método Sum da calculadora:
namespace MyCalculator;

public class Program { static void Main(string[] args) { } }

public class MyCalculator { public int Sum(int term1, int term2 ) => term1 +
term2; }

Parece que está tudo pronto para rodar o teste. Execute novamente dotnet test .
Starting test execution, please wait... A total of 1 test files matched the
specified pattern.

Passed! - Failed: 0, Passed: 2, Skipped: 0, Total: 2, Duration: 5 ms -
/home/tiago/calculator/MyCalculator.Test/bin/Debug/net6.0/MyCalculator.Test.dll
(net6.0)

Para os métodos de subtração e multiplicação, os passos podem ser os mesmos.
Recomendo que você volte ao arquivo de teste e pense na implementação do teste
para depois implementar os métodos no projeto base. Vamos pular para o método de
divisão. Este exige um item a mais: Como testar uma exeção com xUnit? A divisão
tem uma regra simples: Uma divisão nunca é realizada quando o valor do dividendo
é igual a 0. Queremos lançar uma exceção avisando o usuário que dividir por 0 é
possível. [Fact] public void MyCalculator_TestMethodDiv() { var message = "Não é
possível realizar uma divisão quando o divisor é igual a 0"; } }

O C# possui tipos de exceções que auxiliam no tratamento de erros. Um deles
serve como uma luva para o nosso caso que é o DivideByZeroException . Para
testar se uma exceção foi lançada na chamada de um método, o xUnit nos fornece
Assert.Throws<T>(), que recebe um tipo de exceção e executa o método verificando
se ocorre o lançamento. Assert.Throws<Exception>(() =>
SomethingThatThrowsAnException()

Se o método SomethingThatThrowsAnException() lança a exceção que a asserção
passa, se não lançar uma exceção, a asserção falhará e, portanto, o teste
falhará. Simples. Porém, também queremos testar a mensagem da exceção. Para
isso, basta adicionar mais uma asserção, dessa vez, passando a mensagem e
comparado com a mensagem lançada pela exceção: [Fact] public void
MyCalculator_TestMethodDiv() { var dividend = 10; var message = "Não é possível
realizar uma divisão quando o divisor é igual a 0"; var sut = new
MyCalculator(); var expected = Assert.Throws<DivideByZeroException>(() =>
sut.Div(dividend, 0)); Assert.Equal(message, expected.Message); } }

Se rodarmos rodamos o teste, vemos ele falhar e partimos para solução: ...
public int Div(int dividend, int divisor) { try { return dividend / divisor;

        }
        catch (DivideByZeroException e)
        {
            throw new DivideByZeroException("Não é possível realizar uma divisão quando o divisor é igual a 0", e);
        }
    }

Rodamos o teste novamente e agora ele deve passar. Nesse artigo, eu comentei
superficialmente, sobre o uso de bibliotecas de terceiros, mas não chegamos a
usar uma. Agora é a hora. Podemos melhorar a legibilidade dos nossos testes com
uma biblioteca chamada FluetAssertions. Essa biblioteca possui diversos métodos
que tornam os testes mais fáceis de entender, modificar, além de melhorar as
mensagens de erro. Para instalar uma dependência usamos o seguinte comando (no
diretório de testes): $ dotnet add package FluentAssertions --version 6.5.1

Note que passamos a versão do pacote. As versões e documentação podem ser
acessadas aqui: Fluent Assertions Execute dotnet restore para confirmar se o
pacote foi instalado pelo NuGet. Depois, defina a dependência no arquivo de
testes através da diretiva using FluentAssertions. Vamos refatorar o teste de
Sun com os recursos oferecidos pela dependência: using Xunit; using System;
using FluentAssertions;

... ...

[Fact] public void MyCalculator_Test_Method_Sum() { var expected = 20; var sut =
new MyCalculator(); var result = sut.Sum(10, 10); result.Should().Be(expected);
// Assert.Equal(result, expected);

    }

A base continua a mesma, no entanto, temos uma melhor legibilidade no momento da
asserção: “O resultado deve ser o esperado”. No método Div() podemos testar o
lançamento da exceção da seguinte forma: ... public void
MyCalculator_Test_Method_Div() { var dividend = 10; var message = "Não é
possível realizar uma divisão quando o divisor é igual a 0"; var sut = new
MyCalculator(); var expected = () => sut.Div(dividend, 0);
expected.Should().Throw<DivideByZeroException>().Which.Message.Should().Be(message);
// var expected = Assert.Throws<DivideByZeroException>(() => sut.Div(dividend,
0)); // Assert.Equal(message, expected.Message); }

Mais verboso? Talvez. Mais legível? Certeza. Eu prefiro assim, mas é uma questão
de gosto. Usando Teorias Vimos que xUnit suporta dois tipos de testes unitários:
os fatos e as Teorias. Usamos os fatos com o atributo [Fact]. Agora usaremos as
teorias com o atributo [Theory] . A ideia por trás de uma [Theory], além de
indicar que se trata de um teste, é tornar o mesmo mais generalista, fornecendo
argumentos para o método testado a fim de verificarmos seu comportamento em
diferentes cenários. Vamos imaginar que nossa calculadora, tem um método
especialista em calcular números ímpares e, caso receba um argumento diferente
disso, lança uma exceção. Para essa teoria \*\*vamos passar um conjunto de
valores que irão testar essa feature: [Theory(DisplayName = "Verifica se o
argumento retornado é ímpar e lança uma exceção caso contrário")]
[InlineData(15)] [InlineData(25)] [InlineData(30)] public void
MyCalculator_Test_Method_IsOdd(int value) { var message = $"IsOdd deve receber
um argumento cujo valor seja ímpar"; var sut = new MyCalculator(); var result =
() => sut.IsOdd(value);
result.Should().Throw<ArgumentException>().Which.Message.Should().Be(message); }

Antes de tudo, perceba o atributo DisplayName . É comum que um software tenha
centenas ou milhares de testes de unidade. Para tornar a identificação do teste
mais eficiente, podemos dar um nome para o teste, uma rápida descrição do que o
teste realiza. Você pode fazer isso com [Fact] também. [Theory] geralmente é
utilizado com outro data annotation: [InlineData] . Este último também indica
que o método é um teste, além de permitir passar os valores que serão usados
como argumentos para o método. Note que passamos um conjunto de 3 valores cujo
último é um valor que fere o escopo que fechamos para o método IsOdd() .
Prevendo essa possibilidade, testamos se uma exceção é lançada caso o argumento
não corresponda ao que definimos. Como das outras vezes, veremos o teste falhar
e implementamos o método logo em seguida: ... public bool IsOdd(int value) { try
{ return value % 2 == 1;

        }
        catch (ArgumentException e)
        {
            throw new ArgumentException($"IsOdd deve receber um argumento cujo valor seja ímpar", e);
        }

    }

Concluíndo Esse foi um resumo da semana 1 dos meus estudos de .NET e C#. Quero
ressaltar que estou totalmente aberto para feedbacks e novas informações! Apesar
de bem simples, nesse resumo vimos alguns conceitos chave para iniciar com
confiança esse mergulho. Tivemos uma visão geral do ambiente .NET, conhecemos as
estruturas da CLI do .NET, introduzimos ao longo do estudo os conceitos básicos
para inicialização e gerenciamento de pacotes, vimos a diferenças entre
linguagem compilada e interpretada, na raça criamos um exemplo guiados por
testes, tema que nos aprofundamos na estrutura básica, e tivemos um primeiro
contato com a sintaxe do C# . Na próxima semana veremos mais sobre a linguagem e
seus tipos primitivos. Até!
