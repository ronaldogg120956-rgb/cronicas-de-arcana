# 🧙 Guia para continuar o Crônicas de Arcana

> **Leia este arquivo antes de começar a mexer.** Ele contém tudo o que você precisa saber sobre o projeto.
> Comunicação com o dono do jogo deve ser em **português brasileiro**, de forma simples (ele é desenvolvedor leigo).

---

## 1. O que é o jogo

**Crônicas de Arcana** é um RPG 2D single-file em HTML5/Canvas.
- É **um arquivo só** (`index.html`) com HTML + CSS + JavaScript juntos.
- Tem também `sw.js` (service worker / cache), `manifest.webmanifest` (PWA) e a pasta `icons/`.
- Roda 100% no navegador. O progresso é salvo em `localStorage` (não tem backend/servidor).

## 2. Onde está publicado (link oficial do jogo)

👉 **https://ronaldogg120956-rgb.github.io/cronicas-de-arcana/**

A publicação é feita pelo **GitHub Pages** (grátis), a partir da branch `main`, pasta raiz `/`.
Toda vez que a `main` recebe um `push`, o GitHub Pages recompila e publica sozinho em ~40 segundos.

⚠️ **NÃO usar mais o Netlify** (`jazzy-llama-2190b3.netlify.app`): ele ficou sem créditos de build e parou de receber atualizações. Ficou no ar com versão velha, mas está abandonado.

## 3. Repositório

- GitHub: https://github.com/ronaldogg120956-rgb/cronicas-de-arcana
- Branch principal: `main`
- Repositório **público**

### Como clonar / enviar mudanças

```bash
git clone https://github.com/ronaldogg120956-rgb/cronicas-de-arcana.git
cd cronicas-de-arcana
```

Para fazer push, o dono usa um token pessoal do GitHub no remote:

```bash
git remote set-url origin https://ronaldogg120956-rgb:COLE_O_TOKEN_AQUI@github.com/ronaldogg120956-rgb/cronicas-de-arcana.git
```

(Por segurança, o token NÃO está escrito neste arquivo. Peça o token diretamente ao dono.)

Sempre commite com:

```bash
git config user.email "dev@arcana"
git config user.name "Arcana Dev"
git add -A
git commit -m "Descrição do que mudou"
git push origin main
```

Depois do push, **confirme que o site atualizou** esperando ~40s e checando no link publicado se o marcador novo apareceu (veja seção 7).

## 4. Como rodar localmente para testar

Na pasta do projeto:

```bash
python3 -m http.server 8000
```

Abre no navegador: `http://localhost:8000/`

> Se for testar com browser automatizado (Playwright), instale as dependências do Chromium:
> `python3 -m playwright install-deps chromium`

## 5. ⚠️ Regra de ouro: NÃO QUEBRAR O SAVE DO JOGADOR

O dono tem uma conta nível 26. Qualquer mudança que torne o save inválido apaga horas de jogo dele.

- Chave do save no localStorage: `arcana-save-v2:anon`
  - `SAVE_KEY = 'arcana-save-v2'`
  - `saveScope = 'anon'`
- O save é um JSON com checksum. Não mude o SALT nem o algoritmo de checksum sem fazer migração.
  - `SALT = 'arcana|integrity|9f4c7a1b'`
  - Checksum: função `saveChecksum` (DJB2 a/b em base36)
- O corpo é codificado com um base64 customizado (`B64C = 'A-Za-z0-9+/'`). **Não é base64 URL-safe.**
- A função `loadGame()` **NÃO deve apagar o save** quando der erro. Ela tenta recuperar/sanitizar ao invés de `removeItem`. Não volte com o comportamento antigo.
- `sanitizeSave()` remove ids quebrados do inventário e equipamentos apontando para itens inexistentes. Mantenha essa blindagem.

## 6. Estrutura rápida do código (dentro do index.html)

Como tudo está em um arquivo, aqui vão os pontos principais (números de linha podem mudar — procure pelo texto):

| Parte do jogo | Procurar por / função |
|---|---|
| Save / carregar | `SAVE_KEY`, `loadGame`, `saveChecksum`, `sanitizeSave` |
| HUD (minimapa, hotbar, quest) | `drawHUD`, `hudConfig`, `drawObjective` |
| Toggle do minimapa (tecla M) | `toggleMinimap`, `syncMinimapTab`, elemento `#minimapTab` |
| Chat do mundo | `refreshWorldChat`, `pruneOldChat`, `.chat-dock-log` |
| Forja de Feitiços (nível 15+, 26 mana, tecla Z) | `useCraftedSpell`, `useBeyondPower`, botão `data-cast-spell` |
| Além da Forma / essência | `beyondState`, `onBeyondEnemyKilled` |
| Arma viva (ex.: Voracius) | `adoptLivingWeapon`, `feedLivingWeapon` |
| Missões | `recommendedLevelForQuest`, `activeQuestEntry` |
| Interior de prédios (Forja, Banco, Mercado) | `drawBuildingInterior`, `enterBuilding`, `leaveBuilding`, `buildingNPC` |
| Inimigos / mobs do mundo | array de inimigos (IDs 70–87 foram os adicionados) |
| Teclas padrão | `defaultBindings` (beyond=Z, beyond=X, missions=M, skills=K, inventory=I...) |
| Service worker / cache | registro `serviceWorker.register('sw.js')` e o `sw.js` |

## 7. Cache / versão do service worker

- O cache do jogo vive no `sw.js`. Quando mexer em algo importante, **suba a versão** do cache:
  - Atual: `CACHE = 'arcana-v57' (v57: Mundo Online & Co-op Realtime via Supabase Channels, sincronização de jogadores remotos, balões de fala flutuantes, sincronização de dano em monstros, e gerador de link de convite ?sala=...)` (v52: velocidade base do herói +12%, novo atributo Velocidade no menu de Status (+4 de movimento por ponto, medido na tela) e Guardião redesenhado: escudo-torre astral bem maior + peitoral até o cinto com runa + ombreiras maiores)
  - Próxima: `'arcana-v56'` (a versão subiu a cada release; confira sempre o valor real no topo do `sw.js`)
- O `netlify.toml` (mesmo não sendo mais usado) tem headers `no-cache` para o HTML.
- O `sw.js` usa estratégia **network-first para HTML** (pega a versão nova do servidor quando tem internet).

Para confirmar que uma publicação entrou no ar, você pode checar por um marcador novo no HTML público, por exemplo:
```bash
curl -s https://ronaldogg120956-rgb.github.io/cronicas-de-arcana/ | grep -o "NOME_DO_MARCADOR"
```

## 8. O que JÁ FOI FEITO (não precisa refazer)

