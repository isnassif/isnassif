# 👩🏻‍💻 Ikhlas Nassif

**`Desenvolvedor Back-end`**

Me chamo Ikhlas Santos Nassif, tenho 19 anos e sou natural da Bahia. Concluí o ensino médio no colégio Nobre, tenho também curso de inglês pelo CCAA. Atualmente, estou cursando Engenharia de Computação na UEFS. Sou proativo, dedicado, responsável e autônomo, conheça um pouco mais de mim pelo meu [Linkedin](https://www.linkedin.com/in/ikhlasnassif/).

<p align="left">
    <a href="https://github.com/isnassif?tab=repositories&sort=stargazers">
        <img 
            alt="Total de estrelas" 
            title="Total de estrelas GitHub" 
            src="https://custom-icon-badges.demolab.com/github/stars/isnassif?color=55960c&style=for-the-badge&labelColor=488207&logo=star&label=estrelas"
        />
    </a>
    <a href="https://github.com/isnassif?tab=followers">
        <img 
            alt="Seguidores" 
            title="Me siga no GitHub" 
            src="https://custom-icon-badges.demolab.com/github/followers/isnassif?color=236ad3&labelColor=1155ba&style=for-the-badge&logo=github&label=Seguidores&logoColor=white"
        />
    </a>
</p>

<h2 id="control">Unidade de Controle</h2>
<p>
A Unidade de Controle, é implementada no módulo control_unity e funciona como o elemento principal do projeto, ela é responsável por coordenar todo o fluxo do sistema, começando pela instanciação de todos os componentes principais, como as memórias ROM e RAM e a ALU, coordenação e geração do clock utilizado, sincronismo das chaves utilizadas, ativação dos algoritmos de redimensionamento e escrita ordenada no Framebuffer, com um resultado final exibido pelo driver VGA, a seguir, será explicado de forma detalhada e minunciosa o funcionamento de cada um dos componentes do módulo.
</p>

<h3>Funções Principais</h3>
<ul>
  <li><strong>Geração e Distribuição de Clocks:</strong>  
      Através da ferramente "IP catalog", disponível na IDE Quartus (utiliada para o desenvolvimento do projeto), foi criado um PLL para fornecer um clock estável aos blocos de memória, o que fornece uma valor preciso para os módulos principais.</li>
  <li><strong>Sincronização de Entradas:</strong>  
      As chaves sw são sincronizadas em registradores para evitar metastabilidade. Esses sinais determinam o modo de operação (replicação, decimação, zoom por vizinho mais próximo ou cópia direta).</li>
  <li><strong>Centralização e Endereçamento:</strong>  
      Calcula dinamicamente x_offset e y_offset, sinais do vga_driver, para centralizar a imagem na tela de 640×480, utilizada para desenvolvimento do projeto, esses sinais, geram o endereço do Framebuffer para cada pixel válido.</li>
  <li><strong>Controle da FSM:</strong>  
      A FSM do módulo de controle coordena a leitura de pixels da ROM, aciona o módulo de algoritmo selecionado e controla os sinais de escrita na RAM, permitindo com que o projeto funcione da melhor forma.</li>
</ul>

<h3>Fluxo Operacional</h3>
<p>
    Nessa seção, falaremos sobre toda a parte operacional do funcionamento da Unidade de Controle, que segue uma sequência bem definida de etapas. Inicialmente, assim que o sinal vga_reset é acionado, o sistema entra no estado de RESET. Nesse momento, são realizados os ajustes iniciais, incluindo a configuração dos fatores de escala de acordo com a opção escolhida pelo usuário por meio das chaves de entrada. Com a configuração concluída, o sistema passa para a fase de leitura sequencial da ROM. Aqui, o endereço rom_addr é continuamente incrementado, percorrendo toda a imagem original armazenada, que possui resolução de 160×120 pixels. Cada pixel lido é então encaminhado para o módulo de algoritmo responsável pelo redimensionamento.
</p>

<p>
    A etapa de redimensionamento é basicamente é o processamento do pixel pelo algoritmo selecionado, vale ressaltar que a escolha do modo de operação depende do código de controle gerado a partir das chaves. Assim, o pixel pode ser replicado, reduzido, interpolado ou simplesmente copiado de forma direta - Por exemplo, no modo de replicação ×2, cada pixel proveniente da ROM é expandido em quatro pixels consecutivos que serão gravados no framebuffer- Após o processamento, leitura e aplicação dos algoritmos, ocorre a escrita na RAM dual-port, que funciona como framebuffer. A posição de memória correta é calculada a partir do registrador de endereço addr_reg, levando em conta tanto a ampliação ou redução da imagem quanto os deslocamentos necessários para centralização. O sinal de controle ram_wren garante que a escrita ocorra apenas em ciclos válidos, evitando sobreposição ou perda de dados.
</p>

<p>
    A unidade de controle também controla o módulo vga_drive, que será explicado posteriormente. Esse fluxo coordenado garante que cada etapa — desde a leitura da ROM até a exibição final pelo VGA — seja sincronizada e controlada pela Unidade de Controle, assegurando o funcionamento estável do sistema.
</p>

<h3>Integração com os Demais Blocos</h3>
<p>
A Unidade de Controle conecta e organiza todos os módulos do sistema:
</p>
<ul>
  <li><strong>ROM:</strong> fornece pixels originais.</li>
  <li><strong>Módulos de Algoritmo:</strong> recebem dados e aplicam o redimensionamento.</li>
  <li><strong>RAM (Framebuffer):</strong> armazena a imagem processada e serve de interface com o VGA.</li>
  <li><strong>Driver VGA:</strong> exibe a imagem final centralizada na tela.</li>
</ul>

<h3>Exemplo de Operação</h3>
<p>
Suponha que o usuário selecione <code>sw = 4'b0000</code> (replicação ×2).  
Nesse caso:
</p>
<ul>
  <li>A Unidade de Controle configura <code>IMG_W_AMP = 320</code> e <code>IMG_H_AMP = 240</code>.</li>
  <li>Os offsets são <code>x_offset = 160</code>, <code>y_offset = 120</code>, garantindo centralização.</li>
  <li>Cada pixel da ROM é replicado em 4 posições consecutivas na RAM.</li>
  <li>O VGA exibe uma imagem de 320×240 pixels centralizada em 640×480.</li>
</ul>

<h3>Pontos de Atenção</h3>
<ul>
  <li>O reset <code>vga_reset</code> é ativo em nível baixo: a documentação deve enfatizar esse detalhe para evitar erros.</li>
  <li>Existe travessia de domínios de clock (<code>clk_vga</code> × <code>outclk_0</code>), devendo-se confirmar que a RAM e a ROM suportam operação dual-clock.</li>
  <li>É importante debouncing das chaves (<code>sw</code>) para garantir estabilidade nas trocas de modo em tempo de execução.</li>
</ul>

