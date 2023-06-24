# PE Structure - From TinyPE to Hello World

Nesse artigo vou tentar apresentar um pouco da estrutura de um PE (Portable Executable). 

## Pré-requisitos

1. Diferença básica entre um PE do disco e na memória: apenas entender que um PE em disco é como uma receita que vai ser usada pelo Windows para montar o programa, que realmente vai poder ser executado, em memória.
2. PE-Bear: um programa que vai mostrar os headers de algum PE. Existem outros também.
3. Ghidra: vai nos permitir ver o programa “puro”. Existem outros (IDA PRO).
4. Microsoft Visual Studio: não é obrigatório, mas recomendado.

# Começando do NÃO básico

Ao invés de começarmos por algo pequeno e fácil, vou mostrar antes a monstruosidade que um simples programa Hello World é por dentro. Crie um programa qualquer que simplesmente printe “Hello World”. Farei isso usando o MSVC.

![Untitled](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled.png)

Compile ele (Ctrl-B). Estarei utilizando a Realease e x86. Mas não faz muita diferença.

![Untitled](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%201.png)

Encontre onde o programa está. Vamos analisá-lo agora. Primeiro note o tamanho que um simples Hello World tem: aproximadamente 10KB (isso pode variar). Abra-o no PE-Bear:

![Untitled](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%202.png)

Eu não espero que você entenda nada acima. Mas note quantas informações um simples Hello World carrega. Clique em Imports, ele vai mostrar todas as Dlls e funções que o Hello World importou. É muita coisa para você entender, né? Eu imagino o que você esperava seria algo como:

1. Algumas informações no PE para identificar que realmente é um executável (magic number).
2. Alguns outros metadados, como a versão (x86/x64).
3. Obviamente, o nosso código em assembly, equivalente ao Hello World que escrevemos.
4. Alguma informação falando aonde que a nossa função “main” está (um endereço), já que o código vai precisar começar a ser executado de algum canto, certo?

E você não está muito errado, essa lista com quatro pontos é mais ou menos o mínimo necessário para que um PE possa ser lido e executado pelo Windows. 

Note que esse Hello World contém muito mais coisas do que o mínimo necessário, pois ele está importando algumas bibliotecas (C Runtime Library) e carregando consigo informações de Debug que o próprio MSVC colocou.

# Começando do básico

