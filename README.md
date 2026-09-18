# Meu Dia

App pessoal de organização diária da Alltoral — um quadro semanal com um post-it por dia (segunda a domingo), mais um checklist de planejamento semanal. Front-end puro (HTML/CSS/JS, um único arquivo), sincronizado em tempo real via Firebase Firestore, instalável como app (PWA).

> Este LEIA-ME substitui a versão antiga (que descrevia uma sincronização via Netlify Blobs/funções serverless). O app não usa mais isso — a sincronização hoje é 100% Firebase Firestore, direto do navegador.

## Estrutura

```
/
├── index.html                ← o app inteiro (HTML + CSS + JS em um único arquivo)
├── manifest.json              ← metadados do app instalável (PWA)
├── sw.js                      ← service worker (busca a rede primeiro; cai pro cache se ficar offline)
├── icon-192.png / icon-512.png
├── larot-avatar.png           ← personagem do Larot (mic acordado)
├── larot-avatar-muted.png     ← personagem do Larot (mic parado/erro)
└── LEIA-ME.md                 ← este guia
```

As pastas `netlify/functions`, `netlify.toml` e `package.json` que ainda estão no repositório são resquício da versão antiga e não são usadas pelo app atual — podem ficar ou ser removidas, tanto faz.

Todos os arquivos da lista acima precisam ficar juntos, na **raiz** do repositório, para os caminhos relativos (`./manifest.json`, `./sw.js`, `./larot-avatar.png` etc.) funcionarem.

## 1. Criar o projeto no Firebase

1. Acesse [console.firebase.google.com](https://console.firebase.google.com) e crie um projeto (ou use um existente).
2. **Firestore Database** → *Criar banco de dados* → modo produção.
3. **Firestore Database → Regras** — como este app não tem login, os dados ficam num board fixo (`alltoral-meudia`) sem trava de usuário; use regras abertas só pra esse app, ou restrinja por outro meio se preferir mais segurança.
4. **Configurações do projeto → Seus apps** → adicione um app **Web** (ícone `</>`) e copie o objeto `firebaseConfig` gerado.

## 2. Colar a configuração no `index.html`

Abra `index.html`, procure por `firebaseConfig` (dentro do `<script type="module">`, perto do início) e substitua pelos seus valores.

## 3. (Opcional) Sincronizar com o Google Agenda

O botão "sincronizar com Google Agenda" usa OAuth do Google direto no navegador (Google Identity Services), sem precisar de servidor. Procure por `GOOGLE_CLIENT_ID` no `index.html` e cole o Client ID de um projeto no Google Cloud Console (com a API do Google Calendar habilitada). Sem isso configurado, o botão mostra uma mensagem avisando que falta configurar.

## 4. Publicar com GitHub Pages

1. Suba este repositório pro GitHub (com `index.html` na raiz).
2. **Settings → Pages** → Source: `Deploy from a branch` → branch `main`, pasta `/root` → Save.
3. Após alguns minutos, o GitHub Pages gera uma URL tipo `https://seu-usuario.github.io/nome-do-repo/`.

Como o `sw.js` busca a rede primeiro (só cai pro cache se estiver offline), atualizações costumam aparecer com um simples recarregar da página — sem precisar de Unregister/Clear Site Data, exceto em casos raros de cache mais teimoso.

## Como funciona o quadro semanal

- Cada dia da semana (segunda a domingo) é um post-it com uma lista de tarefas — clique em "+ tarefa" pra adicionar, no texto pra editar, no ✕ pra excluir, na caixinha pra marcar como feita.
- As setas `‹` `›` no topo navegam entre semanas; "hoje" volta pra semana atual.
- **Checklist semanal ("Essa semana")** — painel rosa acima dos dias, pra anotar o que precisa fazer naquela semana antes de distribuir nos dias específicos.
- **Arrastar entre dias** — cada item (tanto nos dias quanto no checklist semanal) tem um "⠿" do lado; arraste-o pra outro dia que ele **se move** pra lá (some de onde estava). Funciona com o dedo no celular, não só com mouse.
- Não tem login: todo aparelho que abre esse link compartilha o mesmo quadro (`alltoral-meudia`), com fallback pro armazenamento local do navegador se ficar sem internet.

## Larot (assistente de voz)

O personagem no canto inferior direito é o **Larot** — toque nele uma vez pra falar, e de novo quando terminar.

- **100% local e gratuito** — reconhecimento de padrões em JavaScript (Web Speech API), sem IA nem API paga.
- **Sem resposta falada**, exceto nas consultas de planejamento semanal (abaixo), que ele fala em voz alta.

**Comandos:**

- **Post-it rápido** — "comprar ração do gato" (sem falar nenhuma data, cria no dia de hoje).
- **Post-its em vários dias numa frase só** — "amanhã comprar leite, sexta reunião com cliente, dia 20 pagar conta" cria um post-it em cada dia mencionado.
- **Modo planejamento** — fale **"vamos planejar a semana"** (ou "organizar a semana") pra abrir o modo: a tela pula pra semana seguinte e o quadro "Essa semana" pisca uma borda amarela. A partir daí, cada frase que você falar vira um item novo no checklist — "ligar pro Lucas", toca de novo, "comer fora", toca de novo, "trocar a ração do Maurício"... Fale **"terminar planejamento"** (ou só "pronto"/"chega") pra sair do modo.
- **Consultar o planejamento** — "essa semana" (sozinho) lê em voz alta o que já está no checklist da semana atual; "semana que vem" lê o da semana seguinte.

Tem um botão 🐞 no canto inferior esquerdo que abre um painel de depuração, mostrando em tempo real o que o Larot está ouvindo/processando — útil pra diagnosticar algo que não funcionou como esperado no celular.

## Limitações a saber

- Sem login, qualquer pessoa com o link do app acessa o mesmo quadro. Pense nisso como um app de uso pessoal/família, não pra dados sensíveis compartilhados com estranhos.
- Último a salvar vence — não há resolução de conflito se dois aparelhos editarem o mesmo dia ao mesmo tempo.
- O reconhecimento de voz depende do navegador (Web Speech API) — funciona melhor no Chrome; no Safari/iPhone é mais limitado.