- ✅ Bug do minimapa que sumia corrigido (regra CSS `.minimap-host{display:none}` removida).
- ✅ Toggle do minimapa: aba `#minimapTab` fixa no canto + tecla M.
- ✅ Hotbar sempre visível (`autoHideHotbar = false`) e `drawHUD` blindado com try/catch por painel.
- ✅ Chat refeito: quebra de linha, scroll, nome com largura limitada; atualiza a cada 5 min; mensagens somem após 10 min.
- ✅ Mais mobs no mundo (+18 inimigos, IDs 70–87) e respawn mais rápido (~45%).
- ✅ Nível recomendado por missão (`recommendedLevelForQuest`).
- ✅ Cursor `crosshair` no canvas; clique direito não abre menu; bug de clique aninhado corrigido.
- ✅ Sprites das habilidades de monstro na aba skills (ícones coloridos no canvas mini).
- ✅ Interiores dos prédios decorados (Forja com bigorna, Banco com ouro, Mercado com caixotes) e NPC movido pro balcão.
- ✅ Forja de Feitiços: botão "✨ Lançar feitiço (26 ✦)" (`data-cast-spell`) que chama `useBeyondPower`; tecla Z também funciona.
- ✅ Essência do Além cai também de elites (+2) e chefes mundiais (+15), não só de inimigos beyond/chefes lendários.
- ✅ Arma viva: trava de `bossKills < 1` removida; "Oferecer essência" funciona mesmo com pouca essência; card mostra a essência atual.
- ✅ Save robusto: `loadGame` não apaga mais; sanitiza inventário/equipamentos; garante `nextItemId` válido.
- ✅ Publicação transferida do Netlify para o GitHub Pages.
- ✅ Service worker `arcana-v2` com network-first.
- ✅ **Sistema de Zonas Seguras embutido no `index.html`** (antes ficava só nos arquivos `safe-zones.js`/`guardas-melhorados.js`, que NÃO são carregados — o `index.html` tem um único `<script>` inline). Agora as funções `isInSafeZone`/`updateSafeZones`/`drawSafeZones` e o array `safeZones` vivem dentro do IIFE, logo depois dos guardas. Monstros que entram em vila/cidade morrem (sem XP) e agendam respawn (5–9s), sem loop de flicker.
- ✅ **Erro de sintaxe fatal corrigido** (um `;` solto dentro do array `drawables` do `drawWorld` impedia o jogo de abrir — sintaxe quebrada matava o `script` inteiro).
- ✅ **Trava de mira por toque (mobile) consertada**: `mobileAimTarget()` agora respeita `mobileLockManual` enquanto o alvo existe e está a ≤620px; antes a trava era zerada a cada frame.
- ✅ Testes Node (`tests/runtime-expansion.js`) atualizados para o estado atual do jogo (guilda em tiers `g{t}_{kind}`, XP/cristais só para inimigos com `engagedByPlayer=true`, side-quests ≥3, givers viram arrays, `setReincTest` exposto, chat com `left:50%`/`max-height:150px`) + novo teste dedicado às zonas seguras. Suite inteira passando.
- ✅ **NPCs com pixel art**: `drawLivingNPC` agora desenha chibi pixelado 12×16 (escala 2) via `drawPixelChibi` + tabela `NPC_SPRITES` (15 roles: mentor, aurora, blacksmith, merchant, lia, banker, captain, ulric, worldGuide, eventMaster, worldMerchant, stormSage, storyGuide, orso, arena). Cada role tem chapéu (`hat`), penteado (`hair`), acessório na mão (`prop`) e emblema no peito (`emblem`). `drawNPCFaceSimple` continua no código mas não é mais usado pelos NPCs de cidade (a Ceifadora mantém seu desenho próprio). Para novo NPC: adicione o role em `NPC_SPRITES` e dê o `role` no objeto do NPC (ex.: `arenaMaster` ganhou `role:'arena'`).
- ✅ **Muralhas de pedra com torres (cidades) e cercas altas de madeira (vilas)** (v39): arrays `wallDefs` (pedra) e `fenceDefs` (madeira) logo após `safeZones`, com segmentos `[x,y,w,h,lado]`, torres de canto (`towers`) e portões (`gates` com `gl/gr` = bordas do vão, `sy` = y de desenho). `wallSegRects`/`wallTowers` entram no `rectObstacles()` (colisão REAL — jogador e mobs não atravessam). Desenho em pixel: `drawMedievalWall` (pedra com tijolos, merlões na borda externa e sombra interna; cerca com estacas pontudas + travessas), `drawWallTower` (torre de pedra com merlões nos 4 lados + bandeira da cidade) e `drawWallGate` (soleira + pilares + portas de madeira abertas para dentro + brilho dourado). Tudo entra no `drawables` com y-sort (segmento desenha na y da borda de baixo; portão na `sy`; guarda de muro no `sortY`). Cidades: Lumina (portões W/E na estrada), Auralis (portão N na estrada), Elyndra (W/E + S), Nimbora (W/E). Vilas: Casa de Lyra + praca (cerca W/E) e Ilha do Alvorecer (cerca com portão W na praia e E no cais do barco). `unstickPlayerFromWalls()` roda no `loadGame`/respawn: se o save colocar o jogador DENTRO de um muro, ele vai pro portão mais próximo (protege o save do jogador). NPCs que ficariam fora dos muros mudaram de lugar (Selene→(3280,1600), Talia→(4180,1640), Zephyra→(5480,1650), Seris→(4710,2155), Ulric→(1450,4320), com `homeX/homeY`), o portal de Nimbora foi pro portão W (5195,1845) e os objetivos que apontavam para Talia/Seris agora usam `npcObjPos(role,fx,fy)` (posição viva do NPC). A borda tracejada antiga das zonas (`drawSafeZones`) some onde existe muro real (`walledZoneIds`).
- ✅ **Guardas reorganizados: 2 de espada e escudo + 1 arqueiro em cima do muro, por cidade** (v39): `setupGuards` usa `guardPosts` (2 guardas por cidade flanciando o portão principal) e `wallArchers` (1 arqueiro no topo do muro, com `sortY` para desenhar por cima do muro). Guardas de chão: espada + **escudo** com a cor da cidade (forçado em `drawPixelGuard`, `wpnV=0` + escudo no off-hand). Arqueiros: `kind:'archer'` (`wpnV=9`) — arco, corda, flecha no ataque e aljavela nas costas; atacam em até ~340px com projétil `kind:'arrow'` (desenho próprio em `drawProjectile`) e alvo via `nearestArchTarget` (até 480px, independente do raio da cidade).
- ✅ **Guardas com pixel art (todos diferentes)**: `drawPixelGuard(g,x,y,t,flip,walking,swing)` desenha o guarda em grid 12×16 (escala 2). A variante vem do `id` do guarda: `seed=(gd.id*7+cityLen*13)%16` → elmo (4 variantes: elmo+pluma / great helm / chifres / bulwark), arma (4 variantes: espada / lança / besta / martelo) e pele (3 tons). A tabard usa a cor da cidade (`cityDefs`). 13 guardas no total (3 Lumina, 4 Auralis, 3 Elyndra, 3 Nimbora). ⚠️ A chamada no `drawGuards` precisa ser `drawPixelGuard(g,x,y,t,flip,walking,swing)` (guarda em 1º, SEM passar o `ctx`) e o `P()` interno desenha com o `ctx` do módulo — trocar isso quebra o sprite (bug real já consertado: a chamada antiga passava `ctx` em 1º e tudo desalinhava, deixando TODOS os guardas idênticos).
- ✅ **Herói com pixel art (21 classes)**: `drawPixelHero(ctx,x,y,o)` + tabela `HERO_CLASS_PX` (uma entrada por classe: `hat` e `emblem`). O corpo do jogador no `drawPlayer` agora é pixel (12×16, escala 2) com cores do `appearance` (roupa/pele/cabelo/olhos), chapéu/emblema da classe e ombreiras para kit `warrior`. `drawCharacterOutfit`/`drawCharacterFace` continuam definidos (usados na prévia do criador) mas não desenharam mais o corpo do jogador.
- ✅ **Motor de poses completo (20×16, escala 2)**: `drawPixelHero` reescrito com corpo completo em pixel (braços incluídos) e poses: `idle0/1` (respiração), `walk0-3`, `run0-3`, `atk0-3`, `cast0/1`, `gather`, `fish`, `fishBite` × facings `front/back/side` (flip espelha). Armas em pixel via `heroWeaponPx` (sword/axe/dagger/bow/gun/staff, com swing por pose e arma pendurada nas costas no facing `back`) e socos via `heroFistsPx`. No `drawPlayer`, a pose vem de `heroPose` (prioridade: pesca > coleta > conjurar > ataque > dash > andar > parado) e a direção de `heroFacing`/`flip` segue o cursor (strings exatas mantidas para a suite de testes). Vara de pescar na col 16 (NUNCA sobre o rosto) com linha + boia; coleta com brilho da cor do recurso; cast com orbe + partículas.
- ✅ **Invocação só das classes que têm no jogo (3 espíritos animados)**: `SUMMON_CLASS_IDS = ['ranger','necromancer','summoner']` (as mesmas que têm a habilidade de companheiro em `commandCompanion`: lobo do Caçador, esqueletos do Necromante, draco do Invocador). `drawClassSummon(g,px,py,clsId,t,active,flip)` desenha o espírito pixelado flutuante ao lado do herói (bob senoidal, variantes wisp/wing/hood via `HERO_SUMMONS`, partícula por classe) com círculo de invocação pulsante (8 dots rotativos + elipse); fica mais forte quando o herói conjura/ataca. As demais 18 classes NÃO têm espírito (pedido do dono).
- ✅ **FX de ataque genérico**: `drawPlayer` desenha o arco da arma (sword/axe/dagger/staff) com a cor de encantamento ou da classe; gun/bow/fists sem arco.
- ✅ **Debug de pose p/ screenshots**: `__ARCANA__.setHeroPose({pose,facing})` sobrescreve a pose do herói (usado para as fotos das mesas de poses; não afeta o jogo normal).

