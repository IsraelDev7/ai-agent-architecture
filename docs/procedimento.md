# Procedimento: a lição vira código, não conversa

## O problema

O agente resolve algo difícil depois de cinco tentativas. Descobre que a
ferramenta exige um formato específico, que a documentação oficial está
desatualizada, que o usuário prefere outra abordagem.

Tudo isso fica no histórico da sessão. Na semana seguinte, o mesmo problema custa
as mesmas cinco tentativas.

**Lição que não sai do histórico não é aprendizado — é anedota.**

---

## A estrutura

```
skill/
  SKILL.md              procedimento + armadilhas (carregado quando relevante)
  references/           detalhe técnico, consultado sob demanda
  scripts/              automação testada
  templates/            ponto de partida reutilizável
```

A separação entre `SKILL.md` e `references/` existe por custo de contexto. O
procedimento principal é curto e carrega inteiro; o detalhe fica num arquivo
separado, lido só quando aquele passo específico aparece.

---

## O critério de carregamento é a descrição

O agente decide carregar um procedimento lendo apenas a descrição dele. Então os
primeiros caracteres precisam dizer **quando** usar, não o que é.

```
✓ "Use quando publicar vídeo vertical no Instagram. Queima texto e mixa trilha."
✗ "Documentação sobre vídeo e redes sociais."
```

A primeira é acionável: o gatilho está explícito. A segunda não ajuda ninguém a
decidir — e procedimento que não é carregado no momento certo é procedimento que
não existe.

**Formato:** `Use quando <gatilho>. <comportamento em uma linha>.`

---

## O que é lição e o que é log

Esta distinção decide se o documento envelhece bem.

```
✓ LIÇÃO — regra imperativa com a razão embutida

  "Nunca pedir 'espaço vazio no topo' ao gerador de imagem: ele interpreta
   como tarja branca sólida. Descrever o ambiente que ocupa o topo (parede
   escura, céu nublado) em vez de pedir vazio."


✗ LOG — narrativa de incidente

  "Em 03/10 o usuário reclamou de uma tarja branca nas imagens 5 e 8. Depois
   de investigar com PIL, descobri 172px de faixa uniforme. Refiz os frames
   e o problema sumiu."
```

A lição serve na próxima vez. O log só serve para quem viveu aquele dia.

**Regras de redação:**
- uma regra por lição, não um parágrafo com cinco
- sem número de incidente, sem data, sem nome de quem reportou
- a razão vem junto: regra sem motivo é revogada na primeira dúvida
- comando exato quando houver — pseudocódigo não reproduz resultado

---

## Registrar o que funcionou, não o que deveria funcionar

Documentação oficial erra e envelhece. O procedimento guarda o que a máquina
aceitou de fato.

Exemplo real. A documentação de uma API de vídeo indicava:

```json
{ "image": { "inlineData": { ... } } }
```

A chamada falhava. O formato que a API realmente aceita:

```json
{ "image": { "bytesBase64Encoded": "...", "mimeType": "image/png" } }
```

Isso vale mais que a documentação, porque foi verificado. Vai para o
procedimento com o payload completo.

---

## Armadilhas também são conteúdo

A seção mais útil de qualquer procedimento maduro é a lista do que **não**
funciona:

| Armadilha | Sinal | Saída |
|---|---|---|
| Download que devolve HTML disfarçado | Dois arquivos diferentes com tamanho idêntico | Baixar do repositório canônico |
| Fonte variável no renderizador | Texto sai fantasma, quase invisível | Usar só TTF estático |
| Campo que a documentação promete | Ausente no schema real da integração | Inspecionar o schema antes de planejar |

Cada linha dessas custou uma investigação. Escrita, custa zero na próxima vez.

---

## Quando atualizar

- **Ao descobrir armadilha nova** — antes de terminar a tarefa, não depois
- **Ao receber correção do usuário** — a preferência dele é parte do
  procedimento, não detalhe da conversa
- **Ao achar o procedimento errado** — corrigir na hora; documento errado é pior
  que documento ausente

A atualização acontece **na mesma sessão** em que a lição apareceu. Postergar
significa perder: na sessão seguinte o contexto já foi.

---

## Memória permanente vs. procedimento

| | Camada permanente | Procedimento |
|---|---|---|
| Carregamento | toda sessão | quando o tipo de tarefa aparece |
| Orçamento | fixo e pequeno | sem limite prático |
| Conteúdo | fatos sobre o usuário e ambiente | como fazer, armadilhas, preferências |
| Forma | fato declarativo | regra imperativa com razão |

A pergunta que separa: **isso vale independente da tarefa?** Se a resposta
precisa de "quando eu estiver fazendo X", é procedimento.
