# The Elysium

Mesa online de **Vampire: The Eternal Struggle (VTES)** para jogar com **cartas físicas e câmeras**, até 5 jogadores. Cada Matusalém aponta a câmera para a própria área de jogo; a plataforma cuida do vídeo e do áudio, do pool, dos pontos de vitória, da Vantagem, dos turnos e fases, da eliminação automática e da **identificação das cartas** clicando nelas no vídeo.

**Site:** https://elysium-vtes.netlify.app · **Projeto irmão:** [Jack In](https://github.com/MarthosM/jack-in) (Netrunner)

## Recursos
- Vídeo e áudio direto entre os jogadores (WebRTC, via PeerJS), sem servidor próprio: o navegador de quem cria a mesa guarda o estado da partida.
- Pool, pontos de vitória, predador e presa, Vantagem, turnos, crônica e chat.
- Identificação de cartas por imagem (OpenCV.js) e por texto (Tesseract.js), comparando com o deck do jogador importado do VDB, do Amaranth ou de uma lista em texto.
- Link de convite que abre direto na opção de entrar; o anfitrião pode remover jogadores.

## Como rodar
É uma página única, sem etapa de build: abra `site/index.html` no navegador (Chrome, Edge ou Firefox) ou publique a pasta `site/` em qualquer hospedagem estática. No Netlify, o arquivo `netlify.toml` já aponta para `site/`.

Detalhes técnicos: [docs/ARQUITETURA.md](docs/ARQUITETURA.md).

## Contribuindo
Sugestões, relatos de erro e pull requests são bem-vindos. Abra uma issue descrevendo o problema ou a ideia. Todas as interações seguem o [Código de conduta](CODE_OF_CONDUCT.md).

## Licença
O código deste projeto está sob a [licença MIT](LICENSE). Partes de terceiros mantêm as próprias licenças; veja [THIRD-PARTY.md](THIRD-PARTY.md).

Projeto de fãs, gratuito e sem fins lucrativos. Vampire: The Eternal Struggle pertence aos seus detentores de direitos (Paradox Interactive / Black Chantry Productions). Este projeto não é afiliado a eles.

---

## English summary
**The Elysium** is a free, open-source web table for playing **Vampire: The Eternal Struggle** with **physical cards over webcams** (up to 5 players). It handles peer-to-peer video/audio (WebRTC via PeerJS), the game state (pool, victory points, the Edge, turns and phases, automatic ousting) and **card identification** by clicking a card in the video (OpenCV.js image matching + Tesseract.js OCR, compared against the player's own decklist). It is a single static page with no build step (`site/index.html`).

Code: MIT License. Third-party assets keep their own licenses (see THIRD-PARTY.md). Contributions are welcome under our [Code of Conduct](CODE_OF_CONDUCT.md). Non-commercial fan project, not affiliated with Paradox Interactive or Black Chantry Productions.
