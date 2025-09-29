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
O módulo ULA é responsável por coordenar e aplicar os diferentes algoritmos de processamento de imagens disponíveis no projeto, ela atua como um seletor inteligente que direciona os dados lidos da ROM para o submódulo adequado (algoritmos) e garante a escrita correta no framebuffer. Abaixo, será explicado o seu funcionamento de forma detalhada:
</p>

<h3>Integração com os Demais Blocos</h3>
<p>
A ULA conecta diretamente a ROM, que fornece os pixels originais, com os diferentes algoritmo do sistema. Cada um deles aplica um tipo específico de transformação e escreve seu resultado na RAM, é possivel ver abaixo todas as funcções disponívels no projeto (não muito detalhadas, dado que cada um dos algoritmos já foi explicado anteriormente):
</p>
<ul>
  <li><code>rep_pixel</code>: algoritmo de realiza replicação por fatores de 2× ou 4×.</li>
  <li><code>decimacao</code>: reduz a imagem por fatores de 2x ou 4x, descartando pixels de forma controlada.</li>
  <li><code>zoom_nn</code>: aplica zoom baseado no vizinho mais próximo, ampliando sem interpolação complexa, podendo ser por 2x ou 4x</li>
  <li><code>media_blocos</code>: calcula a média em blocos, reduzindo a imagem com suavização em fatores de 2x ou 4x</li>
  <li><code>copia_direta</code>: transfere os pixels sem modificação.</li>
</ul>
<p>
Após aplicar toda a lógica, o resultado de cada algoritmo é escrito na RAM dual-port, que funciona como framebuffer. Assim, o driver VGA pode ler continuamente os dados processados para exibir a imagem final. A seleção do algoritmo ativo é feita pela entrada "seletor", que conversa com as chaves da placa, enquanto a FSM interna garante que apenas um módulo seja habilitado por vez.
</p>

<h3>Fluxo Operacional</h3>

<p>
O funcionamento da ULA é simples e direto, ela segue uma sequência coordenada pela sua máquina de estados. Inicialmente, configurada no estado RESET, onde todos os submódulos recebem um sinal de reset ativo-baixo e as saídas de controle são zeradas, garantindo que todas as vezes que um algoritmo específico for selecionado, os demais não interfiram na sua leitura. A partir daí, a FSM verifica o valor da entrada e direciona o fluxo para o estado correspondente ao algoritmo escolhido.
</p>

<p>
Cada estado ativa somente o submódulo relacionado, mantendo os demais em reset. Por exemplo, no estado ST_REPLICACAO, o bloco rep_pixel é ativado, e os seus sinais de endereço (rom_addr e ram_wraddr), dados de saída (ram_data) e controle (ram_wren, done) são conectados diretamente às saídas da ULA. Esse mesmo padrão se repete para decimação, zoom, média e cópia direta. Para cada algoritmo, o sinal done indica quando a operação foi concluída. Caso o usuário altere o seletor durante o processamento, a FSM força um retorno ao estado RESET, garantindo a integridade dos dados e reinicializando corretamente o fluxo, com todas as entradas novamente limpas.
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