- ✅ **Herói pixel redesenhado: magro, pernas longas e arma VISÍVEL com animação** (v40): `drawPixelHero` agora usa `const s=o.s||2` e x0 centralizado (`x-10*s`); tronco magro de 8 colunas (6–13) com cinturão/fivela, cabeça estreita (10 colunas, 5–14) com variantes de `appearance.eyeStyle` (closed/wide/sharp) e cabelo nas bordas, braços de 2px na col 14 (lado) e pernas de 4 linhas (rows 12–15) com poses de caminhar/correr mais longas. `heroWeaponPx` reescrita com assinatura `(g,P,pose,facing,wpn,wc,acc,t)` — arma maior em pixel (lâmina de espada 2px, guarda de 4px, brilho) com balanço idle de 2 frames (`Math.floor(t*2.4)%2`) e poses de ataque reescaladas: atk0 lâmina vertical acima, atk1 horizontal na frente (5×2 com ponta clara), atk2 baixo; costas com arma 2×6 cruzada. `heroFistsPx` com socos de 3×3. As 2 chamadas dentro de `drawPixelHero` passam `t`. ⚠️ `drawWeaponOnBack`/`drawFistPunch` (linhas de morte/reencarnação) NÃO mexer — strings exigidas pelos testes.
- ✅ **"Crie seu Herói" em pixel + armaduras em pixel de verdade** (v40): `renderCharacterPreview(t)` reescrita — desenha o herói pixelado grande (`drawPixelHero(g,90,136,{s:5,...})` no canvas `#characterPreview` 212px, 150px no mobile) com cores de `appearance`, chapéu/emblema da classe, saia se `gender==='female'` e animação (`idle` alternando + golpe `atk0-3` de 0.7s a cada 4.6s que mostra a arma). Hook no `draw()`: `if(!ui.creator.classList.contains('hidden'))renderCharacterPreview(state.time)`. Armaduras viram pixel de verdade no herói: bloco `if(gArm)` desenha peitoral (8×3 na cor da raridade) + borda inferior + brilho central + pauldrons de armadura; o corpo só fica levemente tingido (`mixColor(base,gArm,.35)`). O boneco do inventário virou `<canvas id="heroDollCv">` 84×84 pixelado: `renderInventory` desenha o herói com o equipamento real equipado (arma/armadura/elmo/luvas/botas com cor de raridade+encanto) via bloco antes do loop de `[data-equip]`.

- ✅ **Guerreiros e tanks com escudo + armadura padrão** (v41): `drawPixelHero` ganhou `o.shield` (desenhado via `drawShield(dy)` antes da arma, em 3 posições: lado cols 11–14 rows 7–12 / frente cols 7–10 rows 7–13 / costas cols 4–7, na cor `o.shieldColor||o.accent` com borda escura, brilho na frente e boss claro no centro) e `o.defArmor` (armadura padrão de aço `#8a94a6` quando não há armadura equipada — `gArm=o.gearArmor||(o.defArmor?'#8a94a6':null)`, reaproveitando o bloco de peitoral+pauldrons). Todas as 5 classes do grupo Guerreiros e Tanques (`c.kit==='warrior'`: guerreiro, cavaleiro, bárbaro, paladino, guardião) recebem `defArmor:true,shield:true` nos 4 pontos de chamada: `drawPlayer`, prévia do criador, retratos das classes no menu e boneco do inventário. Escudo fica ABAIXO da arma no z-order (a espada desenha por cima).

- ✅ **Escudo de frente em formato de escudo de verdade** (v43): o `drawShield` do facing `front` saiu do retângulo 4×7 e virou escudo kite/heater de 5 colunas (6–10): borda `scD` nas rows 7–11, afunilamento row 12 (3 cols) + ponta row 13, face interna `sc` (cols 7–9 rows 8–11), filete superior claro `scL` e **cruz central no boss** (`P(8,8,1,4,boss)` vertical + `P(7,9,3,1,boss)` horizontal). O facing `back` ganhou base apontada (row 12 com 2 cols). Lado não mudou (o usuário aprovou).

- ✅ **Muro que sumia nos cantos corrigido** (v44): o culling dos segmentos de muro em `draw()` usava `worldPointNearViewport(o.x+o.w/2,o.y+o.h/2,280)` (só testava o PONTO CENTRAL do segmento) — nos cantoneiros (ex.: NE de Lumina, player em ~(1665,766)) o centro do segmento ficava fora da viewport+margem e o muro inteiro sumia enquanto a torre (culling próprio) continuava visível. Agora é overlap de retângulo: `o.x<cam.x+W+280&&o.x+o.w>cam.x-280&&o.y<cam.y+H+280&&o.y+o.h>cam.y-280`. Torres/portões seguem o culling antigo (já ok).
- ✅ **Esqueleto da Forma da Morte em pixel** (v44): `drawDeathFormPlayer` (vetorial, ~315 linhas) trocado pelo motor `drawSecretFormPx` com `species:'skeleton'` — crânio por direção (frente com olhos brilhando na cor do estágio + nariz + dentes / costas = "nuca: coluna cervical" / lado profile), coluna+costelas+pélvis, capa esfarrapada por caminho, pernas com juntas (mesmas coords do herói), arma via `heroWeaponPx`, escudo kite para kit warrior (erguido no defend, olhos ficam por cima). Variantes mantidas em pixel: Carniçal (carne nos braços/coxas/peito, estágio 1+ sem caminho vida), Cavaleiro da Morte (peitoral de placas + cinto, death estágio 2+), Vampiro (casaco escuro, life estágio 2), coroa dourada estágio 3 + **Rei de Ossos** (death estágio 3: coroa + chifres enormes), 3º olho estágio 2+, asas estágio 3 (osso/douradas), **cicatrizes: uma nova marca a cada 100 mortes**, armadura/elmo/luvas/botas equipados em pixel (elmo não tapa os olhos), aura radial do antigo mantida, marca da colheita (🧟) e espiral de troca de pele mantidos.
- ✅ **Classes secretas (reencarnações) em pixel com poses completas** (v44): novo motor `drawSecretFormPx(g,x,y,o)` (grade 20×16 idêntica ao herói, reutiliza `heroWeaponPx`) com espécies `skeleton`/`slime`/`spider`/`goblin` e poses `idle0/1`, `walk0-3`, `run0-3`, `atk0-3`, `cast0/1`, **`defend`** (novo: guerreiro ergue o escudo; slime endurece a borda; aranha agacha; goblin ergue o buckler) e `gather` × facings front/back/side (flip). Slime: blob com squish de 2 frames, boca por estado (ataque/abrir/consumir com língua), equipamento flutuando dentro (arma mini alpha .75 + armadura), gota de essência no cast. Aranha: abdômen+listra, cabeça com 6 olhos vermelhos (brancos no cast), 8 pernas de 2px animadas em 2 frames com garras, presas abrem no ataque, veneno/web/enxame/teia por habilidade. Goblin: orelhas pontudas, olhos amarelos, túnica com tira diagonal, pernas curtas com pés garra, buckler (frente/lado/costas, erguido no defend), rocha/orbe por cast. `drawReincFormPlayer`/`drawDeathFormPlayer` derivam pose/facing/flip com a MESMA prioridade do herói (coleta > cast > ataque > dash > **defend** (guardActive) > andar > parado) + `heroPoseOverride`. `drawSlimeForm`/`drawSpiderForm`/`drawGoblinForm` viraram wrappers finos (nomes mantidos p/ os testes) e os retratos dos popups (`drawReincPortrait` → `drawReincFormPlayer`) ganharam pixel art de brinde. ⚠️ Strings exigidas pela suite: `pose=Math.abs(faceX)>.45?'side':faceY<-.35?'back':'front'` (nos 2 dispatchers), `nuca: coluna cervical`, `animações de cast do esqueleto`, `investida de gosma`, `presas avançando`, `arco da clava`, `golpe de osso com trilha`, `boca gigante engolindo`, `picaretada com lascas`, `Carniçal: carne volta`, `Cavaleiro da Morte: peitoral`, `Rei de Ossos: crânio com coroa`.

- ✅ **Escudos em pixel maiores** (v45): o `drawShield` do `drawPixelHero` (usado por guerreiros, cavaleiros, bárbaros, paladinos e guardião no herói, criador, retratos e boneco do inventário) foi ampliado mantendo o formato kite aprovado: frente cols 5–11 rows 7–14 (era 6–10 rows 7–13) com afunilamento em 3 etapas e cruz no centro; lado cols 10–14 rows 7–13 (era 11–14 rows 7–12) com filete lateral; costas cols 3–7 rows 7–13 (era 4–7 rows 7–12). O esqueleto da Forma da Morte (`shieldPx` do `drawSecretFormPx`) usa as MESMAS coordenadas maiores (no `defend` o escudo ergue 2 rows e os olhos ficam por cima). O buckler do goblin cresceu de 4×4 para 5×5 (frente/lado/costas, erguido no defend). O escudo do guarda em pixel (`wpnV===0` do `drawPixelGuard`) foi de 4×6 para 5×8 com emblema maior. O escudo vetorial de guarda ativa (`drawPlayerShield`, efeito de bloqueio) NÃO mudou.

