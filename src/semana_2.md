Proximo artigo

- Gerenciamento de pacotes
- Referencia entre projetos
- Testes unitários com xUnit e FluentAssertions

Uma dependência é um conjunto de código reutilizável que realiza alguma
funcionalidade dentro do contexto em que o aplicativo está inserido. Em outras
palavras, é um código desenvolvido por você ou outras pessoas que é "empacotado"
e disponibilizado para que a comunidade.

É fundamental que que qualquer plataforma de desenvolvimento moderna possua um
mecanismo através do qual os desenvolvedores possam criar, compartilhar e
consumir código útil. No caso do .NET esse mecanismo é chamado de NuGet esse
mecanismo é chamado de NuGet, que define como os pacotes para o .NET são
criados, hospedados, consumidos e fornece as ferramentas para cada uma dessas
funções.

Como sabemos, o .NET possui uma biblioteca padrão de classes que nos ajudam com
muitas coisas, desde gerenciamento de arquivos até HTTP. Vamos explorar o uso de
uma dessas bibliotecas.

Vamos gerar uma mensagem “Olá, Mundo!” de uma maneira diferente agora. Em um
novo aplicativo de console, abra o arquivo Program.cs no editor e cole esse
código.

```c
class Program
{
    static void Main()
    {
        string path = @"./message.txt";
        if (!File.Exists(path))
        {
            // Crie um arquivo e grave uma mensagem nele.
            using StreamWriter message = File.CreateText(path);
            message.WriteLine("Olá, Mundo");
        }
        // Leia o conteúdo do arquivo criado.
        using (StreamReader message = File.OpenText(path))
        {
            string content;
            while ((content = message.ReadLine()) != null)
            {
                Console.WriteLine(content);
            }
        }
    }
}
```

Se você executar esse código verá a mensagem "Olá, Mundo!", mas isso está vindo
de dentro de um arquivo de texto.

Este código cria um arquivo no diretório atual chamado `message.txt` e grava
nele a mensagem “Olá, Mundo!”, depois, enquanto existirem bytes para serem
lidos, ele imprime no console o conteúdo desse arquivo. Quando isso se torna
falso, o programa encerra.

Muita coisa pode ser dita a respeito desse código. Por exemplo, no primeiro
artigo conhecemos os conceito de código gerenciado e não gerenciado e como o
.NET trabalha com isso. `File`, aqui, é um exemplo de tipo gerenciado que acessa
recursos não gerenciados pelo CLR (no caso, cria um arquivo e o acessa no
contexto do dispositivo). Note que o arquivo `.txt` foi criado na pasta atual.
Há muitos outros tipos de recursos não gerenciados e tipos de biblioteca de
classes que os encapsula.

`File`, `StreamWriter` e `StreamReader`, são tipos que lidam com fluxos de dados
permitindo leitura e gravação em arquivos definidos na biblioteca `IO`
(Input/Output) no namespace `System.IO` que fornece suporte básico a arquivos e
diretórios do sistema operacional. O `Console.WriteLine()` também é um tipo
definido no namespace `System`, ou seja, `System.Console.WriteLine`.

Se você estiver se perguntando o motivo de não ver isso no código, eu te digo
que nem sempre foi assim. Antes da versão 10 do C#, era necessário declarar
namespaces individuais usando a diretiva `using` no topo de cada arquivo, por
exemplo:

```c
using System;
using System.Io;

class Program
{
 ...

```

Mas o uso de alguns namespaces eram (e ainda são) muitos comuns, portanto, com a
versão 10 do C# foi estabelecido as
[diretivas globais de uso](https://github.com/dotnet/csharplang/blob/main/proposals/csharp-10.0/GlobalUsingDirective.md),
que, entre outros detalhes, permite ao compilador considerar certos namespaces
para todo o projeto. Isso quer dizer o SDK, importa implicitamente os seguintes
namespaces:

- System
- System.Collections.Generic
- System.IO
- System.Linq
- System.Net.Http
- System.Threading
- System.Threading.Tasks

Contudo, a diretiva `using`, que veremos mais para frente, ainda é necessária
para casos que fogem desse padrão.

Com isso, espero que tenha ficado mais claro o uso dos namespaces e bibliotecas
padrão. Agora, podemos usar uma dependência de terceiros.

------------->>>

Nesse momento, meus estudos são endereçados para testes unitários. Uma
ferramenta popular para testes unitários e focada na comunidade .NET é o xUnit.
Segundo a documentação, “é a tecnologia mais recente para testes de unidade em
aplicativos .NET”. Portanto, vamos utilizá-la. Antes de iniciar, eu pensei em
criar uma calculadora simples. Uma classe MyCalculator que possui um método para
cada operação básica: Soma, Subtração, Multiplicação e Divisão. Quão difícil
deve ser testar isso? Vamos ver: Primeiro vamos criar um projeto do tipo
console: $ dotnet new console -o MyCalculator

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