Ao invés de analisarmos um PE grande como o Hello World, vamos montar um [TinyPE](http://www.phreedom.org/research/tinype/). Isto é, um PE com pouquíssimas coisas. Entenda que é possível montar um PE ainda menor do que vou mostrar, mas não farei isso pois atrapalha a legibilidade do PE.

Abra o MSVC, crie um programa mínimo que não utilize nenhuma biblioteca:

![Untitled](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%203.png)

Vamos utilizar o compilador e linker do MSVC para gerarmos o binário, pois assim temos um pouco mais de controle sobre o que o MSVC vai ou não adicionar no binário. Vá em View → Terminal e digite:

```powershell
cl /nologo /c /Od .\TinyPE.cpp
link /nologo /EMITPOGOPHASEINFO /emittoolversioninfo:no /ENTRY:main /NODEFAULTLIB /SUBSYSTEM:WINDOWS .\TinyPE.obj
```

Pesquise um pouco sobre o que cada flag dessas faz. Mas basicamente entenda que ele vai montar um binário sem nenhuma biblioteca (por isso não vamos conseguir printar um Hello World) e sem informações de debug.

![Untitled](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%204.png)

Abrindo no PE-bear, perceba que muita coisa já sumiu. Agora nosso PE possui apenas 1.5KB. Perceba que, por exemplo, o Rich Hdr (Rich Header) sumiu completamente. Se você pesquisar um pouco, vai descobrir que ele é um header adicionado pelo MSVC apenas para debug. Não é nem um pouco necessário para o PE. Note também que as tabs Disasm e General são informacoes adicionais que o PE-bear mostra, não estão no PE de verdade. Então podemos ignorar. 

Agora que temos um TinyPE, é uma boa hora para começar a tentar entender quais informações esses headers guardam. Spoiler: muitas dessas informações não são necessárias para que um PE seja válido. Então é menos confuso do que parece. Recomendo a leitura desse artigo da [Microsoft sobre a estrutura do PE](https://learn.microsoft.com/en-us/windows/win32/debug/pe-format).

O que eu vou mostrar agora é que um PE precisa de 3 headers: o DOS Header, o [File Header](https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-image_file_header) e o [Optional Header](https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-image_optional_header32?source=recommendations). 

Obs.: a junção do File Header e do Optional Header normalmente ganha o nome de [NT Header](https://learn.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-image_nt_headers32?source=recommendations) (NT Header = File Header + Optional Header).

## DOS Header

Obs.: as duas imagens representam a mesma coisa e apenas o que está marcado de vermelho é necessário.

![PE-Bear](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%205.png)

PE-Bear

![Struct inside winnt.h](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%206.png)

Struct inside winnt.h

Resumo: possui o magic number e um ponteiro (offset) para o NT Header (File Header + Optional Header). Apenas isso.

## File Header

Obs.: as duas imagens representam a mesma coisa e apenas o que está marcado de vermelho é necessário.

![PE-Bear](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%207.png)

PE-Bear

![Struct inside winnt.h](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%208.png)

Struct inside winnt.h

Resumo: possui alguns metadados e a quantidade de sessões que o PE vai ter. O nosso TinyPE vai ter apenas uma sessão (.text). Vou falar sobre sessões depois. Mas apenas para você ter ideia, nosso Hello World tinha 5 sessões.

## Optional Header

Obs.: as duas imagens representam a mesma coisa e apenas o que está marcado de vermelho é importante para nós agora. Além disso, apesar do nome, o optional header não é nada opcional.

![PE-Bear](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%209.png)

PE-Bear

![Struct inside winnt.h](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%2010.png)

Struct inside winnt.h

Resumo: possui alguns metadados, um ponteiro (offset) para o código de entrada do programa (no nosso caso a main). No final, ele possui um array (Data Directory) que é muito importante, pois guarda endereços para informações importantes, como por exemplo de funções importadas e exportadas. Mas como nós não estamos usando isso, podemos literalmente ignorar. Mas vou falar mais sobre esse Data Directory depois.

## Continuando

Pesquise sobre o que cada field desses headers faz. Você não precisa entender tudo, apenas leia para ter uma noção e foque nos que estão sublinhados de vermelho. Uma coisa legal de se fazer é usar algum Hex Editor (HxD, por exemplo) para modificar algum desses valores e ver o que quebra. Por exemplo, mude o magic number do DOS Header de MZ (0x5a4d) para alguma outra coisa e veja como o programa não vai iniciar.

Apesar de parecer complicado, veja que isso tudo são apenas headers que contém metadados para que o Windows possa depois montar o programa em memória.

## Sections Headers

Logo abaixo do Optional Header vamos encontrar as Sections Headers. Basicamente vai ser um array de headers. No nosso caso, só temos um section header, que vai ter o nome de .text, que vai ser exatamente onde nosso código vai ficar armazenado. Eu não quero complicar muito. Não quero falar aqui sobre os outros tipos de sections que podem existir (o nosso Hello World, por exemplo, possuía .text, .rdata, .data, .rsrc, .reloc). A única coisa que eu quero que você entenda é que esses headers possuem um nome (pode ser qualquer nome que você quiser) e um ponteiro (offset) para a sessão correspondente.

![Nosso TinyPE possui apenas uma sessão: .text](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%2011.png)

Nosso TinyPE possui apenas uma sessão: .text

## Sections

Onde está nosso código? Sim, aquele return 42; que escrevemos. Ele vai ficar armazenado em um canto qualquer do PE. Essa sessão vai receber um nome (pode ser qualquer nome, mas o MSVC, e outros compiladores, nomeiam de .text) e uma proteção (não vou explicar, pode ignorar). Ou seja, em algum canto do nosso PE vai ter uma sessao (literalmente um intervalo de memória qualquer: sei lá, começando do byte 1000 até 1200) que vai conter nosso código compilado.

![Untitled](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%2012.png)

Esse é um print do Ghidra. Ignore os endereços na esquerda. Só quero que você entenda que nosso código vai ficar gravado em algum canto do PE, e que esse algum canto normalmente recebe o nome de .text (pois guarda o código).

Mas o que diabos é uma sessão afinal? Nada. Pegue um intervalo de memória qualquer no PE, dê um nome legal e diga que nele você vai guardar X informações (código, strings, imagens). Isso é uma sessão. Vou explicar mais depois, mas a sessão .text normalmente guarda código, a sessão .rdata normalmente guarda strings, a sessão .rsrc normalmente guarda resources - como imagens -, mas isso NÃO é obrigatório. Tecnicamente você pode dar o nome que quisessee tecnicamente poderia criar APENAS uma sessão para tudo (sei lá, chame-a de .program e jogue absolutamente tudo lá).

# Resumo

O PE tem três headers: DOS Header, o File Header e o Optional Header. Eles contêm muitas coisas que são inúteis para a gente agora. Basicamente a única coisa útil é o Entry Point que vai indicar onde está nosso código inicial (no nosso caso a função main). Aonde nosso código vai estar armazenado? Em algum canto do PE (abaixo de todos esses headers). Esse algum canto normalmente se chama .text. 

Pronto. Esse é a estrutura básica de um PE.

---

# Usando o Ghidra

Se você nunca usou o Ghidra antes (ou sei lá, o IDA PRO), talvez a interface inicial parece complicada. Talvez seja uma boa ideia brincar um pouco antes (veja uns vídeos) antes de continuar. Mas vamos utilizar somente coisas muito básicas. Abra o TinyPE no Ghidra. Perceba logo que nosso PE tem apenas duas coisas, os headers e a sessão .text.

![Untitled](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%2013.png)

Veja que o Ghidra identifica o DOS Header, NT Header e Sections Headers (eles estão minimizados para não poluir muito a tela).

![Veja que do endereço 400000 até 4001a0 temos os headers. Depois disso temos alguns bytes vazios (padding).](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%2014.png)

Veja que do endereço 400000 até 4001a0 temos os headers. Depois disso temos alguns bytes vazios (padding).

Depois desses headers, temos a section .text como eu já tinha mostrado.

![Untitled](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%2015.png)

Recomendo você brincar um pouco. Vá no DOS Header, veja que se você somar 400000 + e_lfanew isso aponta exatamente para o NT Header. Análise esses valores. O binário é pequeno o suficiente para você não se perder. Gaste algumas horas ai.

---

# Adicionando mais código - .text

Agora, que já sabemos como é a estrutura de um TinyPE, vamos começar a aumentar nosso PE para ele começar a se assemelhar a um PE de verdade. Vamos começar adicionando mais código.

![Untitled](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%2016.png)

![test1()](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%2017.png)

test1()

![test2()](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%2018.png)

test2()

![main()](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%2019.png)

main()

Perceba que quando adicionamos mais funções, apenas a sessão .text é modificada. Bem simples.

---

# Adicionando dados - .data .rdata

![Untitled](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%2020.png)

Para compilar isso vamos ter que desativar o Buffer Security Check (não sei exatamente o porquê). Veja a nova flag /GS-

 

```powershell
cl /nologo /c /Od /GS- .\TinyPE.cpp
link /nologo /EMITPOGOPHASEINFO /emittoolversioninfo:no /ENTRY:main /NODEFAULTLIB /SUBSYSTEM:WINDOWS .\TinyPE.obj
```

![Ignore a sessão .reloc por enquanto](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%2021.png)

Ignore a sessão .reloc por enquanto

Temos quatro “dados”. Uma string literal - isto é, uma string que não vai mudar - uma variável global inicializada, uma variavel global não inicializada e uma variável na stack. Você vai ver que a string vai ser guardada em uma sessao .rdata, isto é, read only data. Vou explicar depois, mas essa sessão só possui permissão de leitura. Isto é, quando o programa está em memória rodando, é apenas possível ler os valores nessa sessão, se você tentar modificar ela de algum jeito (a = “Hello World”) vai dar erro. 

![Untitled](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%2022.png)

As variáveis b e c globais vão ser guardadas na sessão .data, pois podem ser lidas e modificadas.

![Untitled](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%2023.png)

Perceba que a variável global b recebe o valor 16 (0x10) já em tempo de compilação. Ou seja, esse valor já foi gravado no PE. Enquanto a variável global c apenas tem seu espaço de memória reservado. Seu valor vai ser dado apenas em tempo de execução.

Perceba que a variável d não é colocada em nenhuma sessão, pois seu valor vai ser armazenado na stack. Vou explicar aonde que diabos a stack fica depois. Mas vou logo dizendo que ela existe só em tempo de execução. Ou seja, não existe uma stack no PE em disco. Apenas quando ele está rodando em memóriaque essa sessão vai existir.

Recomendo você brincar mais um pouco. Crie structs, arrays e depois analise no Ghidra. Veja como essas informações são armazenadas no PE.

---

# Adicionando Resources - .rsrc

Por simplicidade eu não irei adicionar nenhum resource manualmente. Essa sessão normalmente guarda recursos genéricos, como as imagens de um binário. Você nunca tinha pensado onde que os ícones e imagens de um binário são guardados? Para testar, abra qualquer binário no Ghidra que tenha um ícone. Eu escolhi o próprio PE-Bear

![Untitled](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%2024.png)

Eu ACHO que se você quisesse colocar sons dentro dessa sessão também seria totalmente viável. Isso vai depender de qual compilador você está usando para fazer essa integração de colocar sons nessa área de resources e depois no .text conseguir usá-los. Enfim, .rscs guarda os resources genéricos. Bem legal.

---

# Adicionando Realocations - .reloc

Não vou entrar em muitos detalhes dessa sessão. Mas basicamente o que eu entendo dela pode ser resumida nessa resposta do [StackOverflow](https://stackoverflow.com/questions/44337712/why-relocations-reloc-section-in-executable-file):

That because of pointers and address reference look to this code :

```
int i;
int *ptr = &i;
```

If the linker assumed an image base of `0x10000`, the address of the variable `i` will end up containing something like `0x12004`. At the memory used to hold the pointer "`ptr`", the linker will have written out `0x12004`, since that's the address of the variable `i`. If the loader for whatever reason decided to load the file at a base address of 0x70000, the address of `i` would be `0x72004`. The *.reloc section is a list of places in the image where the **difference** between the linker assumed load address and the actual load address needs* to be factored in.

---

# Adicionando Debug Informations - Debug Data

Escreva um programa return 42; e compile ele, mas dessa vez não vamos retirar as informações de debug

```powershell
cl /nologo /c /O1 .\TinyPE.cpp
link /nologo /emittoolversioninfo:no /ENTRY:main /NODEFAULTLIB /SUBSYSTEM:WINDOWS .\TinyPE.obj
```

As informações de debug são armazenadas na sessão .rdata. Quais informações são guardadas vai depender do compilador. Portanto, não irei me aprofundar mais nesse artigo.

---

# Resumo

Acabamos de ver onde que variáveis globais, strings, ícones e etc. são guardadas. Agora você já tem condição de abrir o nosso Hello World.exe no Ghidra e ler os headers e sections. O próximo passo vai ser entender o que são os DATA DIRECTORIES e um pouco sobre como as funções são importadas e exportadas. Spoiler: um data directory é apenas um “shortcut” para informações realmente importantes, como qual DLL e quais funções um PE está importando. Essas informações são armazenadas em sessões que já vimos, como por exemplo a .rdata.

---

# Data Directories

Abra nosso Hello World, ou qualquer PE, no PE-Bear e veja o array de data directories no optional header:

![Untitled](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%2025.png)

Parece complicado, mas não é. Isso é só uma lista de offsets que apontam para sessões já conhecidas. Faça um teste: abra o Ghidra, abra o IMAGE_OPTIONAL_HEADER → IMAGE_DATA_DIRECTORY e abra o segundo (conhecido também como import directory, veja a ordem na imagem acima).

![Untitled](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%2026.png)

![Untitled](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%2027.png)

Esse valor é um offset para algo chamado IMAGE_IMPORT_DESCRIPTOR que vai conter informações sobre quais Dlls e funções o PE está importando. Cada DLL importada vai ter um IMAGE_IMPORT_DESCRIPTOR. Se voce seguir esse offset vai ver que essas informações estão armazenadas na sessão .rdata. Nada de mais, certo?

![Untitled](PE%20Structure%20-%20From%20TinyPE%20to%20Hello%20World%2023f6899ba8604d5f9267bb0a16a6cbef/Untitled%2028.png)

No meu caso do Hello World, existiam várias DLLs sendo importadas. A última era a famosa Kernel32.dll. Veja que esse descriptor possui alguns offsets interessantes. Siga eles usando o Ghidra. Você vai ver que ele aponta para os nomes das funções que estão sendo importadas. Eu não vou explicar como que em runtime o endereço dessas funções é encontrado.

Pense um pouco. Se você quisesse colocar em um PE que você está usando uma DLL chamada CoolDll.dll e três funções dela func1, func2 e func3 qual seria sua intuição? A minha seria colocar em algum canto, nesse caso no data directory, o endereço de uma sessão com o nome da DLL e suas funções. É exatamente o que ocorre. Não pense ainda em como essa importação funciona em runtime, é um pouco mais complicado. A única coisa que você precisa entender é que esses data directories vão ser apenas offsets para regiões que guardam algum tipo de informação. Se você tivesse uma DLL que exporta algumas funções, essa informação, sobre qual o nome da sua DLL e quais funçõesela está exportando seria guardada em alguma sessão (ex.: .rdata) e haveria uma entrada no IMAGE_DATA_DIRECTORY apontando para essa região. De novo, DATA DIRECTORIES só são offsets para informações importantes.

---

# Resumo

Data directories são apenas offsets para informações importantes. O que vai está armazenado nas sessões vai depender de qual data directory estamos falando: import, export, debug, …?