- ✅ **Armadura sem "bigode" + Servo Esqueleto em pixel** (v46): a barra escura embaixo do queixo vinha do TOPO DO ESCUDO frente/lado na row 7 (7 cols logo abaixo do rosto) + barra cheia da armadura na row 8. Fix: `drawShield` do `drawPixelHero` — frente/lado desceram 1 row (topo row 7→8: frente `P(5,8+dy,7,1,scD);P(6,9+dy,7,4,scD);P(7,13+dy,5,1,scD);P(8,14+dy,3,1,scD);P(9,15+dy,1,1,scD)`…; lado `P(10,8+dy,5,5,scD);P(11,13+dy,3,1,scD);P(12,14+dy,1,1,scD)`…), costas seguem na row 7 (sem rosto ali). A armadura do torso (`gArm`) ganhou decote em V por facing — row 7 do centro fica túnica (frente: `P(4,7,2,2,plate);P(14,7,2,2,plateD);P(6,8,2,3,plate);P(12,8,2,3,plate);P(8,9,4,2,plate);P(9,9,2,2,plateL);P(6,10,8,1,plateD)` + filete dourado `P(9,11,2,1,mixColor(gArm,'#ffd36b',.45))`). O esqueleto da Morte (`o.gearArmor` no `drawSecretFormPx`) usa o mesmo V por facing; o crânio termina na row 5, então ele MANTÉM o escudo na row 7. Servo Esqueleto do necromante (`c.type==='skeleton'` no `drawCompanionBody`, ~6034) saiu do vetorial (costelas só de um lado) para pixel: crânio 10×7 (rows -14..-7, olhos 2×2 verde + glint, nariz `Q(-1,-10,2,1,DK)`, dentes com gaps, queixo BL+DK), costelas dos DOIS lados rows -5..-2 (`Q(-4,y,3,1,B);Q(2,y,3,1,B);Q(-1,y,2,1,BL)`), pélvis, pernas 2 frames de caminhada, espada curta erguida em repouso e estendida no atk (lâmina desce + arco verde `ctx.arc(2,-2,11,-.9,1.1)` alpha `atk*.85`), special = olho claro + gema no peito. Cores: B `#e9e5d2`, BL `#cfc9b0`, DK `#413c2d`, GR `#9ee87d` (special `#c8ff9e`).
- ✅ **Guia de missão consertada (seta da linha principal)** (v47): 5 fixes no fluxo pós-boss. (1) `questEntries` — a etapa do portal agora é por POSIÇÃO: `state.quest>=5&&state.sideQuest===0` → se `player.x>2850` a entry 'story-first' vira "🌀 Bem-vindo a Auralis / Fale com a Capitã Selene" com obj `{x:cityNPCs[0].x,y:cityNPCs[0].y,label:'Capitã Selene'}`; senão "Portal para Auralis" (`travelPortals[0]` 920,1110). Acabou o ping-pong "use o portal e volte pro portal". (2) `activeQuestEntry` — trocou `entries.find(e=>e.story)` por ordem explícita `mainIds=['tutorial','beyond-dream','beyond-hunter','beyond-war','story-world','side-crypt','story-heart','chest','story-first','secret','event','story-final','story-done']` (a linha principal nunca perde a seta para `zone:*` rastreada, contrato, side do livro ou home; a ordem world/crypt/heart/chest/first passa nos testes do diário: story-world > side-crypt > story-heart, e story-world/story-heart > chest). O baú "Tesouro antigo" ganhou id próprio `'chest'` (`recommendedLevelForQuest` retorna 5) — não conflita mais com o id 'story-first'. O gate de nível (sugerir secundária compatível quando `player.level<rec-1`) agora filtra por `!mainIds.includes(e.id)`. (3) Talia (`worldGuide` em `handleTownNPC`) ganhou gates: q0 exige nível 8 (Titã nv 22), q2 exige nível 14 (Hidra nv 26), q4 exige nível 22 (Dragão nv 30) — mensagem amigável com o nível atual do player e sem iniciar a expedição. (4) Seris (`talkStoryGuide`) — Coração Partido exige nível 10 p/ iniciar (alvos variam nv 5–14 por aliança); abaixo disso recusa com a mesma mensagem de caça. (5) `handleTownNPC` — a auto-aceite do livro (`offerSideQuest`) NÃO intercepta mais a conversa da linha principal: `const mainObj=getObjective()`; se `mainObj.label===town.name||Math.hypot(mainObj.x-town.x,mainObj.y-town.y)<140` a conversa segue para a história (ex.: Selene inicia a cripta, Talia inicia/avança as expedições) e o livro fica para a próxima conversa. Sem campos novos de estado → save antigo 100% compatível. Marcadores p/ curl: `Bem-vindo a Auralis` e `Missão do livro não intercepta a linha principal`.

- ✅ **Armadura refeita (placas de verdade, sem "avental" nem anéis flutuantes)** (v48): dois problemas no visual das armaduras. (1) `drawPlayer` (~5774) desenhava DOIS círculos `ctx.arc(x±13,bodyY+1,5)` na cor da melhor gear quando `gearRank[rarity]>=2` (épico+ `#c48fff`, lendário `#ffd873`) — anéis roxos flutuando do lado da cabeça como "furo de orelha". Linha removida (comentário `// v48: anéis flutuantes...` no lugar dela); o anel elipse NO CHÃO abaixo do herói (rank>=1) foi mantido. (2) `drawPixelHero`/`torso()` — o bloco `gArm` era uma placa lisa 8×3 sem contorno (parecia avental). Substituído por placas segmentadas por facing: FRENTE = ombreiras 3×3 saindo do corpo (`P(3,7,3,1,plateL);P(3,8,3,1,plate);P(3,9,3,1,plateD)` e espelho em 14), gola ao lado do pescoço (`P(8,7,1,1,plateL);P(11,7,1,1,plate)`), peitoral omoplata→esterno→abdômen (`P(6,8,2,1,plate);P(12,8,2,1,plate);P(6,9,8,1,plate)` + esterno claro `P(9,9,2,1,plateL)` + filetes laterais + brilho `P(7,9,1,1,'#ffffff88')` + **brasão sobre base escura** `P(8,9,3,3,shadeColor(gArm,-.5))` para o emblema da classe) — o filete dourado `P(9,11,2,1,mixColor(gArm,'#ffd36b',.45))` no cinto foi mantido. COSTAS = ombreiras 3×3, placa 8×3 com filete claro no topo, costura central `P(9,9,2,2,shadeColor(gArm,-.14))`, base escura p/ emblema. LADO = ombro à frente 2×2 (`P(13,7,2,2,plate)` + topo `plateL`), ombro traseiro 2×2 `plateD`, peitoral ESTREITO cols 7–12 rows 8–10 (`plate/plate/plateD` + filete + glint) em vez da barra larga 6–13 que cobria o corpo inteiro. Marcadores p/ curl: `Peitoral em placas` (comentário novo no torso) e `anéis flutuantes de gear épica+ removidos` (comentário no drawPlayer). Sem campos novos de estado → save antigo 100% compatível. ⚠️ A FRENTE das classes kit warrior continua dominada pelo escudo kite grande (v43/v45, aprovado pelo usuário) — o peitoral novo aparece de lado/costas e nas classes sem escudo (mago, necromante etc.).

