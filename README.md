# Cubo Mágico 3D

##  Integrantes
* Lucas Ribeiro d'Azevedo
* Pablo Felipe
* Pedro Alves
* Vic Rocha

##  Proposta do Projeto
Este projeto consiste na simulação interativa de um Cubo Mágico (Rubik's Cube) em 3D rodando diretamente no navegador. Ele foi construído em HTML5 e JavaScript, utilizando a biblioteca **Three.js** para a renderização, aplicação de materiais e mapeamento de interações em um ambiente tridimensional.

##  Requisitos do Projeto
O desenvolvimento cumpriu as especificações exigidas para a avaliação da disciplina:

**Requisitos básicos implementados:**
* Construção de um cubo 3x3x3 composto por 27 cubinhos individuais.
* Faces com as cores corretas e interior do cubo na cor preta.
* Rotação de faces com animação suave através da interpolação de ângulos.
* Controle de câmera orbital utilizando o módulo `OrbitControls`.
* Arquitetura baseada em agrupamentos: uso de um `THREE.Group` temporário para ancorar as peças durante a rotação e devolvê-las à cena após o movimento.

**Extras:**
* Implementação da rotação para todas as 6 faces e fatias centrais do cubo.

##  Como Executar
1. O projeto não exige instalação de pacotes locais (como NPM), pois o Three.js é consumido via CDN.
2. Basta abrir o arquivo `Cubo-3D.html` em qualquer navegador web moderno.
3. Para girar a câmera, clique no fundo da tela e arraste. Use o *scroll* para aplicar zoom.

---

##  Arquitetura do projeto: Funcionalidade das Funções
O código foi estruturado de maneira modular para facilitar o gerenciamento do estado do cubo. As principais funções e como elas se complementam são:

* **`init()`:** É o ponto de partida. Configura os pilares do Three.js: a Cena, a Câmera e o Renderizador. Também invoca a criação do cubo e adiciona os *event listeners* de mouse e teclado.
* **`criarCuboMagico()`:** Cria os 27 pequenos `THREE.Mesh` com `BoxGeometry`. Utiliza um laço triplo (x, y, z) para mapear os materiais de cor para as extremidades e posicionar os cubos no espaço com o espaçamento adequado.
* **`onCubeClick()`:** Complementa o `init()`. Ao clicar, usa o `Raycaster` para descobrir qual peça e face foram tocadas. Salva essa informação no objeto global `selecaoAtual`, destacando a peça clicada (`setHex(0x444444)`) e travando a movimentação da câmera.
* **`onKeyDown()` e `buscarEDispararGiro()`:** Trabalham juntas após a peça ser selecionada. O `onKeyDown` capta as setas direcionais do teclado e, consultando a `selecaoAtual`, decide qual eixo vai girar. Ele envia o eixo, o valor da camada (coordenada) e a direção matemática para `buscarEDispararGiro()`, que por sua vez filtra as 9 peças corretas que compõem a fatia escolhida.
* **`rotateSlice()`:** É o motor de animação. Pega as 9 peças filtradas, coloca-as num `THREE.Group` temporário e aplica incrementos no ângulo escolhido (via `requestAnimationFrame`) até atingir 90 graus ($\pi / 2$). Ao finalizar, destrói o grupo, devolve as peças à hierarquia principal, libera a câmera e finaliza a interação.
* **`animate()`:** O loop constante de renderização, que garante que qualquer atualização visual (da câmera do `OrbitControls` ou das rotações do cubo) seja desenhada no *canvas* a cada *frame*.

---

##  Melhorias em relação a o Cubo3D 1.0 (primeira versão)
Durante o desenvolvimento, substituímos um modelo de movimentação baseado em **arraste de mouse (`mousedown`/`mouseup`)** pelo atual modelo híbrido de **clique e teclado (`click` + `keydown`)**.

**Anterior:**
A versão anterior capturava as coordenadas do momento em que o mouse era pressionado (`mousedown`) e do momento em que era solto (`mouseup`), calculando a diferença em pixels (`deltaX` e `deltaY`). O algoritmo dependia de um aglomerado complexo de lógicas (`if/else`) para "adivinhar" o que o usuário queria fazer: tentar deduzir o eixo de giro (`x`, `y` ou `z`) e a direção baseando-se apenas em qual Delta era maior na tela 2D. 

**Atual:**
1. **Fim da Ambiguidade:** Em um espaço 3D, converter um movimento retilíneo 2D (mouse na tela) para uma rotação volumétrica costuma ser confuso dependendo do ângulo da câmera. A nova versão elimina esse erro: o usuário marca um pivô com maior precisão (clique) e ordena a ação (setas para linha ou coluna).
2. **Conflito com o `OrbitControls`:** O método antigo frequentemente causava conflitos onde o usuário tentava girar a fatia do cubo e acabava girando a câmera inteira. Agora, a separação de *estado* resolve isso: clicar na peça seleciona e desativa os controles da câmera; ao terminar a rotação (ou clicar no fundo para cancelar), a câmera é liberada novamente.
3. **Feedback Visual:** A adição da função `limparSelecaoVisual()`, combinada à seleção atual, providencia uma experiência tátil, escurecendo sutilmente a peça alvo, algo que não existia na versão de arraste baseada em deltas matemáticos invisíveis ao usuário.

##  4. Onde IA foi utilizada
O uso de Inteligência Artificial Generativa foi integrado ao processo de desenvolvimento para solucionar desafios de lógica geométrica tridimensional e refatoração de código, agilizando a entrega das seguintes funcionalidades:

1. **Cálculo da Lógica de Seleção de Peças por Camada:**
   * **Como foi utilizada:** A IA auxiliou na modelagem matemática para filtrar quais das 27 peças pertenciam a uma fatia específica no momento do giro. A partir das coordenadas globais `(x, y, z)` dos cubinhos individuais, a IA sugeriu a lógica condicional exata para que a função `buscarEDispararGiro()` identificasse as 9 peças corretas que compartilhavam o mesmo plano, independentemente das rotações acumuladas na cena.

2. **Gerenciamento de Matrizes de Rotação e Ciclo de Vida do `THREE.Group`:**
   * **Como foi utilizada:** Um dos maiores desafios técnicos foi evitar que as peças saíssem de órbita ou ficassem desalinhadas após sucessivos giros de 90 graus. A IA foi utilizada para estruturar o fluxo da função `rotateSlice()`. Ela ajudou a projetar o algoritmo que:
     1. Transfere temporariamente os cubinhos para um grupo pivô.
     2. Realiza a interpolação suave do ângulo via `requestAnimationFrame`.
     3. Atualiza as matrizes de transformação globais dos cubinhos (`mesh.updateMatrixWorld()`) antes de devolvê-los à cena principal, garantindo que o estado do cubo permanecesse consistente para os movimentos seguintes.

3. **Refact de bugs**

##  5. Responsabilidades de Cada Membro
Com base no histórico de desenvolvimento e contribuições no repositório, as tarefas foram distribuídas da seguinte forma entre os integrantes do grupo:

* **Pablo Felipe:**
  * **Fundação do Projeto:** Inicialização do repositório, configuração dos pilares do Three.js e renderização do cenário básico.
  * **Estrutura 3D:** Desenvolvimento da lógica e renderização do grid 3x3x3 composto pelos 27 cubinhos.
  * **Controles & Regras:** Implementação do sistema de controle de rotação por setas de teclado e criação da funcionalidade de contabilização de movimentos.

* **Vic Rocha:**
  * **Estética e Iluminação:** Configuração do sistema de iluminação da cena e aplicação das cores iniciais nos cubinhos.
  * **Interação com Mouse:** Desenvolvimento do rastreamento de posição do mouse e primeira lógica de movimentação de faces.

* **Pedro Alves:**
  * **Câmera Orbital:** Implementação e ajuste da câmera funcional para navegação e visualização em 360 graus.
  * **Otimização Visual:** Desenvolvimento e refatoração da lógica para pintar de preto as faces internas/não visíveis do cubo mágico.
  * **Gerenciamento do Repositório:** Revisão de código e integração (*merge*) de funcionalidades ramificadas (*branches*).

* **Lucas Ribeiro d'Azevedo:**
  * **Refinamento de Rotação:** Correções e ajustes finos na física e animação de rotação do cubo.
  * **Experiência do Usuário (UX):** Adição de linhas e colunas guia para facilitar a noção espacial e a percepção de movimento das seções do cubo.
  * **Refatoração:** Limpeza de código, remoção de comentários obsoletos e padronização do espaçamento.