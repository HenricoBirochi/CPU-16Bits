# CPU de 16 bits – N2 (Logisim-evolution)

Arquivo principal: `CPU.circ`  
Ferramenta: **Logisim-evolution 3.8.0**  
Disciplina: **Arquitetura de Computadores**  
Avaliação: **N2 – Continuação do projeto “N1 – Projeto MIPS”**  
Professor responsável: **Vinícius S. Borges**

- Link do Vídeo da Resolução:
  https://youtu.be/diWdncHh0Os

- Link do Repositório no GitHub:
  https://github.com/HenricoBirochi/CPU-16Bits

---

## 1. Contexto do projeto

Este repositório contém a implementação de uma **CPU de 16 bits multi–ciclo** desenvolvida no **Logisim‑evolution** como continuação direta do projeto da N1 (**“N1 – Projeto MIPS”**).

Enquanto o trabalho da N1 se concentrou em um **processador MIPS simplificado em ciclo único**, esta N2:

- Adota um **conjunto de registradores próprio**.
- Implementa uma **arquitetura multi–ciclo**, com controle explícito de micro–etapas.
- Integra **memória de programa**, **memória de dados**, **pilha** e **periféricos mapeados em hardware** (TTY, vídeo e display em matriz).

O objetivo pedagógico é aprofundar:

- O entendimento do **datapath** de uma CPU.
- O projeto e a implementação de uma **unidade de controle sequencial**.
- A interação entre **CPU, memórias e dispositivos de E/S** em nível lógico–digital.

---

## 2. Características da arquitetura

### 2.1 Parâmetros principais

- **Largura de dados:** 16 bits  
- **Largura de endereços:** 16 bits  
- **Tamanho lógico do espaço de endereçamento:** 2¹⁶ posições  
- **Modelo de execução:** CPU **multi–ciclo**, com contador de ciclos dedicado  
- **Clock:** 1 sinal de clock global (`CLK`), com enable (`clk_enb`)  

### 2.2 Registradores

Principais registradores visíveis no circuito:

- **PC – `CONTADOR_DE_PROGRAMA` (16 bits)**  
  Responsável pelo endereço da próxima instrução a ser buscada na ROM.

- **IR – `REGISTRADOR_DE_INSTRUCAO` (16 bits)**  
  Armazena a instrução corrente durante as fases de decodificação e execução.

- **MAR – `REGISTRADOR_DE_ENDERECO_DE_MEMORIA` (16 bits)**  
  Fornece endereços para acesso à memória de dados (RAM) ou a regiões mapeadas (vídeo, periféricos).

- **AC – `ACUMULADOR` (16 bits)**  
  Registrador acumulador, principal operando da ULA e destino de muitos resultados.

- **RB – `REGISTRADOR_B` (16 bits)**  
  Segundo operando da ULA, usado em operações aritméticas/lógicas e em manipulações de memória/pilha.

- **Registradores auxiliares – `REGISTRADORES_Y_E_Y`**  
  Registradores intermediários utilizados para implementação de micro–etapas do datapath (por exemplo, pipeline de endereços, resultados parciais, etc.).

- **Registrador de flags – `FLAGS`**  
  Conjunto de registradores de um bit (Zero, Carry e possivelmente outros) atualizados pela ULA e consultados pela unidade de controle.

- **`REGISTRADOR_TTY`**  
  Registrador de saída associado ao periférico TTY, representando o dado a ser “impresso” no terminal.

- **`REGISTER_SAIDA`**  
  Registrador de saída auxiliar para monitoramento/depuração ou ligação a outros periféricos.

### 2.3 Unidade de Controle

A unidade de controle é implementada de forma distribuída, por meio de:

- Um **contador de ciclos** (`CONTADOR_CICLO`, 4 bits).  
- Lógica combinacional que, a partir de:
  - `opcode` da instrução (campos do `REGISTRADOR_DE_INSTRUCAO`), e
  - valor do `CONTADOR_CICLO`,

  gera todos os **sinais de controle** necessários:

- Seleção e escrita de registradores (`*_EN`, `*_LOAD`).  
- Operações da ULA (`ADD`, `SUB`, `A_XOR_B`, etc.).  
- Acesso à memória de dados (`ADDRESS_WRITE`, `ADDRESS_SELECT`, `RAM_IN`).  
- Controle de pilha (`STACK_UP`, `STACK_DOWN`, `STACK_STORE`, `STACK_DATA_OUT`).  
- Escrita em periféricos (`REGISTRADOR_TTY`, placa de vídeo, buffer de vídeo).

---

## 3. Visão geral do datapath

O datapath da CPU, agrupado no circuito superior `CIRCUITO`, possui os seguintes elementos:

