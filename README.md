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

<h2 id="ula">Unidade Lógica e Algorítmica (ULA)</h2>
<p>
O módulo <code>ULA</code> é responsável por integrar e coordenar diferentes algoritmos de processamento de imagens
(replica&ccedil;&atilde;o, decima&ccedil;&atilde;o, zoom por vizinho mais pr&oacute;ximo, m&eacute;dia de blocos e c&oacute;pia direta),
atuando como um seletor inteligente que direciona os dados lidos da ROM para o subm&oacute;dulo adequado e garante a escrita correta no framebuffer.
</p>

<h3>Integração com os Demais Blocos</h3>
<p>
A <code>ULA</code> conecta diretamente a <strong>ROM</strong>, que fornece os pixels originais, com os diferentes
<strong>submódulos de algoritmo</strong>. Cada submódulo aplica um tipo específico de transformação:
</p>
<ul>
  <li><code>rep_pixel</code>: realiza replicação por fatores de 2× ou 4×.</li>
  <li><code>decimacao</code>: reduz a imagem por fatores de 2 ou 4, descartando pixels de forma controlada.</li>
  <li><code>zoom_nn</code>: aplica zoom baseado no vizinho mais próximo, ampliando sem interpolação complexa.</li>
  <li><code>media_blocos</code>: calcula a média em blocos, reduzindo a imagem com suavização.</li>
  <li><code>copia_direta</code>: transfere os pixels sem modificação.</li>
</ul>
<p>
O resultado de cada algoritmo é escrito na <strong>RAM dual-port</strong>, que funciona como framebuffer. Assim,
o <strong>driver VGA</strong> pode ler continuamente os dados processados para exibir a imagem final. A seleção do algoritmo ativo é feita pela entrada <code>seletor</code>, enquanto a FSM interna garante que apenas um módulo seja habilitado por vez.
</p>

<h3>Fluxo Operacional</h3>
<p>
O funcionamento da <code>ULA</code> segue uma sequência coordenada pela sua máquina de estados. Inicialmente, no estado
<code>RESET</code>, todos os submódulos recebem um sinal de reset ativo-baixo e as saídas de controle são zeradas.
A partir daí, a FSM verifica o valor da entrada <code>seletor</code> e direciona o fluxo para o estado correspondente ao algoritmo escolhido.
</p>
<p>
Cada estado ativa somente o submódulo relacionado, mantendo os demais em reset. Por exemplo, em
<code>ST_REPLICACAO</code>, o bloco <code>rep_pixel</code> é liberado, e seus sinais de endereço (<code>rom_addr</code> e <code>ram_wraddr</code>),
dados de saída (<code>ram_data</code>) e controle (<code>ram_wren</code>, <code>done</code>) são conectados diretamente às saídas da ULA.
Esse mesmo padrão se repete para decimação, zoom, média e cópia direta.
</p>
<p>
Enquanto um submódulo processa, o sinal <code>done</code> indica quando a operação foi concluída. Caso o usuário altere o seletor durante o processamento, a FSM força um retorno ao estado <code>RESET</code>, garantindo a integridade dos dados e reinicializando corretamente o fluxo.
</p>

<h3>Exemplo de Operação</h3>
<p>
Suponha que o usuário configure <code>seletor = 4'b0000</code>, escolhendo o modo de replicação ×2. Nesse caso,
a FSM libera o bloco <code>rep_pixel</code> com fator 2. A cada pixel lido da ROM, o submódulo gera quatro pixels de saída
em posições consecutivas da RAM. Esse processo continua até que todos os pixels sejam processados, momento em que
<code>done</code> é ativado.
</p>
<p>
Se o usuário alterasse o seletor para <code>4'b0001</code> (decimação ×2), a FSM retornaria ao estado de reset,
desabilitaria o módulo de replicação e ativaria o <code>decimacao</code>. A partir daí, apenas um a cada dois pixels seria
escrito no framebuffer, reduzindo a imagem de maneira controlada.
</p>
<p>
Esse comportamento padronizado, em que cada estado habilita exclusivamente o algoritmo correspondente, garante
robustez ao sistema, permitindo que múltiplos modos de redimensionamento coexistam em um mesmo projeto sem conflito de sinais.
</p>