- ✅ **Sapos fora da vila + seta da guerra + elmos de verdade + soldado de guerra** (v49): 4 fixes. (1) **Spawn de sapo dentro da cidade**: 3 `makeEnemy('shade',...)` nasciam DENTRO da zona segura do Vale de Lumina — `1130,1080` → `1520,900`, `1290,930` → `1690,1150`, e `980,1250` → `1470,1330` (floresta, a leste da vila/portal). Como `updateSafeZones` mata inimigo que entra na zona, eles morriam e renasciam em loop "na vila" (a ~150-210px do portal de Auralis). Blindagem: na morte por zona segura, se `isInSafeZone(e.homeX,e.homeY)` o ponto de respawn é deslocado (offsets `-300/300/0,-300/0,300/±480,±180/0,±480` — primeiro que sair da zona) e só vira `noRespawn` se nenhum offset funcionar. (2) **Guerra entre facções roubava a seta**: na ordem antiga de `mainIds` o `beyond-war` (eventos "além": sonho/caçadores/guerra) vinha ANTES de `chest`/`story-first` — com a guerra ativa, a seta mandava o jogador nv 6-8 para os combatentes de nv 15 em Campos Boreais (5360,2960) no meio do fluxo baú→portal. Nova ordem: `['tutorial','story-world','side-crypt','story-heart','chest','story-first','beyond-dream','beyond-hunter','beyond-war','secret','event','story-final','story-done']` — a linha principal (baú/portal/Selene/campanha da Talia) nunca perde a seta; a guerra continua no diário (tecla M) para quem quiser rastrear. Testes: `questNow()` exposto em `__ARCANA__` (getter de leitura da `activeQuestEntry`) para a suíte Playwright. (3) **Elmos equipados**: o bloco `if(gHelm)` de `drawPixelHero` desenhava o MESMO disco liso 14×6 para "Capacete de Couro", "Elmo do Vigia" e "Coroa Arcana". Novo sistema: `helmetKindOf(it)` (regex no nome: `/Coroa/i`→crown, `/Elmo/i`→helm, senão cap) + `gearHelmetPx(P,dy,o,gHelm,gHelmRank)` (função de topo, usada pelo herói, pela Forma da Morte e pelo doll do inventário) com 3 formas por facing (front/side/back): **cap** = boquete arredondado + aba na testa + abas laterais na nuca; **helm** = cúpula alta + crista (`trim` no topo) + barrota nasal cols 9-10 rows 2-5 + guardas de bochecha + filete na testa (front) / fresta escura (back); **crown** = 3 pontas (cols 5-6, 9-10 altas, 13-14) + faixa + filete dourado + gema da cor da classe na frente. Cor segue a raridade/encanto como antes; common fica couro marrom; brilho extra em épico+. `gearHelmetKind` passou a ser enviado nos 3 call sites (`drawPlayer`, `drawSecretFormPx`, doll `#heroDollCv`). ⚠️ O doll do inventário re-equipa o "melhor" item ao abrir (equipmentAutoMode) — no teste via API, desligar o auto (`#inventoryList [data-auto-mode]`) antes de `setEquip`. (4) **Combatentes de guerra pareciam sapo**: o `drawBeyondBody` usava o blob genérico (cabeça redonda + corpo + bastão) para `kind==='war'` — os Combatentes de facção (cores: Lumina `#ffd56b`, Auralis `#79b8df`, Horizontes `#72e0c7`, Umbral `#c27ad9`) viraram soldado em vetor: pernas com passo, torso com ombreira clara, cabeça com elmo (arco superior + viseira escura + brilho de olho), espada diagonal com guarda dourada e escudo kite no braço oposto. O `heroHunter` mantém o blob com bastão. Marcadores p/ curl: `tipos de elmo equipados`, `soldado de verdade`, `ponto de respawn não pode ficar dentro`. Sem campos novos de estado → save antigo 100% compatível.

- ✅ **Todos os mobs em pixel art** (v50): o usuário pediu "deixar os mobs tb em pixel art" (e perguntou se pesaria menos — sim: saiu `spriteGrad`/`createLinearGradient` + curvas/strokes do corpo de TODO mob e entrou `fillRect` RLE). Novo sistema em `index.html` (bloco `v50 — mobs em pixel art`, logo antes de `drawBeyondBody`): (1) `pxMobRows(g,cx,cy,s,rows,pal,flip)` — sprite = array de strings (1 char = 1 px), render RLE (runs de mesmo char viram UM `fillRect`), escala `s=max(1,round(e.r/perPx))`, flip por espelhamento de índices; (2) atlas `PXM` com 17 archetypes de corpo (2 frames onde anima: human/skel/soldier/bat/beast/cons/wisp) + ~28 peças (helm, horns, hornsBig, earsBig, earsElf, beard, fangs, hat, crown, crest2, snout, hood, band, shield, sword, staff, club, bow, wingA/B, spikes, core, crystalTop, crescent, tailBig, stinger, armL/R, head3 p/ hidra); (3) `pxMobDraw(g,e,cx,cy,arch,o)` — escolhe frame A/B por `walkCycle` (ou `flap` p/ asa temporal), bob de caminhada, `o.float` p/ flutuantes, `o.bob` p/ pulo (sapo), partes back/front com paleta mesclada; (4) `pxMobLegs` — pernas em 2 trechos quantizados na grade p/ aranha/escorpião/besouro. **Direção**: o `drawEnemy` já espelha o canvas quando `dirX<0`, então archetypes assimétricos nascem com a cabeça à DIREITA — `beast` e `beetle` levam `flip` interno no `pxMobDraw` (a cabeça original deles era à esquerda); dragão/serpente já nascem corretos. `drawCreatureDetails` (rostos vetoriais por facing) virou no-op comentado — rosto/cabelo/olhos agora vivem nos sprites. Mapeamento: `drawBestiaryBody` (24 ramos de regex mantidos na MESMA ordem: dragon→spider→wasp→scarab→insect→beast→undead→elemental→slime→beholder→minotaur→goblin→troll→gargoyle→serpent→quara→vampire→demon→orc→elf→dwarf→knight→hunter→witch→darkappr→legendBoss→dragonLord→genérico), `drawEnemy` (23 types: shade=frog, wisp/emberWisp=fuga, bat, golem/iceSentinel/glassTitan=cons, cultist/secretGuardian/boss/eclipseBoss=robe, stormDragon/dragonLord=dragon, mireHydra=serpent+3 head3, voidDevourer=blob+olho+tentáculos de pixels, etc.) e `drawBeyondBody` (colossus=cons+arms, futureEcho=losango, organ=blob+tentáculos, war=soldado pixel (v49 era vetor), heroHunter=human+capuz+bow, dreamling/reality/consciousness=wisp, nightmare=ghost, betrayer/nemesis=robe). Auras/flash de hit/evolução/tier/elite/legendary/efeitos (queimando/congelado/etc.) e barras/labels do `drawEnemy` foram mantidos. API nova p/ teste: `__ARCANA__.spawnBeyond(kind)` (hook só de visual, id 932000+len). Sem campos novos de estado → save antigo 100% compatível. Marcadores p/ curl: `mobs em pixel art`, `TODOS os mobs viraram pixel art`. ⚠️ As linhas do atlas precisam manter largura igual por sprite (o centro é por `row[0].length`).


- ✅ **Cache de sprites dos mobs em canvas offscreen — mais fluido com horda** (v50b / sw `arcana-v51`): o usuário sentiu lag depois do pixel art de todos os mobs (v50) — na GPU de verdade cada mob saía com centenas de `fillRect` + troca de `fillStyle` por frame (muitos retângulos pequenos são caros p/ o rasterizador GPU). Fix no `pxMobDraw` (MESMO bloco `v50 — mobs em pixel art`): (1) `MOBCV` = `Map` de cache (limpa acima de 420 entradas), chave `mobCvKey(arch,fi,s,flip,pal,parts)` — inclui frame A/B, escala, flip, paleta completa e as partes (peças referenciadas por id estável de `rows` via `WeakMap` + `mobRowTag`, offsets dx/dy e paleta local); (2) `mobRenderCv(rows,s,flip,basePal,parts)` — desenha o mob inteiro (partes back → corpo → partes front, paleta mesclada `Object.assign({},basePal,p.pal)`) num `<canvas>` offscreen com âncora central em `(W/2-minX, H/2-minY)`; (3) `pxMobDraw` agora só faz `g.drawImage(cv, round(cx-cv.width/2), round(cy+bob-cv.height/2))` — 1 chamada por mob por frame; o `bob` (flutuante/caminhada) e o mirror de direção do `drawEnemy` continuam valendo sobre o canvas inteiro (partes andam junto). Frames A/B e flash de hit geram variantes de cache (normais). A hidra continua desenhando os 3 `head3` direto por `pxMobRows` (leve). **Validação**: A/B com o build v50 (commit `7f7122c` servido em porta separada) — 12 de 14 mobs testados com diff pixel a pixel = 0.000% (offset 0,0, MESMO frame); os 2 restantes (shade/golem) só divergiram por mobs NATURais da cena + aura anilhada animada (fase de `state.time`), não pelo cache. Suite `tests/runtime-expansion.js` 3× PASS; strings exigidas intactas. Bench no sandbox (swiftshader, 24 mobs, sem GPU): neutro (v49 23.9ms / v50 22.9ms / v50b 23.8ms por quadro em DSF=1) — a ganho aparece na GPU real (1 draw call de imagem em vez de ~100 retângulos). Sem campos novos de estado → save antigo 100% compatível. Marcador p/ curl: `v50b: cache de sprites em canvas offscreen`.


