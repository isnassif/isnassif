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

<h2 id="control">Unidade de Controle (Control Unity)</h2>
<p>
A Unidade de Controle, implementada no módulo <code>control_unity</code>, é o elemento top-level responsável por
<strong>orquestrar o fluxo completo do sistema</strong>: desde a leitura sequencial da ROM, passando pela escolha e ativação
dos algoritmos de redimensionamento, até a escrita ordenada no Framebuffer e exibição pelo driver VGA.
</p>

<h3>Funções Principais</h3>
<ul>
  <li><strong>Geração e Distribuição de Clocks:</strong>  
      Utiliza um PLL (<code>pll_0002</code>) para fornecer um clock estável aos blocos de memória e divide o clock de 50&nbsp;MHz para gerar o clock VGA de 25&nbsp;MHz.</li>
  <li><strong>Sincronização de Entradas:</strong>  
      As chaves (<code>sw[3:0]</code>) são sincronizadas em registradores para evitar metastabilidade. Esses sinais determinam o modo de operação (replicação, decimação, zoom por vizinho mais próximo ou cópia direta).</li>
  <li><strong>Centralização e Endereçamento:</strong>  
      Calcula dinamicamente <code>x_offset</code> e <code>y_offset</code> para centralizar a imagem na tela de 640×480, e gera o endereço do Framebuffer para cada pixel válido.</li>
  <li><strong>Controle da FSM:</strong>  
      A FSM do <code>control_unity</code> coordena a leitura de pixels da ROM, aciona o módulo de algoritmo selecionado e controla os sinais de escrita na RAM.</li>
</ul>

<h3>Fluxo Operacional</h3>
<ol>
  <li><strong>Reset e Configuração Inicial</strong>  
      O sistema inicia no estado <code>RESET</code>, aguardando o sinal <code>vga_reset</code>. Nessa fase, são configurados os fatores de escala de acordo com a entrada do usuário.</li>
  <li><strong>Leitura Sequencial da ROM</strong>  
      O endereço <code>rom_addr</code> é incrementado, varrendo a imagem original de 160×120 pixels. Cada pixel lido é enviado ao módulo de algoritmo escolhido.</li>
  <li><strong>Processamento pelo Algoritmo</strong>  
      Dependendo do OpCode, o pixel é replicado, reduzido, interpolado ou copiado diretamente.  
      Exemplo: no modo replicação ×2, cada pixel da ROM gera 4 pixels consecutivos na RAM.</li>
  <li><strong>Escrita no Framebuffer</strong>  
      O resultado é armazenado na RAM Dual-Port na posição calculada por <code>addr_reg</code>. O sinal <code>ram_wren</code> habilita a escrita em ciclos válidos.</li>
  <li><strong>Exibição pelo VGA Driver</strong>  
      Em paralelo, o <code>vga_driver</code> lê continuamente a RAM pela porta de leitura, gerando os sinais VGA (hsync, vsync, RGB) em 25&nbsp;MHz.</li>
</ol>

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

