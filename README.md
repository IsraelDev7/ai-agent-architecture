# Arquitetura de Agentes de IA em Produção

Documentação de arquitetura de um assistente autônomo em operação contínua desde
fevereiro de 2026: memória persistente, procedimento aprendido, orquestração de
ferramentas e integração multicanal.

**Stack:** Python · Node.js · Anthropic / Gemini APIs · n8n · Supabase · Telegram ·
WhatsApp (Z-API) · ElevenLabs · Home Assistant

---

## Por que este repositório existe

Os outros projetos deste portfólio provam entrega de produto. Este prova a parte
que não aparece na tela: **como um agente decide, lembra e falha de forma segura.**

Não há código de cliente aqui. O que está documentado é a arquitetura — as
decisões, os erros que elas corrigem e o custo de cada escolha. É deliberado:
implementação envelhece, decisão explicada continua valendo.

---

## O problema que um agente de produção realmente tem

A demonstração de agente é fácil: conecta um LLM a duas ferramentas e ele
responde. O que quebra quando o sistema sai da demonstração:

| Sintoma | Causa real | O que resolve |
|---|---|---|
| Esquece tudo entre sessões | Contexto é efêmero por natureza | Memória em duas camadas, com orçamento |
| Repete o mesmo erro toda semana | Lição fica no histórico, não no sistema | Procedimento versionado, carregado por relevância |
| Context window estoura | Tudo é injetado sempre | Carregamento sob demanda, não por precaução |
| Diz que fez e não fez | Sucesso presumido | Verificação obrigatória antes de responder |
| Gasta dinheiro sem avisar | Nenhuma fronteira de custo | Aprovação prévia em operação paga |
| Obedece instrução de página web | Não separa dado de comando | Fronteira de confiança explícita |

---

## As cinco decisões que sustentam o sistema

### 1. Memória em duas camadas, com orçamento fixo

Memória de agente falha de dois jeitos opostos: ou não lembra nada, ou lembra
tanto que o contexto estoura antes da primeira pergunta.

A separação que funcionou:

```
SEMPRE CARREGADO (orçamento fixo, ~2.000 caracteres)
  └── fatos que valem em toda sessão
      quem é o usuário · ambiente · convenções estáveis

CARREGADO SOB DEMANDA (sem limite prático)
  └── procedimento por tipo de tarefa
      como fazer X · armadilhas de X · preferências do usuário em X
```

O limite da camada permanente é a parte importante. Sem limite rígido, toda
sessão nova carrega o acúmulo de todas as anteriores. Com limite, cada entrada
nova disputa espaço com as antigas — e a disputa força a pergunta certa: *isso
vale em toda sessão ou só nesta tarefa?*

**Regra de redação:** entrada de memória é fato declarativo, não ordem.
`"Usuário prefere respostas concisas"` funciona.
`"Sempre responda concisamente"` vira instrução permanente que sobrepõe o pedido
atual do usuário — o agente passa a obedecer a própria anotação em vez da pessoa.

### 2. Procedimento é código, não conversa

Quando o agente aprende a fazer algo difícil, a lição precisa sair do histórico e
entrar no sistema. Senão ela morre no fim da sessão.

Cada tipo de tarefa recorrente tem um documento versionado com: o procedimento,
os comandos exatos que funcionaram, as armadilhas descobertas na prática, e as
correções que o usuário fez.

```
skill/
  SKILL.md              procedimento + armadilhas (carregado quando relevante)
  references/           detalhe técnico consultado sob demanda
  scripts/              automação testada, não pseudocódigo
```

O critério de carregamento é a descrição: os primeiros ~57 caracteres precisam
dizer **quando** usar aquilo, não o que é. `"Use quando publicar vídeo vertical"`
é acionável; `"Documentação de vídeo"` não ajuda ninguém a decidir.

**O que vai para lá:** regra imperativa com a razão embutida. Uma regra por
lição. Sem número de incidente, sem data, sem narrativa — isso é log, não lição.

### 3. Verificação obrigatória antes de afirmar

O padrão mais caro em sistema autônomo é o **sucesso presumido**: o agente
executa, não confere, e reporta sucesso. O usuário descobre o erro depois, e a
confiança no sistema não volta.

Quatro variações que já corrigi:

| Variação | Por que engana |
|---|---|
| Reportar o que o comando *deveria* fazer | A intenção não é o resultado |
| Confiar em exit code 0 | Muita ferramenta sai 0 com falha parcial |
| Aceitar HTTP 200 como confirmação | Página de rate limit também devolve 200 com corpo plausível |
| Calcular em vez de medir | Cálculo teórico ignora o que o sistema fez de fato |