- ✅ **Velocidade do herói + atributo Velocidade + Guardião de verdade** (v52): pedido do usuário em 3 partes. (1) **Velocidade base +12%**: na fórmula de movimento do `update` (as 2 ocorrências dx/dy) `(...+attributeSpeedBonus())*1.12` — vale para as 21 classes, não afeta dash/posse (fórmulas separadas) nem a recompensa de fim de história (`player.speed+=12` ficou como estava). Medido no jogo: Guardião 157px/s → ~175px/s. (2) **Atributo Velocidade no menu de Status** (7º, icon 👟): `attributeDefs.speed` (+4 de velocidade de movimento por ponto), `attributes.speed` (reset em `resetPlayer`), helper `attributeSpeedBonus(){return attributes.speed*4;}`, somado na fórmula de movimento. A UI do status mapeia `Object.entries(attributeDefs)` (card novo automático); a linha de valor no card: ``speed:`${Math.round((player.speed+getEquipmentStats().speed+reincFormSpeedBonus()+attributeSpeedBonus())*1.12)} de velocidade de movimento`` (bug "undefined" no card corrigido acrescentando a chave em `values` de `renderStatus`). Save-safe: o load já zera chaves ausentes (`Number(d.attributes[key])||0`). Suite atualizada: 7 atributos (`deepStrictEqual` +`'speed'`, `attributePoints=7`, assert do bônus 4, persistência `speed:0`, `attributeSpeedBonus` na lista `__ARCANA_TEST__`). (3) **Guardião redesenhado** (`o.guardian` nos 4 call sites de `drawPixelHero` — player `state.classId`, doll `state.classId`, criador `pendingClass`, retrato `id`): escudo-torre astral ~2× maior (3 facings: frente cols 3-11 rows 5-15 / lado cols 9-15 rows 6-15 / costas cols 1-7 rows 5-14) com runa em losango brilhante (`astral=mixColor(sc,'#8fe5ef',.65)`), peitoral cobre até o cinto (row 11), runa astral vertical no centro do peito e ombreiras +1 col (cols 2 e 17). Elmo bulwark já existia (`HERO_CLASS_PX.guardian`). Marcadores p/ curl: `v52: +12% de velocidade base`, `escudo-torre do Guardião`, `peitoral do Guardião`, `attributeSpeedBonus`.

- ✅ **Correção de recorte de sprites dos mobs e Aranha redesenhada** (v53 / sw `arcana-v53`):
  1. **Bug do recorte dos mobs resolvido**: em `mobRenderCv`, a âncora `ax/ay` somava `W/2 - minX` (duplicando a largura), deslocando o desenho para fora do canvas offscreen e cortando 75% de todos os monstros (sapos, fagulhas, aranhas pareciam pedaços flutuantes). Corrigido para `ax = -minX, ay = -minY`, restaurando 100% dos corpos em pixel art.
  2. **Sprite da Aranha redesenhado**: (a) Forma do jogador (`Aranha Filhote`, `Tarântula Sombria`, `Aracne`) com 8 patas arqueadas animadas, abdômen redondo, olhos brilhantes expressivos, presas afiadas e torso humanoide na forma Aracne; (b) Monstro `Giant Spider` com 8 patas articuladas (`pxMobLegs`), olhos vermelhos e quelíceras em 2 frames de animação (`PXM.spider`).
  3. **Marcadores p/ curl**: `v53: correcao de recorte de sprites e aranha redesenhada`, `ax=-minX,ay=-minY`. Save 100% compatível.

- ✅ **Guia de missão inteligente com rota automática por portais** (v54 / sw `arcana-v54`):
  1. **Navegação entre zonas com portais**: novo helper `routeObjective(obj)` — quando o objetivo está em outra região (ex.: Capitã Selene / Cripta do Eclipse em Auralis a `x>2850`, e o jogador está no Vale de Lumina a `x<=2850`), a seta amarela e a contagem de distância guiam automaticamente até o **Portal para Auralis** (`travelPortals[0]` a `(920, 1110)`) com o label inteligente `Portal para Auralis (→ Capitã Selene)`. Assim que o jogador cruza o portal, a seta aponta diretamente para o NPC/monstro no destino.
  2. **Cripta do Eclipse**: busca de cultistas restrita à região de Auralis (`e.x > 2850`) e posição de retorno da Capitã Selene sincronizada com `npcObjPos('captain', 3280, 1600)`.
  3. **Marcadores p/ curl**: `v54: guia de missao inteligente`, `routeObjective`. Save 100% compatível.

- ✅ **Correção definitiva de sprites dos mobs e Aranha** (v55 / sw `arcana-v55`):
  1. **Fix do recorte de todos os mobs no canvas offscreen**: `mobRenderCv` ancorado em `ax = -minX, ay = -minY` garantindo que 100% do corpo de sapos, fagulhas, aranhas e chefes fique totalmente dentro do canvas sem nenhum corte.
  2. **Aranha do jogador e do bestiário**: 8 patas arqueadas articuladas, olhos brilhantes e presas.
  3. **Marcadores p/ curl**: `v55: correcao definitiva de recorte dos mobs`, `ax=-minX,ay=-minY`. Save 100% compatível.

- ✅ **Guarda Ágil / Aparar com Armas e Interceptação de Magias** (v56 / sw `arcana-v56`):
  1. **Aparar com armas para classes sem escudo**: Assassino, Ladino, Monge, Ranger, Pistoleiro e Engenheiro agora usam o Botão Direito (ou botão `⚔️` no mobile) para postura de aparar/guarda (`weaponGuardStyles`).
  2. **Janela de Aparo Perfeito (Parry)**: timing preciso anula 100% do dano com faíscas metálicas (`CLANG!`), som cortante e rebate projéteis com dano aumentado baseado no ataque do jogador.
  3. **Cálculo de Ataque & Defesa**: Poder de bloqueio combina Ataque + Defesa + Vigor. Se o ataque inimigo for superior ao poder do jogador, o excesso penetra como dano parcial (`✦ APARADO PARCIAL`); se o jogador for mais forte, bloqueia completamente.
  4. **Cortar e destruir magias ao atacar**: Golpes corpo a corpo (básico/adaga/espada) cortam e anulam magias inimigas no ar se o Ataque do jogador for maior/igual ao poder da magia (`✦ MAGIA DESTRUÍDA! 💥`) ou rebatem projéteis físicos (`✦ REBATIDO! ⚔️`).
  5. **Interceptação no ar de projéteis**: Disparos e adagas lançadas do jogador que colidem com projéteis inimigos em voo interceptam e destroem a magia no ar. Save 100% compatível.

## 9. Coisas que PODEM ser pedidas no futuro

- Balanceamento detalhado de dano/HP/XP por classe.
- Novas missões ou conteúdo de história.
- PvP (o toggle do minimapa já ajuda).
- Novas funções pra Forja / arma viva.
- Se o dono quiser domínio próprio, pode configurar no GitHub Pages (Settings → Pages → Custom domain).

## 10. Fluxo padrão de trabalho (resumo)

1. Leia este arquivo e confirme o link publicado.
2. Faça as mudanças no(s) arquivo(s) necessário(s) (quase sempre só `index.html`).
3. Teste localmente com `python3 -m http.server 8000`.
4. Cuide para **não quebrar o save** (seção 5).
5. Se mudou o cache offline, suba a versão no `sw.js` (`arcana-v2` → `v3`...).
6. Commit:
   ```bash
   git add -A
   git commit -m "O que mudou"
   git push origin main
   ```
7. Espere ~40s e confirme que o site atualizou.
8. Avise o dono do jogo em português simples, com o que mudou e o que ele deve testar.

---

- ✅ **v77 — TRAVA de layout no deitado (barra 1-6 voltava para cima)**:
  - Causa raiz: a barra 1-6 que ficava no meio da tela era o próprio `#mobileQuickbar` (DOM). O editor de HUD salva o arraste em `buttonPos.quick-*` / transform do `.touch-ui` e aplica via `el.style.translate`/`.touch-ui{transform}` (inline, aplicado por `applyHUDCustomTouch`). A CSS de posição (`right/bottom`) só move a âncora — o `translate` inline vencia o CSS (inline > stylesheet mesmo sem !important), então a barra arrastada continuava alta. O editor de HUD no celular deixa os controles vulneráveis a esse deslocamento acidental.
  - Fix: no `@media (pointer:coarse) and (orientation:landscape)`, **trava** todo o arranjo de toque: `.touch-ui{transform:none!important}` e `.touch-ui *{translate:0 0!important;scale:1 1!important}`. Assim o deitado SEMPRE usa o layout fixo (mesma especificidade/`!important` no CSS que o inline, mas o reset zera qualquer valor salvo). Retrato (em pé) e desktop não são afetados (lá o editor segue valendo).
  - Migração renovada para flag `arcana-touch-layout-v === 'v77'` (zera `buttonPos/buttonScale/touch*` na primeira abertura).
  - Marcadores: `TRAVA: no deitado`, `v77 — LAYOUT PAISAGEM`. Save 100% compatível. sw.js → `arcana-v77`.