- **PC → ROM → IR**  
  - O PC é usado como endereço para a ROM.  
  - A palavra de 16 bits lida da ROM é armazenada em `REGISTRADOR_DE_INSTRUCAO`.

- **IR → Unidade de Controle**  
  - O conteúdo do IR alimenta a lógica de decodificação de instruções.  
  - Dependendo do opcode e campos auxiliares, a execução prossegue ao longo de múltiplos ciclos.

- **AC, RB, registradores auxiliares e ULA**  
  - Dados trafegam entre `ACUMULADOR`, `REGISTRADOR_B` e `REGISTRADORES_Y_E_Y` via barramentos.  
  - A ULA realiza operações aritmético–lógicas de 16 bits.  
  - O resultado pode ser:
    - Regravado no AC,  
    - Armazenado em registradores intermediários,  
    - Escrito em RAM,  
    - Enviado a periféricos (TTY, vídeo, etc.).

- **MAR, RAM e periféricos mapeados**  
  - O `REGISTRADOR_DE_ENDERECO_DE_MEMORIA` define o endereço para operações de leitura/escrita na RAM.  
  - Endereços específicos podem ser mapeados para:
    - Regiões de pilha,  
    - Buffer de vídeo,  
    - Outros recursos, conforme convenções adotadas no programa.

---

## 4. ULA (Unidade Lógica e Aritmética)

A ULA é encapsulada no subcircuito **`ULA`**, com largura de 16 bits.  
Principais sinais de controle e funções observáveis:

- **Operações aritméticas**  
  - `ADD` – soma dos operandos A e B.  
  - `SUB` – subtração (A − B ou B − A, conforme configuração interna).

- **Operações lógicas**  
  - `A_XOR_B` – XOR bit a bit entre A e B.

- **Flags**  
  - Atualização de flags no circuito `FLAGS` (por exemplo: Zero, Carry).  
  - Essas flags são consultadas pela unidade de controle para implementar desvios condicionais.

### 4.1 Entradas e saídas típicas

- Entradas:
  - Operando A: normalmente ligado ao `ACUMULADOR`.  
  - Operando B: normalmente ligado ao `REGISTRADOR_B` (ou a registradores auxiliares).  
  - Sinais de controle de operação (`ADD`, `SUB`, `A_XOR_B`).

- Saídas:
  - Resultado de 16 bits (principal).  
  - Sinais internos para atualização de flags (Zero, Carry, etc.).

---

## 5. Controle por ciclos (multi–ciclo)

O comportamento multi–ciclo é regido pelo subcircuito **`CONTADOR_CICLO`** (4 bits), que define a fase de execução de cada instrução.

### 5.1 Fases conceituais de uma instrução

Embora os nomes dos estados não estejam explicitamente rotulados, conceitualmente o fluxo é:

1. **Busca (Fetch)**  
   - PC alimenta a ROM.  
   - A instrução é carregada em `REGISTRADOR_DE_INSTRUCAO`.

2. **Decodificação (Decode)**  
   - A unidade de controle interpreta o opcode e campos da instrução.  
   - Sinais para carregar registradores, preparar endereços, etc.

3. **Execução (Execute)**  
   - Ativação da ULA, acesso à RAM, manipulação de pilha e/ou periféricos.  
   - Atualização de registradores intermediários.

4. **Write-back / atualização de estado**  
   - Gravação de resultados no `ACUMULADOR` ou em registradores de destino.  
   - Atualização de PC (próxima instrução, desvio, retorno de subrotina, etc.).  
   - Reinicialização do `CONTADOR_CICLO` para a próxima instrução.

### 5.2 Sinais típicos oriundos da unidade de controle

- Acesso a memória:
  - `ADDRESS_WRITE`, `ADDRESS_SELECT`, `BUFFER_ADDRESS_WRITE_ENB`, `RAM_IN`.

- Pilha:
  - `STACK_UP`, `STACK_DOWN` – incremento/decremento do ponteiro de pilha.  
  - `STACK_STORE` – habilita escrita na pilha.  
  - `STACK_DATA_OUT` – habilita leitura do topo da pilha.

- ULA:
  - `ADD`, `SUB`, `A_XOR_B` e possíveis sinais derivados.

---

## 6. Memória, pilha e periféricos

### 6.1 Memória de programa – ROM

- Componente: `ROM` (Logisim-evolution).  
- Larguras:
  - Endereço: 16 bits.  
  - Dados: 16 bits.

O conteúdo da ROM é configurado diretamente no arquivo `.circ` (aba **Contents…** da ROM) e corresponde ao programa de teste corrente da CPU.  
Para alterar o programa:

1. Clique com o botão direito na ROM.  
2. Selecione **Edit Contents…**.  
3. Atualize as palavras de 16 bits conforme a codificação de instruções definida para o trabalho.  
4. Salve e retorne ao circuito.

