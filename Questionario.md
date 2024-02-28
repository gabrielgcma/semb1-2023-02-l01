# Questionário Sistemas Embarcados I  

```
Aluno: Gabriel Carneiro Marques Amado
Matrícula: 12111ECP002
```

## 1. Explique brevemente o que é compilação cruzada (***cross-compiling***) e para que ela serve.

A compilação cruzada é um processo por meio do qual pode-se compilar programas para uma plataforma diferente da plataforma host. Um exemplo de cross-compiling seria compilar para a plataforma Android usando uma plataforma Windows como host.

## 2. O que é um código de inicialização ou ***startup*** e qual sua finalidade?

A finalidade do código de startup é preparar o ambiente/plataforma de execução para que todos os requisitos de programa e de arquitetura do processador sejam cumpridos antes de chamar a função `main()`. 

Em desenvolvimento bare metal para a plataforma ARM Cortex-M, o código de inicialização é responsável por:

1. Declarar e inicializar a stack;
2. Inicializar e preencher o vetor de interrupções de acordo com os requisitos do processador e do integrador (fabricante do microcontrolador);
3. Implementar o `Reset_handler()`, código que é executado sempre que o processador é iniciado/reiniciado;
4. Copiar o conteúdo da seção `.data` da memória flash para a SRAM do microcontrolador;
5. Preencher o endereço das variáveis não inicializadas, na seção `.bss`, com o valor 0;
6. Executar a função `main()`.

## 3. Sobre o utilitário **make** e o arquivo **Makefile responda**:

#### (a) Explique com suas palavras o que é e para que serve o **Makefile**.

O Makefile é um arquivo interpretado pelo utiliário `make`. Ele contém instruções específicas e/ou generalizadas de compilação, gerenciamento de arquivos gerados/compilados e de dependências, de forma a possibilitar a automatização completa do processo de compilação. 

#### (b) Descreva brevemente o processo realizado pelo utilitário **make** para compilar um programa.

O `make` interpreta as regras, variáveis e diretivas contidas no arquivo `Makefile` e as executa. 

#### (c) Qual é a sintaxe utilizada para criar um novo **target**?

Um *target* pode ser tanto um nome de arquivo a ser gerado, como `main.o`, como também o nome de uma ação a ser executada, por exemplo, `clean`. Para criar um novo target, basta especificar o nome do target e sua receita de compilação/geração. 

#### (d) Como são definidas as dependências de um **target**, para que elas são utilizadas?

As dependências do target são definidas logo após sua declaração:

```make
target: pré-requisitos
```

Elas são utilizadas para garantir que um arquivo que possui dependências esteja sempre atualizado de acordo com suas dependências. Quando uma de suas dependências for atualizada ou deixar de existir, por exemplo, o target deverá ser recompilado e o `make` saberá disso na próxima execução do comando. 

#### (e) O que são as regras do **Makefile**, qual a diferença entre regras implícitas e explícitas?

Uma regra possui a seguinte estrutura:
```make
targets: prerequesites
  recipe
```

A regra *explícita* define quando e como refazer o(s) target(s), explicitando seus nomes e os nomes de suas dependências, além de conter a receita.

A regra *implícita* usa da generalização, por meio de recursos de manipulação de string/filename embutidos no `make` para que os targets sejam definidos como uma classe de arquivos (arquivos `.o`, por exemplo), ao invés de nomes específicos. O mesmo pode ser aplicado à lista de dependências do target (arquivos `.c`, por exemplo). Também contém uma receita.

## 4. Sobre a arquitetura **ARM Cortex-M** responda:

### (a) Explique o conjunto de instruções ***Thumb*** e suas principais vantagens na arquitetura ARM. Como o conjunto de instruções ***Thumb*** opera em conjunto com o conjunto de instruções ARM?

O conjunto de instruções Thumb é um subconjunto do conjunto principal de instruções (ARM). Enquanto o conjunto principal contém instruções de 32 bits, o Thumb contém instruções de 16 bits. Toda instrução Thumb tem uma instrução equivalente de 32 bits no conjunto principal. Por operar na mesma configuração de registros, o modo Thumb permite interoperabilidade entre os modos ARM e Thumb. 

Os modos ARM e Thumb operam em conjunto por meio da decodificação das instruções Thumb de 16 bits lidas em instruções ARM de 32 bits, em tempo real e sem perda de performance.