- ✅ **v76 — Barra 1-6 volta para embaixo do E/escudo, rente ao analógico (foto 08:20) + chat centralizado e baixo**:
  - **Quickbar reposicionado para o padrão desejado** (na foto 1 ela estava ALTA, na foto 2 o Ronald mostrou onde quer): `right:132px` normal (embaixo do E/além/escudo e imediatamente à esquerda do analógico de ataque), `right:118px` compacta, `right:108px` ultra. (A v75 tinha puxado para 196/166/146, ficando longe do analógico.) A barra estava alta porque o Ronald a arrastara no editor de HUD (buttonPos salvo) — a migração `arcana-touch-layout-v === 'v76'` zera essas posições arrastadas para o padrão travar.
  - **Chat centralizado e embaixo** ("no meio, alinhado, pode pôr mais embaixo"): na camada normal `left:50%` (centro de verdade), `bottom:16px`, largura `min(280px,25%)`; quando aberto sobe para `bottom:66px`. Compacta (`bottom:48`, 26%) e ultra (`bottom:50`, 22%) sobem só um pouco para não encostar na barra em telas estreitas.
  - Geometria validada sem sobreposição. Marcadores: `v76 — LAYOUT PAISAGEM`, flag v76. Save 100% compatível. sw.js → `arcana-v76`.

- ✅ **v75 — Barra 1-6 deslocada para a esquerda (foto 02:11) + chat ainda mais embaixo**:
  - **Quickbar (barra 1-6) puxada à esquerda** no deitado: `right:196px` normal (era 132), `right:166px` compacta (era 118), `right:146px` ultra (era 108) — agora a fileira termina logo abaixo dos botões E/🌌 e **não chega perto do analógico de ataque** (na foto do Ronald ela fica abaixo do E/além, começando sob o E), como ele pediu ("1 ao 5 tem que ficar igual a essa print").
  - **Chat ainda mais embaixo**: na camada normal (tela larga, que é o caso dele) `bottom:18px` (era 62), `left:41%`, largura `min(260px,24%)` — vai para o vão central baixo, entre os botões da esquerda e a barra da direita. Nas camadas compacta (`bottom:52`, `left:44%`, 200px) e ultra (`bottom:54`, `left:45%`, 150px) ele sobe um pouco por cima da barra para não colidir quando a tela é estreita.
  - Migração de arranjo de toque renovada para flag `arcana-touch-layout-v === 'v75'`. Geometria validada sem sobreposição de 600px a 1170px.
  - Marcadores p/ curl: `v75 — LAYOUT PAISAGEM`, flag v75. Save 100% compatível. sw.js → `arcana-v75`.

- ✅ **v74 — Arranjo paisagem FINAL (foto 01:59) + chat bem mais embaixo**:
  - **Direita vira 3 fileiras empilhadas** (antes a barra 1-6 ficava no centro-esquerda): de cima para baixo — **⇧ dash** (acima do escudo, `right:140`,`bottom:124`) · fileira **E dourado (right:262) · além 🌌 (right:200) · escudo 🛡 (right:140)** (`bottom:58–62`) · **barra de habilidades 1-6 em fileira horizontal ancorada à direita** (`right:132`,`bottom:12`, abaixo do E/além/escudo, à esquerda do analógico de ataque). Analógico de ataque continua inteiro no canto (`right:14`).
  - **Esquerda inalterada** (aprovada): analógico de andar + **pata 🐾 (left:140)** e **poção 🧪 (left:202)** à sua direita.
  - **Chat BEM mais embaixo e mais estreito**: `bottom:62`, `left:46%` (ligeiramente à esquerda do centro, assim não encosta nos botões da direita), largura `min(280px,30%)`; log ~22vh. (Antes estava em `bottom:96`, largo 40% — o Ronald pediu "muito mais em baixo".)
  - 3 camadas por largura: normal ≥900, compacta ≤899 (botões 42px / quickbar 34px / chat `bottom:56`), ultra ≤699 (40/30px / chat `bottom:50`). Validadas sem sobreposição até ~640px.
  - Migração de arranjo de toque renovada para a flag `arcana-touch-layout-v === 'v74'` (zera buttonPos/buttonScale/touch* uma vez).
  - Marcadores p/ curl: `v74 — LAYOUT PAISAGEM`, `arcana-touch-layout-v` (v74). Save 100% compatível. sw.js → `arcana-v74`.

- ✅ **v73 — Layout paisagem (deitado) no arranjo escolhido pelo Ronald (foto 01:38)**:
  - **Esquerda:** analógico de andar (110px) rente à borda; **pata 🐾 (left:140) e poção 🧪 (left:202)** ficam à direita dele (antes estavam no lado direito da tela). Ancoradas por `left` (não por `right`).
  - **Direita:** fileira horizontal **E dourado (56px, right:262) · além 🌌 (right:200) · escudo 🛡 (right:140)** na mesma faixa (bottom ~28–34), com o **dash ⇧ acima do escudo** (bottom:92). O **analógico de ataque fica INTEIRO no canto** (right:14, 110px) — antes o arranjo personalizado do Ronald (via editor de HUD, guardado em `hudCustom.buttonPos`) o empurrava para fora da tela e cortava metade.
  - **Barra de habilidades (1-6):** 1 fileira horizontal ancorada à **esquerda** (`left:270`, `right:auto`) → centro-baixo, entre a poção e o E, sem colidir.
  - **Chat:** centralizado na **parte de baixo** (acima da barra), `bottom:96`, largura `min(300px,40%)`.
  - **3 camadas por largura** (todas validadas sem sobreposição): normal (≥900px), compacta (`max-width:899px`: botões 42px, joysticks 96px), ultra (`max-width:699px`: botões 40px/30px, joysticks 88px).
  - **Migração única (`arcana-touch-layout-v`):** ao abrir, zera SÓ o arranjo de toque personalizado (`buttonPos`, `buttonScale`, `touchScale/Opacity/Offset`) para o novo padrão não ser sobreposto nem cortar o analógico; mantém atalhos, áudio, HUD e escala da barra de habilidades.
  - Marcadores p/ curl: `v73 — LAYOUT PAISAGEM`, `arcana-touch-layout-v`. Save 100% compatível (só CSS/UI + limpeza de preferência local). sw.js → `arcana-v73`.

- ✅ **v72 — CONTROLES MOBILE REATIVADOS (bug crítico) + topo reorganizado + chat embaixo** (baseado em foto do Ronald no Chrome deitado):
  - **🐞 Bugs raiz que escondiam TODO o controle de toque:**
    1. `.touch-ui` (container de joysticks/botões) nascia com `display:none` e **nada nunca religava** — a classe `.shown` só setava transform/opacidade (sem `display`) e não era adicionada em lugar nenhum. Fix: `.touch-ui.shown{display:block;...}` + `updateMobileLayout()` agora faz `touchUI.classList.toggle('shown', show)` (show = started && touch && !paused && !dialogOpen). `updateMobileLayout()` é chamada dentro do `updateAdaptiveDOM()` (roda ~10x/s no jogo) e no `applyDeviceProfile`.
    2. **Analógico de ataque (twin-stick) estava incompleto**: o `#attackJoystick` nascia com `hidden` e NÃO tinha nenhum handler de toque nem disparava (só o de movimento tinha IIFE). Fix: novo IIFE espelhando o de movimento (mira 360°, `attackJoystick.dx/dy/active`), `updateMobileLayout` remove o `hidden` em tela de toque, e no `update()` (ao decrementar `attackCd`) **dispara sozinho** enquanto o analógico é arrastado (`hypot(dx,dy)>.25`). `getAim()` já lia esse vetor (só faltava alimentá-lo).
    3. Regra `.mobile-quickbar{display:block!important}` (mobile) forçava a barra de habilidades para bloco vertical; separada em `.mobile-smart-actions{display:block}` e `.mobile-quickbar{display:grid}`.
  - **Menu ☰ não fica mais em cima da vida (topo-esq):** `.device-menu-btn` não tinha `position` (as coords right/top eram ignoradas → caía no fluxo normal, no canto superior esquerdo sobre o HUD de vida). Agora `position:fixed!important;z-index:100002;...` → vai para o **topo direito**.
  - **Botão do mapa (🗺️ Ocultar) agrupado ao lado do Menu, no topo** (`#minimapTab`: `top:10px`, `right:108px`, z 100002; media ≤520px ajustada p/ `right:104px`) — não flutua mais sobre o minimapa.
  - **Minimapa desce no deitado** para não ficar sob os botões do topo: helper `minimapTop()` = touch+landscape ? **108** : 60; usado no `drawMinimap()` e nas DUAS áreas de clique/zoom (mouse e toque, antes fixas em `py>=58&&py<=198`, agora `minimapTop()-2 … +142`). Abaixa dentro dos 540px internos (em paisagem H=540 fixo), ficando logo abaixo da faixa de botões.
  - **Chat volta para BAIXO no deitado** (Ronald pediu): `bottom:118px` (fechado) / `128px` (aberto), centralizado, largura `min(330px,46%)` — acima da faixa de controles para não encostar nos botões; log limitado a ~26vh. (Antes a v71 tinha jogado o chat no topo.)
  - **Posições dos botões (faixa baixa, da v71) mantidas** e validadas sem sobreposição.
  - Marcadores p/ curl: `Analógico de Ataque (twin-stick)`, `CONTROLES MOBILE REATIVADOS`, `minimapTop`. Save 100% compatível (só CSS/UI + lógica de input; nenhum campo de save novo). sw.js → `arcana-v72`.