### 6.2 Memória de dados – RAM

- Largura de dados: 16 bits.  
- Endereços: fornecidos por `REGISTRADOR_DE_ENDERECO_DE_MEMORIA`.  
- Sinais de controle típicos:
  - Enable de leitura/escrita.  
  - Sinais de seleção de origem do endereço (`ADDRESS_SELECT`).

A RAM é usada tanto para variáveis de programa quanto para regiões reservadas (por exemplo, pilha ou memória mapeada para periféricos).

### 6.3 Pilha

A pilha é manipulada por um conjunto específico de sinais:

- `STACK_UP` – movimenta o ponteiro de pilha em direção ao topo (push).  
- `STACK_DOWN` – movimenta o ponteiro de pilha em direção à base (pop).  
- `STACK_STORE` – habilita escrita do dado atual no topo da pilha.  
- `STACK_DATA_OUT` – habilita leitura do dado no topo da pilha.

Na prática, a pilha está implementada usando:

- Um registrador/contador de posição de topo.  
- Uma área dedicada de RAM (endereços reservados) ou um bloco próprio, dependendo da configuração adotada.

### 6.4 TTY (terminal de saída)

O periférico TTY é conectado via:

- `REGISTRADOR_TTY` (16 bits), que armazena o valor a ser exibido.  
- Um componente de saída no circuito top‑level (por exemplo, um “TTY” ou display numérico).

Instruções específicas podem escrever no `REGISTRADOR_TTY`, permitindo observar resultados de operações em formato textual ou numérico.

### 6.5 Vídeo e display

A interface de vídeo é composta pelos subcircuitos:

- **`PLACA_DE_VIDEO`** – responsável por traduzir dados e endereços de vídeo provenientes da CPU em escritas no buffer de vídeo.  
- **`BUFFEFR_VIDEO`** – armazena o conteúdo atual da tela (framebuffer).  
- **`DISPLAY`** – display em matriz 8×8 (parâmetros `matrixcols = 8`, `matrixrows = 8`).

A CPU pode escrever em posições específicas do buffer de vídeo, alterando o padrão de pontos exibidos na matriz (por exemplo, para representar caracteres, sprites simples ou padrões geométricos).

---

## 7. Execução e simulação no Logisim-evolution

### 7.1 Requisitos

- **Logisim-evolution 3.8.0** (ou versão compatível).  
  Outras versões podem abrir o arquivo, mas é recomendado validar a largura de barramentos, memórias e componentes de vídeo.

### 7.2 Passos básicos

1. **Abrir o projeto**  
   - Iniciar o Logisim-evolution.  
   - Abrir o arquivo `CPU.circ`.  
   - Certificar-se de que o circuito ativo é `CIRCUITO`.

2. **Configurar o programa na ROM (se necessário)**  
   - Editar o conteúdo da ROM (ver Seção 6.1) para carregar o programa desejado.

3. **Inicializar registradores**  
   - Zerar o PC e outros registradores relevantes via sinais de reset.  
   - Usar `INSTRUCT_RESET` para garantir estado inicial consistente do IR e da unidade de controle.  
   - Habilitar o clock (`clk_enb`) quando iniciar a execução.

4. **Executar a CPU**  
   - Ativar **Simulate → Ticks Enabled** no Logisim.  
   - Ajustar a frequência do clock, se necessário (mais lento para depuração).  
   - Monitorar:
     - PC, IR, AC, RB, FLAGS, MAR.  
     - Sinais de controle em `CONTADOR_CICLO`.  
     - Atividade na RAM, pilha, TTY e display.

5. **Depuração passo a passo**  
   - Desativar `Ticks Enabled`.  
   - Utilizar **Simulate → Tick Once** para avançar um ciclo por vez.  
   - Acompanhar o valor do `CONTADOR_CICLO` para entender em qual micro–etapa a instrução se encontra.

---

## 8. Equipe

| Integrante                          | RA        |
|------------------------------------|-----------|
| Vítor Agostino Braghittoni         | 081230024 |
| Henrico Birochi                    | 081230027 |
| Nicholas Birochi                   | 081230038 |
| Edgar Camacho Seabra Ribeiro       | 081230039 |
| Vinicius Yamaguti                  | 081220040 |

**Disciplina:** Arquitetura de Computadores – N2  
**Professor:** Vinícius S. Borges

---

## 9. Referências para inspiração

- Vídeo (YouTube):  
  https://www.youtube.com/watch?v=GRM1Oc3erew

- Material complementar (Google Drive):  
  https://drive.google.com/file/d/1kzTs-QVto3cTFqfeCo2QPnT4D38jn9qm/view
