# Estrutura de um SO

## Sumário
- [Elementos do sistema operacional](#elementos-do-sistema-operacional)
- [Elementos de hardware](#elementos-de-hardware)
- [Interrupções e exceções](#interrupções-e-exceções)
- [Níveis de privilégio](#níveis-de-privilégio)
- [Chamadas de sistema](#chamadas-de-sistema)

# Elementos do sistema operacional

Um sistema operacional não é um bloco único e fechado de software executando sobre o hardware. Na verdade, ele é composto de diversos componentes com objetivos e funcionalidades complementares. Alguns dos componentes mais relevantes de um sistema operacional típico são:

- Núcleo: é o coração do sistema operacional, responsável pela gerência dos recursos do hardware usados pelas aplicações. Ele também implementa as principais abstrações utilizadas pelos aplicativos e programas utilitários.

- Código de inicialização: (boot code) Uma série de tarefas complexas, como reconhecer os dispositivos instalados, testá-los e configurá-los adequadamente para seu uso posterior. Carregar o núcleo do sistema operacional em memória e iniciar sua execução.

- Drivers: módulos de código específicos para acessar os dispositivos físicos. Existe um driver para cada tipo de dispositivo, como discos rígidos SATA, portas USB, placas gráfica, etc. Muitas vezes o driver é construído pelo próprio fabricante do hardware e fornecido em forma compilada (em linguagem de máquina) para ser acoplado ao restante do sistema operacional.

- Programas utilitários: são programas que facilitam o uso do sistema computacional, fornecendo funcionalidades complementares ao núcleo, como formatação de discos e mídias, configuração de dispositivos, manipulação de arquivos (mover, copiar, apagar), interpretador de comandos, terminal, interface gráfica, gerência de janelas, etc.

## Elementos de hardware

O sistema operacional executa diretamente sobre o hardware, abstraindo e gerenciando recursos para as aplicações. Para que o SO possa cumprir suas funções com eficiência e confiabilidade, o hardware deve prover algumas funcionalidades básicas:

- Processadores: Responsáveipela execução das instruções das aplicações
- Áreas de Memoria: Armazena o programa a ser executado junto com os dados
  - Unidade de Gerência de Memória (MMU - Memory Management Unit): controlador específico mediador, que pode estar fisicamente dentro do próprio chip do processador, responsável por analisar cada endereço de memória acessado pelo processador, validá-lo, efetuar conversões de endereçamento porventura necessárias e executar a operação solicitada pelo processador
(leitura ou escrita de uma posição de memória)
  - Cache: uma memoria que permite armazenar os dados mais recentemente lidos da memória, para melhorar o desempenho.
- Barramentos:  Meio que os principais elementos constituintes do computador estão interligados
  - Barramento de endereços: indica a posição de memória (ou o dispositivo) a acessar 
  - Barramento de controle: indica a operação a efetuar (leitura ou escrita)
  - Barramento de dados: transporta a informação a ser transferida entre o processador e a memória ou um controlador de dispositivo.

## Interrupções e exceções

A comunicação entre o processador e os dispositivos se dá através do acesso às portas de entrada/saída, que podem ser lidas e/ou escritas pelo processsador. Esse acesso é feito por iniciativa do processador, quando este precisa ler ou escrever dados no dispositivo.

Muitas vezes um dispositivo precisa informar o processador rapidamente sobre um evento interno, como a chegada de um pacote de rede, um clique de mouse ou a conclusão de uma operação de disco. Neste caso, o controlador tem duas alternativas:

- aguardar até que o processador o consulte, o que poderá ser demorado caso o processador esteja ocupado com outras tarefas (o que geralmente ocorre);
- notificar o processador, enviando a ele uma requisição de interrupção (IRQ – Interrupt ReQuest) através do barramento de controle.

Ao receber a requisição de interrupção, os circuitos do processador suspendem seu fluxo de execução corrente e desviam para um endereço predefinido, onde se encontra uma rotina de tratamento de interrupção (interrupt handler)

A maioria das interrupções recebidas pelo processador têm como origem eventos externos a ele, ocorridos nos dispositivos periféricos e reportados por seus controladores. Entretanto, alguns eventos gerados pelo próprio processador podem ocasionar o desvio da execução usando o mesmo mecanismo das interrupções: são as exceções. Ações como instruções ilegais (inexistentes ou com operandos inválidos), tentativas de divisão por zero ou outros erros de software disparam exceções no processador, que resultam na ativação de uma rotina de tratamento de exceção, usando o mesmo mecanismo das interrupções (e a mesma tabela de endereços de funções)

Quando ocorre uma interrupção, a execução é desviada para um endereço predefinido (desvio incondicional) e o processador passa a operar em nível núcleo (elevação de privilégio).

## Níveis de privilégio

Núcleo, drivers, utilitários e aplicações são constituídos basicamente de código de máquina. Todavia, devem ser diferenciados em sua capacidade de interagir com o hardware: enquanto o núcleo e os drivers devem ter pleno acesso ao hardware, para poder configurá-lo e gerenciá-lo, os aplicativos e utilitários devem ter acesso mais restrito a ele, para não interferir nas configurações e na gerência, o que poderia desestabilizar o sistema inteiro. 

Para permitir diferenciar os privilégios de execução dos diferentes tipos de software, os processadores modernos implementam níveis de privilégio de execução. Esses níveis são controlados por flags especiais na CPU, que podem ser ajustados em função do tipo de código em execução. Os processadores no padrão Intel x86, por exemplo, dispõem de 4 níveis de privilégio (0 . . . 3, sendo 0 o nível mais privilegiado), embora a maioria dos sistemas operacionais construídos para esses processadores só use os níveis extremos (0 para o núcleo do sistema operacional e 3 para utilitários e aplicações)

Na forma mais simples desse esquema, podemos considerar dois níveis básicos de privilégio:

- Nível núcleo: também denominado nível supervisor, sistema, monitor ou ainda kernel space. Para um código executando nesse nível, todas as funcionalidades do processador estão disponíveis e todos os recursos internos (registradores e portas de entrada/saída)
e áreas de memória podem ser acessados. Ao ser ligado durante a inicialização do computador, o processador entra em operação neste nível.
- Nível usuário: também chamado userspace, somente um subconjunto das instruções do processador e registradores estão disponíveis. Por exemplo,
instruções consideradas “perigosas”, como RESET (reiniciar o processador) e IN/OUT (acessar portas de entrada/saída), são proibidas. Caso o código em execução tente executar uma instrução proibida, o hardware irá gerar uma exceção

## Chamadas de sistema

A ativação de uma rotina do núcleo usando o mecanismo de elevação de privilégio é denominada chamada de sistema (system call ou syscall). Os sistemas operacionais definem chamadas de sistema para todas as operações envolvendo o acesso a recursos de baixo nível(periféricos, arquivos, alocação de memória, etc.) ou abstrações lógicas (criação e encerramento de tarefas, operadores de sincronização, etc.). Geralmente as chamadas de sistema são oferecidas para as aplicações em modo usuário através de uma biblioteca do sistema (system library), que prepara os parâmetros, invoca a chamada e, no retorno desta, devolve à aplicação os resultados obtidos.

A maioria dos sistemas operacionais implementa centenas de chamadas de sistema distintas, para as mais diversas finalidades. O conjunto de chamadas de sistema oferecidas por um núcleo define a API (Application Programming Interface) do sistema operacional. 

O conjunto de chamadas de sistema de um SO pode ser dividido nas seguintes grandes áreas:
- Gestão de processos: criar, carregar código, terminar, esperar, ler/mudar atributos.
- Gestão da memória: alocar/liberar/modificar áreas de memória.
- Gestão de arquivos: criar, remover, abrir, fechar, ler, escrever, ler/mudar atributos.
- Comunicação: criar/destruir canais de comunicação, receber/enviar dados.
- Gestão de dispositivos: ler/mudar configurações, ler/escrever dados.
- Gestão do sistema: ler/mudar data e hora, desligar/suspender/reiniciar o sistema