- ✅ **v71 — Layout Mobile PAISAGEM (deitado) redesenhado, sem sobreposições**:
  - Bloco CSS novo `/* v71 — LAYOUT MOBILE PAISAGEM */` dentro do `<style>` (logo antes do fim), com `@media (pointer:coarse) and (orientation:landscape)`. **Não toca no modo retrato (em pé) nem no desktop.**
  - **Controles todos numa faixa baixa e fáceis de alcançar** no modo deitado: joysticks menores (116px; 104px no ultracompacto) e rentes ao rodapé com `env(safe-area-inset-bottom)`; botão **E** dourado colado acima do joystick de ataque (right:46, bottom:128); leque de ações em 2 colunas à esquerda do joystick (dash ⇧ / além 🌌 na coluna de baixo e cima; poção 🧪 e escudo 🛡/🔮 na coluna ao lado). Antes os botões subiam até bottom:198–252, ficando perto do topo e fora do alcance do polegar em telas baixas.
  - **Companhia 🐾 em altura própria** (3ª fileira, bottom:128 normal / 116 compacto): Necromante e Invocador têm escudo de mana 🔮 **e** pet 🐾 ao mesmo tempo — antes os dois caíam na mesma posição e se sobrepunham.
  - **Barra de habilidades vira 1 fileira horizontal** ancorada à direita (à esquerda do leque de ações): `grid-template-columns:repeat(6,42px)` (38px no compacto), sem mais quebrar em grade 3×2 que colidia com os botões.
  - **Chat (dock) sobe para o TOPO central no deitado** (`top:8px`, largura `min(340px,46%)`), liberando 100% da faixa inferior para jogar — antes ele ficava no centro-embaixo (92% de largura) e caía em cima de joystick/botões em telas baixas. Log limitado a ~30vh.
  - **Camada ultracompacta** para celular deitado baixo/estreito (`max-width:700px` ou `max-height:340px`): botões 44px, joysticks 104px, quickbar 38px — verificada sem sobreposição até 590×300.
  - Verificação de geometria (script Node que simula os retângulos): **0 sobreposições e nada fora da tela** em 900×420, 850×400, 740×360, 660×360, 620×320 e 590×300, inclusive com TODOS os botões visíveis (Invocador nv15+).
  - Marcadores p/ curl: `v71 — LAYOUT MOBILE PAISAGEM`, `Chat: sobe para o TOPO`. Save 100% compatível (só CSS). sw.js → `arcana-v71`.

- ✅ **v70 — HUD Mobile Twin-Stick Perfeito, Botão "E" & Minimapa Desacoplado**:
  - Aba flutuante do minimapa (`#minimapTab`) movida para o cabeçalho superior ao lado do botão `[☰ Menu]`, eliminando 100% de qualquer sobreposição visual no mapa.
  - Botão de Interação **"E"** (`#interactTouch`) posicionado em destaque dourado logo acima do analógico de ataque (fácil alcance do polegar direito para falar com NPCs, abrir baús e coletar).
  - Coluna de ações inteligentes (`#dashTouch`, `#healTouch`, `#guardTouch`, `#companionTouch`) e barra rápida de habilidades (`#mobileQuickbar`) reorganizadas sem cortes de tela ou sobreposições.
  - Service worker atualizado para a versão de cache `arcana-v70`.

- ✅ **v69 — Analógico de Ataque (Twin-Stick) & Correção Definitiva dos Cards de Classe**:
  - Adicionado o analógico de ataque e mira (`#attackJoystick`) no canto inferior direito para celulares e telas touch, com ícone de espada ⚔, mira precisa 360° e disparo contínuo automático ao arrastar.
  - Correção definitiva do layout dos cartões de seleção de classe (`.class-card`), isolando o texto em `.class-card-body` e fixando o retrato `.class-portrait` em bloco lateral sem sobreposição nas media queries mobile.
  - Ocultamento estrito da aba flutuante do minimapa fora do mapa de jogo ativo.
  - Service worker atualizado para a versão de cache `arcana-v69`.

- ✅ **v68 — Otimização Mobile Responsiva Completa & Correção de Layout**:
  - Refatoração dos cartões de seleção de classe (`.class-card`) com Flexbox estruturado e container `.class-card-body`, impedindo totalmente que o canvas do personagem cubra o título e a descrição da classe em qualquer resolução ou orientação de tela mobile.
  - Ocultamento estrito da aba flutuante do minimapa (`#minimapTab` / "Ocultar") fora do gameplay ativo (telas de início, seleção de classe, login, splash e modais).
  - Media queries responsivas aprimoradas para telas pequenas e modo paisagem (`@media (max-height: 520px)`, `@media (max-width: 520px)`, pointer coarse).
  - Atualização do Service Worker para a versão de cache `arcana-v68`.

- ✅ **v67 — Painel de Administrador (F2) 100% Interativo & Correções**:
  - Atalho F2 no teclado agora abre/fecha o painel de administrador diretamente.
  - Botão 👑 Painel ADM no menu superior (topActions) integrado.
  - Correção completa do erro `escapeHtml`, permitindo total interatividade dos botões de inspecionar jogadores, doar moedas, teleporte rápido, controle de clima e transmissão global.
  - Sistema de guarda ágil e aparo com armas (assassin, rogue, monk, ranger, gunslinger, engineer) perfeitamente integrado.
  - Service worker atualizado para cache `arcana-v67`.

- ✅ **v58 — Iluminação Volumétrica & Gráficos Aprimorados**:
  - Bloom e brilho suave multi-estágio (`drawScreenGlow`).
  - Camadas exuberantes de realce na copa das árvores com movimento orgânico do vento (`drawTree`).
  - Suporte completo ao Co-op Multiplayer Realtime com balões de fala, sincronização de dano e teleporte.
  - Guarda e aparo dinâmico com armas e adagas para classes ágeis.
  - Testes 100% validados via `node tests/runtime-expansion.js`.

**Bom trabalho, e cuide bem do Arcana.** 🧙‍♂️✨

- ✅ **Equipamento integrado em pixel (bug das armaduras corrigido)**: antes, `drawEquippedArmor/Helmet/Gloves/Boots` desenhavam caixas/elmos vetoriais por cima do herói em pixel (ficava tudo torto e cobria o rosto). Agora o equipamento muda as CORES do sprite: no `drawPixelHero`, `gearArmor` tingir tronco/braços (`mixColor(base,cor,.5)`), `gearGloves` tingir as mãos, `gearBoots` tingir as botas e `gearHelmet` desenha um elmo em pixel (dome 6 linhas, metal p/ comum ou cor da raridade p/ raros+, filete dourado p/ épicos+) que substitui o chapéu da classe. As cores vêm de `gearColOf` (encantamento > raridade) no `drawPlayer`. No `drawPlayer` só o `drawEquippedCharm` continua vetorial (é pequeno). A forma Goblin de reencarnação ainda usa os vetoriais antigos (deixado de propósito).