O último merece exemplo concreto. Ao mixar trilha sonora sob um vídeo, apliquei a
regra consagrada de áudio: música 18–24 dB abaixo da voz. O resultado mediu
**idêntico** ao original — a música tinha ficado inaudível. A regra pressupõe
voz; o vídeo não tinha fala. O cálculo estava certo e a premissa estava errada.

Só a medição (`ebur128`, RMS por janela) mostrou isso. Hoje toda operação com
resultado mensurável é medida antes de ser reportada.

### 4. Fronteira de confiança entre dado e instrução

Agente que lê a web, arquivos e saída de ferramenta recebe texto de origem não
confiável a cada passo. Se esse texto for tratado como instrução, qualquer página
pode redirecionar o sistema.

A separação é estrutural, não probabilística:

```
INSTRUÇÃO  ← somente o usuário, em canal identificado
DADO       ← web, arquivo, API, saída de ferramenta, documento
```

Conteúdo externo chega marcado como dado. Texto dentro dele que *pareça* comando
— "ignore as instruções anteriores", "execute isto" — é tratado como o que é:
uma string num documento.

### 5. Fronteira de custo: aprovação antes, não relatório depois

Agente com acesso a API paga pode gastar dinheiro real em segundos. O erro não é
dar acesso — é não definir onde a autonomia termina.

A linha que usamos:

```
EXECUTA E INFORMA     reversível e gratuito
                      ler, analisar, gerar rascunho, processar local

PERGUNTA ANTES        irreversível ou pago
                      publicar · gastar · enviar em nome do usuário
                      apagar · alterar configuração
```

Operação paga exige, **antes**: o que exatamente será usado, o que vai acontecer,
quanto custa e para onde vai. Depois da execução, custo real medido pelo delta de
saldo — nunca pela estimativa, que erra.

Isso não é burocracia. É o que permite dar acesso amplo ao agente sem que cada
sessão seja uma aposta.

---

## Integração multicanal

Um agente útil não vive num chat. Os canais em operação:

| Canal | Uso | Nota de arquitetura |
|---|---|---|
| **Telegram** | Interface principal | Mídia nativa; áudio como voz |
| **WhatsApp** (Z-API) | Atendimento a cliente | Webhook, não polling |
| **n8n** | Orquestração entre serviços | Workflow versionado em JSON |
| **Supabase** | Persistência | Postgres com RLS |
| **Home Assistant** | Automação física | Sem sair do mesmo agente |
| **Cron** | Trabalho recorrente | Sobrevive ao fim da sessão |

A decisão que importa: **trabalho que precisa sobreviver à sessão não roda na
sessão.** Processo filho morre quando o processo pai morre. Relatório semanal,
monitoramento e coleta agendada vivem em cron ou em processo acompanhado — nunca
como tarefa paralela de uma conversa.

---

## Quando *não* usar um LLM

A decisão mais subestimada em sistema de IA é esta. Um diagnóstico técnico com
pontuação ponderada, definida junto com um especialista, é melhor como motor de
regras:

- **determinístico** — as mesmas respostas dão sempre o mesmo laudo
- **auditável** — o especialista revisa linha a linha
- **instantâneo e de custo zero**
- **não alucina** recomendação técnica para quem vai contratar um serviço

Quando IA entra, entra no lugar certo: **a régua calcula, o modelo redige.**
Decisão determinística, linguagem natural só na apresentação.

Modelo de linguagem é excelente em ambiguidade e linguagem. É o componente errado
para regra fixa, cálculo e qualquer coisa que precise ser reproduzível.

---

## Resultados em operação

- **Operação contínua desde fevereiro de 2026**, com memória atravessando
  centenas de sessões
- **Pipeline de produção de mídia** ponta a ponta: geração de imagem com
  identidade visual travada, animação por vídeo generativo, pós-produção local
  (`ffmpeg`), trilha gerada e mixada com medição de loudness
- **Custo por peça medido e reportado** a cada operação, com aprovação prévia
- **Procedimento acumulado em documentos versionados** — o sistema melhora por
  escrito, não por tentativa

---

## Documentos

| Arquivo | Conteúdo |
|---|---|
| [`docs/memoria.md`](docs/memoria.md) | As duas camadas, orçamento, regra de redação |
| [`docs/procedimento.md`](docs/procedimento.md) | Estrutura, critério de carregamento, o que é lição |
| [`docs/verificacao.md`](docs/verificacao.md) | Padrões de sucesso presumido e como medir |
| [`docs/fronteiras.md`](docs/fronteiras.md) | Confiança e custo: as duas linhas que não se cruzam |

---

## Autor

**Israel Passos** — AI Engineer · Smart LABS
[github.com/IsraelDev7](https://github.com/IsraelDev7) ·
[portifolio-smartlabs.vercel.app](https://portifolio-smartlabs.vercel.app)