A maior vantagem do modo Thumb é a redução de memória ocupada: código em Thumb é 65% menor que o código ARM. Logo, torna-se uma ferramenta essencial em contextos de memória restrita e requisitos de baixo consumo de energia. 

### (b) Explique as diferenças entre as arquiteturas ***ARM Load/Store*** e ***Register/Register***.

Na arquitetura Load/Store, as únicas operações que acessam a memória diretamente são as instruções de *load* e *store*, e os dados são movidos entre o processador e a memória via registros. Nessa arquitetura, as instruções são divididas em duas categorias: acesso de memória (*load* e *store*) e operações da ALU -- que só ocorrem entre registros, isto é, todos os operandos para instruções de lógica e aritmética devem vir de registros, ou devem ser fornecidos pela instrução (e não pela memória). Todos os resultados devem ser salvos em algum registro (e não na memória).

A arquitetura Register/Memory permite que operações possam ser executadas não somente em registros, mas também diretamente na memória. Isso significa que, por exemplo, um dos operandos de uma operação ADD esteja armazenado em um registro, e o outro na memória. 

### (c) Os processadores **ARM Cortex-M** oferecem diversos recursos que podem ser explorados por sistemas baseados em **RTOS** (***Real Time Operating Systems***). Por exemplo, a separação da execução do código em níveis de acesso e diferentes modos de operação. Explique detalhadamente como funciona os níveis de acesso de execução de código e os modos de operação nos processadores **ARM Cortex-M**.

Processadores Cortex-M3 e Cortex-M4 têm dois estados de operação e dois modos de operação. 

Os estados são:

- *Estado de depuração (debug state)*: quando o processador é interrompido (pelo debugger ou ao atingir um breakpoint), ele entra no modo de depuração e para de executar instruções.

- *Thumb state*: estado no qual o processador está executando código (instruções Thumb). Não existe um "ARM State" porque os processadores Cortex-M não suportam o conjunto de instruções ARM de 32 bits.

Além disso, há níveis de acesso privilegiado e não privilegiado:

- *Privilegiado*: pode-se acessar todos os recursos do processador.

- *Sem privilégios*: algumas regiões de memória ficam inacessíveis, e algumas operações não podem ser feitas.

No Thumb state, dois modos de operação podem ocorrer:

- *Modo handler (handler mode)*: quando o processador está executando um tratador de exceção, como um Interrupt Service Routine (ISR). Nesse modo, o processador sempre tem nível de acesso privilegiado. 

- *Modo thread (thread mode)*: quando o processador está executando o código da aplicação normalmente. Pode-se estar em nível de acesso privilegiado ou sem privilégios, o que é controlado por um registro específico chamado "CONTROL".

Via software, pode-se alternar do Thread mode privilegiado para o Thread mode não privilegiado, mas não o contrário -- para isso, é necessário que uma interrupção faça a troca.

Sistemas embarcados robustos, como aplicações RTOS, podem ser construídos por meio dessa separação de níveis de acesso. Regiões de memória críticas (como a memória usada pelo kernel de um sistema operacional embarcado, executando em modo privilegiado) são protegidas da execução de aplicações de usuário, executadas em modo não privilegiado. Pode-se, dessa maneira, configurar permissões de acesso à memória e e evitar que uma aplicação de usuário corrompa a memória e periféricos usados pelo kernel do sistema operacional. Caso uma aplicação de usuário trave, outras aplicações e o kernel do sistema operacional não são afetados diretamente.

### (d) Explique como os processadores ARM tratam as exceções e as interrupções. Quais são os diferentes tipos de exceção e como elas são priorizadas? Descreva a estratégia de **group priority** e **sub-priority** presente nesse processo.

Na terminologia ARM, uma interrupção é um tipo de exceção. 

As exceções numeradas de 1 a 15 são exceções do sistema, e as exceções 16 e acima são entradas de interrupção. A maioria das exceções, incluindo todas as interrupções, têm prioridades programáveis, e algumas exceções de sistema têm prioridade fixa.

