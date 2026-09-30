# The Elysium — arquitetura (v2.6, set/2026)

Nome da plataforma: **The Elysium**. Página única, sem servidor próprio.

## Onde está o código e como atualizar
- **Código oficial:** `claude/site/index.html` neste projeto. É a versão que está publicada. Toda sessão nova deve partir deste arquivo (ler com project_read), editar, subir o `APP_VERSION` e salvar de volta aqui.
- **Hospedagem:** Netlify Drop (conta do Marthos). Para atualizar: gerar a pasta `the-elysium/index.html` (zip) → Netlify → site → **Deploys** → arrastar a pasta. O endereço continua o mesmo.
- **Versão visível:** constante `APP_VERSION` no código, mostrada no lobby ("Versão …"). Serve para confirmar que todos recarregaram a versão nova.
- Mudanças que alteram mensagens entre navegadores (como `snapReq`/`snapRes` e as ações) exigem que **todos recarreguem a página** antes de jogar.

## Rede e mesa
- **Rede:** PeerJS (WebRTC) com o servidor público de sinalização do PeerJS. O anfitrião usa o ID `the-elysium-vtes-<CÓDIGO>` e guarda o estado oficial do jogo; os convidados enviam ações e recebem o estado completo. Vídeo em malha completa; até 5 jogadores.
- **Link da mesa:** clicar no código da mesa copia `https://<site>/#CÓDIGO` quando publicado (ou só o código, se aberto como arquivo).
- **Modo convite (v2.4):** quem abre o site pelo link com `#CÓDIGO` vê só "Entrar na mesa", com o código já preenchido; o botão de criar mesa fica escondido. Para criar mesa, abrir o site sem o código no endereço.
- **Remover jogador (v2.4):** o anfitrião tem um ✕ na mesa de cada outro jogador (online ou desconectado). O jogador sai do estado, a tela dele some para todos, o turno passa adiante se era a vez dele e a Vantagem fica sem dono. O convidado removido recebe `{t:'kicked'}`, volta ao lobby e vê o aviso; pode tentar entrar de novo com o código.
- **Layout:** grade com todas as mesas (5 = 3 + 2). O modo foco mostra uma mesa grande e as outras em miniatura (botão ⤢, teclas 1–5, Esc volta). O botão ⛶ abre a tela cheia.
- **Orientação da câmera:** girar 90° / espelhar / inverter, aplicada **na origem** (canvas com relógio num Worker); todos recebem a imagem corrigida.
- **Estado:** pool, PV, eliminação automática (o predador recebe +1 PV e +6 de pool), Vantagem, turno/fases, crônica + chat, cronômetro, `deck` (booleano: o jogador tem deck carregado).

## Deck (aba "Deck")
- Importação: **link do VDB**. O link "padrão", com as cartas no próprio endereço (`#100001=3;…`), é lido localmente; o link `vdb.im/decks/<id>` passa pela KRCG `POST /vdb`, com a API do VDB como alternativa. **Amaranth** via KRCG `POST /amaranth`. **Texto colado** (VDB/TWD/Lackey/"3x Nome"), lido localmente com o banco de cartas; sem o banco, usa KRCG `/convert/json`.
- **Privacidade:** a lista fica só no navegador (localStorage). Os outros jogadores sabem apenas que existe um deck.

## Identificação de cartas
1. Clique → janela ao redor do ponto (tamanho da carta aprendido automaticamente) ou retângulo arrastado.
2. Quem clica pede a análise ao **navegador do dono da mesa** (`snapReq`/`snapRes`, repassadas pelo anfitrião), que tem a imagem original da câmera.
3. O dono roda **OpenCV.js** (`@techstark/opencv-js@4.10.0-release.1`, ~10 MB, carregado sob demanda ~1,5 s após entrar na mesa): contorno da carta (Canny + limiar adaptativo, filtro de proporção 63×88) → correção de perspectiva para 540×754.
4. Se o dono tem deck: **ORB + homografia RANSAC** contra as imagens das cartas do deck (KRCG, com o VDB como alternativa). Aceita o resultado se os pontos coincidentes (inliers) forem ≥ 20 e ≥ 2× o ruído, porque cartas diferentes coincidem um pouco pela moldura em comum. Devolve no máximo 2 cartas.
5. **(v2.5) A imagem decide, o texto só sugere.** Se o deck do dono não deu certeza:
   - Primeiro compara, só por imagem, com as **cartas já vistas na mesa daquele jogador** (e, na sua própria mesa, com o seu deck). A lista cresce quando uma carta é reconhecida pela imagem ou quando alguém escolhe uma candidata (memória da sessão, por nome do jogador, até 30 cartas).
   - Depois roda o OCR do nome (Tesseract.js) e confere as 8 melhores candidatas contra a **imagem oficial** de cada uma. Para vampiros, testa até 5 versões (grupos diferentes / avançado), porque as imagens mudam, e abre a versão que bateu. Aceita quando inliers ≥ 18 e ≥ 2× o 2º colocado; candidatas cuja imagem não bate (< 12) são rebaixadas. Passadas extras de OCR só acontecem se a imagem ainda não confirmou.
   - Imagens de referência: KRCG → VDB; se o servidor não liberar a leitura de pixels (CORS), passam pelo proxy público **wsrv.nl** (`vtes.pixelVia` guarda o caminho que funcionou). Assinaturas em memória (até 160 cartas). Sem imagens de referência, o resultado é só pelo texto, e isso aparece na tela.
6. Sem resposta do dono em 9 s → análise local sobre o vídeo recebido, sem o deck.

Testado com uma cena sintética (1920×1080, 4 cartas, uma travada, perspectiva): 10/10 corretas. Com deck: ~0,5 s. Só com texto: ~0,3–2 s. Mesa vazia → nenhuma candidata.

## Limitações conhecidas / próximos passos
- Não testado ainda com cartas reais nem entre duas máquinas. Leitura de pixels das imagens da KRCG depende de CORS: se bloqueada, o reconhecimento por imagem cai para o reconhecimento pelo nome.
- Sem TURN, algumas redes não conectam o vídeo (existe um campo TURN).
- O anfitrião sair encerra a mesa.
- Ideias: OCR em português; migração de anfitrião; varredura automática da mesa; publicação automática (GitHub + Netlify).

## Publicação pelo GitHub (a partir da v2.6)
- Repositório: https://github.com/MarthosM/the-elysium (licença MIT, código de conduta, README e THIRD-PARTY.md na raiz).
- O Netlify fica ligado ao repositório: cada commit na branch `main` gera um deploy de produção. O `netlify.toml` publica a pasta `site/`, sem etapa de build.
- Para economizar créditos do Netlify, juntar várias mudanças num commit só; testes podem ir para outra branch (prévias de deploy não gastam créditos).
- O rodapé do lobby traz os links exigidos pelo plano Open Source do Netlify: licença, código-fonte, código de conduta e "This site is powered by Netlify".
