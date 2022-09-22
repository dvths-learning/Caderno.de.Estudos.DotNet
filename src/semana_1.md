# Semana 1

## Introdução

Neste segundo artigo, minha intenção é explorar os recursos iniciais de
desenvolvimento .NET. A linguagem que estou estudando é C#. Uma linguagem
moderna, orientada a objetos e fortemente tipada. Durante o estudo (e nos
próximos artigos), mais detalhes sobres essa linguagem serão abordados.

[No artigo anterior](https://dev.to/dvths/me-tornando-um-desenvolvedor-net-0-1n15),
abordei alguns conceitos introdutórios sobre a plataforma, seus componentes e
recursos. Eu recomendo que você o leia para poder compreender melhor, se for o
caso, alguns pontos que irei abordar mais para frente. Vou considerar que já
conhecemos basicamente o que é e para que serve o .NET, e que já temos o SDK
(Software Development Kit) instalado (porque hoje tem código).

Eu estou estudando em um Arch Linux. Isso é importante destacar por duas razões:
Primeiro porque o IDE (Itegrated Development Environment) Visual Studio,
indicado para desenvolvimento .NET, não tem suporte para Linux. Apesar de uma
IDE possuir diversos recursos que facilitam o desenvolvimento, o Visual Studio
Code dá conta do recado neste momento de aprendizado. Particularmente, eu uso
Neovim 0.7 com
[Language Server Protocol](https://docs.microsoft.com/en-us/visualstudio/extensibility/language-server-protocol?view=vs-2022)
configurado.

Segundo: eu começarei estudando a CLI (Command Line Interface) do dotnet, algo
que um usuário do IDE usará poucas vezes no dia a dia. Portanto, alguns podem
considerar irrelevante parte desse conteúdo.

No fim do artigo, você terá percebido que alguns comandos podem ser bem longos e
repetitivos. Eu acho isso um pouco desconfortável. Por isso, escrevi um script
em bash que auxilia na criação de projetos sem que fiquemos digitando linhas
muito longas no terminal. O script e suas instruções de uso estão neste
[repo](https://github.com/dvths/cs-dotnet). Nessa primeira versão, ele apenas
ajuda na criação de um aplicativo de console com um diretório de testes
unitários, e na criação de uma estrutura para construção de uma APIWeb seguindo
os padrões SOLID/DDD (veremos mais sobre esses termos no futuro). Conforme a
necessidade irei incrementando o script para torná-lo mais útil.

Por fim, abordarei um pouco dos seguintes assuntos:

- Visão geral da CLI (Command Line Interface) .NET
- Estrutura de comandos
- Compilação condicional
- Aplicativos de Console e Entry Point
- Gerenciamento de pacotes
- Referencia entre projetos
- Testes unitários com xUnit e FluentAssertions

---

## CLI .NET -- Visão Geral

A Interface de Linha de Comando do .NET é uma ferramenta multiplataforma para
desenvolvimento, criação, execução e publicação de aplicações .NET e vem como
parte do conjunto de recursos do SDK.

Como a maioria das CLIs, a do .NET consiste no driver -- um arquivo executável
-- opções e argumentos de comando. O driver (dotnet) tem basicamente as
seguintes responsabilidades:

**Executar programas:** abrange o fluxo de desenvolvimento, isto é, execução de
testes, compilação do código e comandos de migração para atualizar projetos.

**Gerenciar dependências:** abrange instalação, remoção, limpeza após a
instalação de pacotes e atualização dos pacotes.

**Criar e publicar pacotes:** abrange tarefas como criar um pacote compactado e
efetuar um push do pacote para um registro, por exemplo (aos poucos veremos cada
uma dessas coisas).

## Estrutura de comando

```bash
$ dotnet <COMANDO> [opções-do-comando] [argumentos]
```

Quando passamos um comando para o executável (dotnet) inicia-se a execução
daquele comando, por exemplo:

```bash
 $ dotnet build
```

Esse comando compila um projeto e suas dependências.

```bash
$ dotnet new --list
```

Lista uma séries de opções de modelos pré preparados que você pode criar.

Cada comando define suas próprias opções e argumentos e também possuem uma opção
`--help` muito útil quando se sentir perdido.

Portanto, temos:

**Comando:**

Executa uma ação, compila o código, adiciona pacotes, referências ou publica o
código (veremos mais para frente).

**Argumentos:**

Argumentos que você passa na linha de comando são aqueles do comando invocado.
Por exemplo, quando você executa `dotnet publish my_app.csproj`, o argumento
`my_app.csproj` indica que o projeto a ser publicado é passado para o comando
publish.

**Opções:**

As opções são aquelas do comando invocado. Por exemplo:
`dotnet publish --output /build_output`, é a opção que indica o caminho da saída
do aplicativo a ser publicado e esse valor é passado para o comando publish.

Uma lista completa dos comandos do CLI .NET e seus detalhes pode ser consultada
[aqui](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet?source=recommendations#dotnet-commands),
mas lembre-se sempre da opção `--help`.

## Iniciando um Aplicativo console

Antes de iniciar um projeto, é importante saber que, ao instalar o SDK do .NET,
recebemos dezenas de modelos pré preparados como aplicativos de console,
bibliotecas de classes, projetos de testes unitários, aplicativos ASP.NET, ect.
Para listar esses modelos disponíveis, como vimos acima, executamos o comando
`dotnet new` com a opção `-l` ou `--list`.

Para iniciar um aplicativo de console, você pode executar:
`$ dotnet new console`. Contudo, você pode usar a opção `--help` tanto após
`new` quanto após console para verificar as opções disponíveis para cada. Se
fizer isso após o comando `new`, verá que existem duas opções (`-n` e `-o`) que
já direcionam a saída argumento passado para um diretório específico (eu não
consegui ver uma diferença prática entre essas opções apesar de estarem listados
de forma distinta). Portanto, se executar:

```bash
$ dotnet new console -o HelloWord
```

Verá que um diretório será criado com o valor passado para opção `-o` , no caso,
`HelloWord`. O mesmo o corre com a opção `n`. Crie um projeto console chamado
HelloWorld e dê uma espiada no que existe dentro dele:

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

Essa é a estrutura básica e, durante o artigo, veremos o que são todos esses
arquivos. Se abrirmos o arquivo `Program.cs`, teremos a instrução clássica
_Hello, World_. Não é nenhuma novidade que, se executarmos esse código, a
mensagem será impressa no console. Para fugir um pouco desse padrão, vamos
começar tentando algo diferente.

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

Você deve ter notado que o loop começou em 1 e não em 0, a razão para isso é que
o primeiro elemento será o caminho para o executável do seu programa. Para não
exibi-lo, começamos o loop da posição 1, o segundo elemento, que representa o
primeiro argumento passado para o nosso comando Echo.

Mas não ligue para os detalhes da implementação. A ideia aqui é entender o que
vai acontecer agora: Execute `dotnet build` e note que um novo diretório foi
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
armazena um conjunto de arquivos binários (assemblies) de saída do programa.
Esses binários incluem:

- o código do projeto em IL (Intermediate Language) -- vimos um pouco sobre isso
  no artigo anterior -- com extensão `.dll` (ou `exe`).

- arquivos de símbolo de depuração, com uma extensão `.pdb` (entenderemos melhor
  isso mais para frente).

- um arquivo `.deps.json`, que lista as dependências do aplicativo (ou
  biblioteca, se fosse o caso).

- um arquivo `.runtimeconfig.json`, que especifica o tempo de execução
  compartilhado e sua versão.

- pode conter também outras bibliotecas das quais o projeto depende (por meio de
  referências de projeto ou referências de pacote do NuGet que ainda veremos).

- por último, o binário executável do programa.

Vamos ver esse programa funcionando:

```bash
# ./bin/Debug

./Echo Olá, mundo!

```

A saída será `Olá, mundo!`. Mas, você não precisa navegar todas as vezes para o
diretório do binário para executar o codigo-fonte. Ao invés disso, execute
`dotnet run` e ele irá compilar e executar o programa para você. No caso do
nosso exemplo, será preciso fornecer os argumentos: `dotnet run Olá, mundo!`.

Nesse ponto, você deve estar se perguntado sobre o diretório `obj`. Se você
reparou, um diretório `Debug` também foi gerado lá e, entre outras coisas,
também se encontram os assemblies de saída do programa. A próxima pergunta que
pode vir a mente é: Por que dois diretórios para a mesma coisa?

Na verdade, não são necessariamente a mesma coisa. As dois diretórios armazenam
o código em IL, mas seus propósitos são distintos. E para deixar isso mais
claro, precisamos entender o processo de compilação.

Acontece que o uso coloquial da palavra "compilação" pode gerar certa confusão,
pelo menos para mim que sou meio lento. Compilar não necessariamente é o mesmo
que apenas gerar um arquivo executável. Em vez disso é um processo de várias
estágios que, de maneira geral, pode ser dividido em dois componentes:
**compilação** e **linking**.

![Compilação e Linker](./img/Compilação_Linking.jpg)

**Compilação** se refere ao processamento do código-fonte e a criação de um
arquivo "_objeto_" (por isso o diretório se chama "obj"), que são unidades
compiladas individuais. Por exemplo, se você compilar (mas não vincular) dois
arquivos separados, você terá dois arquivos objeto criados como saída, cada um
com o nome `QualquerCoisa.dll` (ou .exe), por exemplo. Cada um desses arquivos
contém uma tradução do seu código-fonte em IL -- mas você ainda não pode
executá-los! Porque ainda é preciso transformá-los em executáveis que seu
sistema operacional possa usar. É aí que entra o linking.

**Linking** (vinculação) refere-se à criação de um único arquivo executável a
partir de vários arquivos objeto. Isso significa que todos esses arquivos de
código compilados são vinculados e compilados em uma unidade assembly que pode
ser uma DLL ou EXE.

Com isso em mente, pense no seguinte: se não houvessem etapas separadas, caso
seu programa possuísse diversos arquivos, a alteração de um exigiria a
compilação (ou recompilação, no caso) de todos os outros. Isso tornaria o
processo bem lento dependendo do tamanho do projeto. Com a compilação de duas
etapas, com unidades compiladas isoladas no diretório `/obj`, podemos ter mais
controle sobres quais arquivos exatamente foram alterados, compilá-los
separadamente e depois vinculá-los no processo de build. O nome que se dá para
esse processo é **compilação condicional**.

É no processo de compilação que também ocorre a análise sintática e léxica do
código. Nesse ponto, se algo não estiver em conformidade com as especificações
do CLS ou CTS (que vimos no artigo anterior), o pocesso é interrompido e um
manipulador de erros integrado ao compilador te envia uma notificação apontando
o local do erro. Por isso, tanto em `/obj`, quanto em `/bin` os arquivos de
saída estão em um diretório chamado `Debug`, pois, essa compilação é feita de
forma a facilitar a depuração. Ou seja, seu programa é compilado com informações
de depuração e sem otimização. Porque otimização complica a depuração, afinal de
contas, a relação entre o código-fonte e as instruções otimização geradas é bem
mais complexa. Essa otimização, portanto, se faz necessária apenas quando
queremos distribuir nossa plicação. Mas, falaremos sobre _release_ mais para
frente.

As informações de depuração, que facilitam o debug são mapeadas naquele arquivo
com extensão `.pdb`. PDB significa Program Database, ele é gerado pelo Linker e
consiste em uma estrutura de dados empregada pelo compilador onde cada
identificador no código-fonte do programa é conectado com informações sobre sua
declaração, seu tipo, nível de escopo e, em alguns casos, a sua posição.

Os estágios de análise e síntese da compilação empregam informações a essa
estrutura (também chamada de tabela de símbolos) para verificar se os
identificadores usados foram especificados, para validar se as expressões e
atribuições são semanticamente precisas e para construir o IL de destino. Em
outras palavras, o arquivo `.pdb` mapeia vários componentes e instruções no
código-fonte para que o seu depurador possa usá-lo para localizar o arquivo de
origem e o local no executável no qual ele deve interromper um processo de
depuração.

Com base no que acabamos de saber e no que estudamos no primeiro artigo, podemos
fazer um desenho do processo de compilação que pode ser mais ou menos como este:

![Processo de Compilação](./img/Processo_Compilação.jpg)

Entendendo basicamente o processo de compilação e build, podemos nos voltar para
os outros dois arquivos: o de extensão`.cs` e `.csproj`.

---

Você pode ter notado que ao criar o modelo HelloWord a única linha de código era
`Console.WriteLine("Hello, World);`. Mas que o código da nossa implementação de
Echo tem uma estrutura diferente, com algumas coisas a mais.

Se olharmos apenas a estrutura do progrma Echo, em Program.cs, veremos:

```c

namespace Echo;

class Program
{
    public static void Main()
    {
        // code
    }
}

```

Como vimos na seção anterior, um programa C# consiste em uma ou mais unidades de
compilação, isto é, geralmente, seu programa irá possuir um ou vários arquivos
que serão compilados cada um em seu próprio arquivo objeto. Certamente, esses
dependem uns dos outros para o programa rodar e, além disso, muito
provavelmente, seu programa também dependerá de código de terceiros através de
bibliotecas que possuem suas próprias funcionalidades e que também geram suas
próprias unidades de compilação. Como organizar isso e evitar ambiguidade? Como
garantir que uma classe ou método não conflite com outra classe ou método que
possui o mesmo nome, por exemplo? É por isso que existem os _namespaces_.

A palavra-chave `namespace` é usada para declarar um escopo que contém um
conjunto de _códigos_ que se relacionam entre si. Isso mantém a organização
lógica do projeto e evita erros de compilação. Usar namespaces é uma boa prática
e em C# é intensamente usado para duas coisas principalmente:

1. Organização das inúmeras classes;

2. Para controlar o escopo dos nomes de classe e de método em projetos de
   maiores.

Conforme evoluirmos nos exemplos práticos ficará mais claro. Por enquanto esse
conceito é suficiente.

Agora pensemos no seguinte: Como o Runtime sabe por onde começar a executar o
programa?

A resposta está no método `Main()`. Este é o primeiro método invocado quando seu
projeto é executado. Isso significa que um programa deve ter apenas um método
`Main` também chamado de _Entry Point_ do programa. No entanto, é até possível
que uma aplicação tenha mais de um, só que antes de executá-lo, será necessário
informar o compilador por qual começar. Talvez vejamos isso na prática em um
outro artigo.

Você pode usar um namespace para organizar elementos de código e criar tipos
globalmente exclusivos.

O arquivo com extensão .csproj representa a base para configuração do projeto.
Ele interpreta todas as dependências do projeto, além de informações de
configuração como requisitos da plataforma, controle de versão, definições para
configuração de servidor e muito mais. Para quem programa em NodeJs, podemos
dizer que é algo semelhante ao `pakage.json`, ou `Cargo.toml`, para os
programadores Rust.

Até aqui eu citei algumas vezes a palavra dependência sem definir exatamente o
que é uma. A seguir, vamos estudar mais a fundo como podemos usá-las.

---

Que é uma dependência e como administrá-las?

Uma dependência é um conjunto de código reutilizável que realiza alguma
funcionalidade dentro do contexto em que o aplicativo está inserido. Em outras
palavras, é um código desenvolvido por você ou outras pessoas que funciona para
um determinado contexto e, cujo qual o seu aplicativo depende. O .NET tem muitas
bibliotecas principais que cuidam de praticamente tudo, desde gerenciamento de
arquivos até HTTP e compactação de arquivos. Também existem pacotes (ou
bibliotecas, ou dependências) que tratam de casos ainda mais específico e que
são gerenciados pelo NuGet, o gerenciador de pacotes do .NET.

O NuGet define como os pacotes .NET são criados, hospedados, consumidos e
fornece as ferramentas para cada uma dessas funções. Bibliotecas, no geral, nos
ajudam a obter um código melhor, mais conciso, o que muitas vezes nos poupa
tempo e facilita a manutenção do programa. Contudo, é preciso levar em
consideração o tamanho e a quantidade de biblioteca usadas, pois o número de
dependências pode gerar um grande volume, o que pode ser um ponto de preocupação
durante o desenvolvimento caso você tenha recursos limitados de hardware, por
exemplo.

Também é preciso verificar se a licença concedida à biblioteca abrange o uso
pretendido, seja comercial, pessoal ou acadêmico. E, por fim, mas não menos
importante, se a manutenção da biblioteca está ativa. Existem bibliotecas
preteridas ou que não são atualizadas há muito tempo. Antes de usarmos a
primeira biblioteca de terceiros, vamos ver um exemplo de uso de uma das
bibliotecas padrão do .NET: `System.IO.File`.


-------->>>> TODO


Vamos fugir um pouco do padrão e gerar uma mensagem “Hello World!” de uma
maneira um pouco diferente. Dentro do diretório do projeto HelloWorld, abra o
arquivo Program.cs no seu editor favorito. Digite ou copie e cole o código a
seguir: using System; using System.IO;

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