| Número | Tipo de exceção | Prioridade | Descrição       |
|-------|-------|-------------------|-----------------------------------------|
| 1     | Reset     | -3 (mais alta) | Reset       |
| 2     | NMI     | -2     | Non-Maskable Interrupt (não pode ser ignorada) |
| 3     | Hard Fault     | -1     | Qualquer falha de hardware que não tenha um handler específico|
| 4     | MemManage Fault     | Programável | Falha de gerenciamento de memória: violação da MPU ou execução em endereços não executáveis.       |
| 5     | Bus Fault | Programável | Erro de barramento.       |
| 6     | Usage Fault     | Programável | Exceções de erros do programa.       |
| 7-10    | Reservado     | NA | -       |
| 11     | SVC     | Programável | SuperVisor Call; comumente usado em um ambiente de OS para que aplicações possam usar serviços do sistema.   |
| 12     | Debug monitor     | Programável | Para eventos de debug como breakpoints, watchpoints, etc.       |
| 13     | Reservado     | NA | -       |
| 14     | PendSV     | Programável | Pendable service call. Usado em contextos de OS.        |
| 15     | SYSTICK    | Programável | System Tick Timer, gerada por um periférico de timer dentro do processador. Pode ser usado pelo OS ou pode ser usado por um periférico de timer.      |
| 16 | Interrupção #0 | Programável | Pode ser gerada por periféricos no chip ou de fontes externas.
| 17 | Interrupção #1 | Programável | Pode ser gerada por periféricos no chip ou de fontes externas.
| ... | ... | ... | Pode ser gerado por periféricos no chip ou de fontes externas.
| 255 | Interrupção #239 | Programável | Pode ser gerado por periféricos no chip ou de fontes externas.

As 240 interrupções descritas são fornecidas pelo NVIC.

As prioridades das exceções (com exceção das exceções do sistema, com prioridades definidas e imutáveis) são programáveis. Como são definidas em registros de níveis de prioridade, o fabricante integrador do chip pode arbitrar a maneira com a qual vai montar o sistema de prioridades, podendo usar de 3 a 8 bits do registro. 

Além disso, podem haver grupos de subprioridade dentro de um nível de prioridade. Os bits de prioridade, neste caso, são sempre os mais significativos dentro dos bits de prioridade, enquanto os de subprioridade são os menos significativos do bloco.

O nível de prioridade de grupo define se uma exceção/interrupção tem prioridade em relação a outra de outro grupo. O nível de subprioridade define se uma exceção/interrupção tem prioridade em relação a outra do mesmo grupo.

### (e) Qual a diferença entre os registradores **CPSR** (***Current Program Status Register***) e **SPSR** (***Saved Program Status Register***)?

Foram ambos introduzidos na arquitetura ARMv3, mas não estão presentes na arquitetura ARMv7-M e ARMv7E-M os Cortex-M3 e Cortex-M4, respectivamente. 

O CPSR é responsável por guardar informações acerca do programa sendo executado no momento, como flags de número negativo, de carry e de overflow, bem como o nível de privilégio do processador durante aquela execução.

O SPSR guarda o status atual do processador, para posterior recuperação. Por exemplo, quando uma interrupção ou exceção chega ao processador, o atual CSPR é salvo no SPSR. Após tratar a interrupção, o processador consegue recuperar o status anterior por meio do SPSR.

### (f) Qual a finalidade do **LR** (***Link Register***)?

O Link Register armazena o endereço de retorno quando uma função ou subrotina é chamada. Quando a execução da função ou subrotina chamada acabar, o processador pode voltar ao lugar anterior por meio do carregamento do valor de LR no PC.

### (g) Qual o propósito do Program Status Register (PSR) nos processadores ARM?

O Program Status Register é um registro composto por três registros de status:

- Application PSR (APSR)
- Execution PSR (EPSR)
- Interrupt PSR (IPSR)

Todos os três podem ser acessados como um registro só. 

No APSR, são encontrados: flags de número negativo, zero, carry, overflow, etc.

No IPSR, é salvo o número de exceção que o processador está tratando.

No EPSR, são encontrados bits de instruções interrompíveis e continuáveis, e bits de IF-THEN, para execução condicional.

O propósito desse registro é o de armazenar informações críticas do status atual do processador, assim como CPSR fazia em arquiteturas anteriores. 

### (h) O que é a tabela de vetores de interrupção?

A tabela de vetores de interrupção é a sequência de ponteiros para as funções que vão lidar com as interrupções, sejam elas emitidas pelo processador ou por periféricos do microcontrolador.

### (i) Qual a finalidade do NVIC (**Nested Vectored Interrupt Controller**) nos microcontroladores ARM e como ele pode ser utilizado em aplicações de tempo real?

O NVIC é um controlador de interrupções programável presente em processadores Cortex-M3 e Cortex-M4. Fornece uma ponte entre o processador e as interrupções de periféricos + interrupções de sistema (hardware, processador, etc). Os endereços de memória do NVIC são fixos.

Os fabricantes integradores podem determinar quantos sinais de interrupção o NVIC deve fornecer, e quantos/quais são os níveis de prioridade programáveis suportados. 

### (j) Em modo de execução normal, o Cortex-M pode fazer uma chamada de função usando a instrução **BL**, que muda o **PC** para o endereço de destino e salva o ponto de execução atual no registador **LR**. Ao final da função, é possível recuperar esse contexto usando uma instrução **BX LR**, por exemplo, que atualiza o **PC** para o ponto anterior. No entanto, quando acontece uma interrupção, o **LR** é preenchido com um valor completamente  diferente,  chamado  de  **EXC_RETURN**.  Explique  o  funcionamento  desse  mecanismo  e especifique como o **Cortex-M** consegue fazer o retorno da interrupção. 

### (k) Qual  a  diferença  no  salvamento  de  contexto,  durante  a  chegada  de  uma  interrupção,  entre  os processadores Cortex-M3 e Cortex M4F (com ponto flutuante)? Descreva em termos de tempo e também de uso da pilha. Explique também o que é ***lazy stack*** e como ele é configurado. 


## Referências

### Básicas

[1] [STM32F411xC Datasheet](https://www.st.com/resource/en/datasheet/stm32f411ce.pdf)

[2] [RM0383 Reference Manual](https://www.st.com/resource/en/reference_manual/rm0383-stm32f411xce-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)

[3] [Using the GNU Compiler Collection (GCC)](https://gcc.gnu.org/onlinedocs/gcc/index.html)

[4] [GNU Make](https://www.gnu.org/software/make/manual/html_node/index.html)

[5] YIU, Joseph. **The Definitive Guide to ARM Cortex-M3 and Cortex-M4 Processors**. 3. ed. Newnes, 2013.

[6] TANENBAUM, A. S., AUSTIN, Todd. **Structured Computer Organization**. 4. ed. Prentice Hall, 2012.

### Vídeos Microsoft Teams

[5] [05 - Introdução à arquitetura de computadores](https://web.microsoftstream.com/embed/channel/f6b3a0de-e6f3-4652-b2d5-f1164032498a?app=microsoftteams&sort=undefined&l=pt-br#)

[6] [06 - Arquitetura Cortex-M Parte 1/2](https://web.microsoftstream.com/embed/channel/f6b3a0de-e6f3-4652-b2d5-f1164032498a?app=microsoftteams&sort=undefined&l=pt-br#)

[7] [07 - Arquitetura Cortex-M Parte 2/2](https://web.microsoftstream.com/embed/channel/f6b3a0de-e6f3-4652-b2d5-f1164032498a?app=microsoftteams&sort=undefined&l=pt-br#)

[8] [08 - Microcontroladores STM32](https://web.microsoftstream.com/embed/channel/f6b3a0de-e6f3-4652-b2d5-f1164032498a?app=microsoftteams&sort=undefined&l=pt-br#)

[9] [10 - Introdução à arquitetura de computadores](https://web.microsoftstream.com/embed/channel/f6b3a0de-e6f3-4652-b2d5-f1164032498a?app=microsoftteams&sort=undefined&l=pt-br#)

### Material Suplementar

[5] [A General Overview of What Happens Before main()](https://embeddedartistry.com/blog/2019/04/08/a-general-overview-of-what-happens-before-main/)
 
[6] [Bare metal embedded lecture-1: Build process](https://youtu.be/qWqlkCLmZoE?si=mn5yDnJYudQ1PpZH)
 
[7] [Bare metal embedded lecture-2: Makefile and analyzing relocatable obj file](https://youtu.be/Bsq6P1B8JqI?si=yuNLPj3JQ-2IT1yo)
 
[8] [Bare metal embedded lecture-3: Writing MCU startup file from scratch](https://youtu.be/2Hm8eEHsgls?si=c27MpZ47ApiMSwHR)
 
[9] [Lecture 15: Booting Process](https://youtu.be/3brOzLJmeek?si=MsHRUEJP8zofjwJQ)
