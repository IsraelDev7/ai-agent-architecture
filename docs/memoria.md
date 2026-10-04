# Memória: duas camadas e um orçamento

## O problema

Memória de agente falha de dois jeitos opostos.

**Sem memória:** cada sessão começa do zero. O usuário repete quem é, o que
prefere, como o ambiente está montado. Insustentável depois da terceira conversa.

**Com memória ilimitada:** tudo que já foi aprendido é injetado em toda sessão. O
contexto estoura antes da primeira pergunta e o custo por token cresce sem teto.

A saída não é escolher um extremo. É separar por **frequência de uso**.

---

## A separação

```
CAMADA PERMANENTE — orçamento fixo (~2.000 caracteres)
  Injetada em toda sessão, sem exceção.
  Só entra o que vale independente da tarefa.

  ✓ quem é o usuário (nome, papel, fuso, preferências estáveis)
  ✓ fatos de ambiente (stack, caminhos, versões)
  ✓ convenções permanentes sem outro lugar para morar

  ✗ progresso de tarefa
  ✗ procedimento (vai para a camada sob demanda)
  ✗ dado facilmente redescoberto


CAMADA SOB DEMANDA — sem limite prático
  Carregada só quando o tipo de tarefa aparece.

  ✓ procedimento ("como publicar um Reel")
  ✓ armadilhas daquele domínio
  ✓ preferências do usuário naquele tipo de trabalho
  ✓ comandos exatos que funcionaram
```

---

## Por que o orçamento fixo é a parte importante

Sem limite rígido, a camada permanente cresce monotonicamente. Nada nunca sai,
porque remover exige julgamento e adicionar não exige nada.

Com limite, **toda entrada nova disputa espaço com as antigas**. A disputa força
a pergunta que mantém o sistema saudável:

> Isso vale em toda sessão, ou só nesta tarefa?

Na prática, nove de dez candidatos a "memória permanente" são procedimento
disfarçado. O orçamento revela isso.

### Quando a camada enche

Não pule a gravação. Reescreva como **uma operação única** que remove ou
consolida o obsoleto e adiciona o novo junto. Operação atômica, limite conferido
só no resultado final — assim um lote consegue liberar espaço e gravar ao mesmo
tempo, mesmo quando a adição isolada estouraria.

---

## Regra de redação: fato, não ordem

Esta é a armadilha mais sutil do sistema inteiro.

```
✓ "Usuário prefere respostas concisas"
✗ "Sempre responda concisamente"
```

A segunda forma é relida em sessões futuras como **diretiva ativa**. O agente
passa a obedecer a própria anotação antiga em vez do pedido atual do usuário. Se
na próxima sessão a pessoa pedir uma análise longa e detalhada, a memória
imperativa trabalha contra ela.

Fato declarativo informa o julgamento. Ordem substitui o julgamento.

**Teste rápido:** se a entrada começa com verbo no imperativo, está errada.

---

## Horizonte de validade

| Dura | Onde vive |
|---|---|
| Uma sessão | Histórico da conversa |
| Uma semana | Histórico; não é memória |
| Um tipo de tarefa | Documento de procedimento |
| Toda sessão, sempre | Camada permanente |

Fato que fica obsoleto em uma semana nunca deveria ter entrado na camada
permanente. Ocupa orçamento e, pior, fica errado sem avisar.

---

## O que nunca entra

- **Progresso de tarefa.** "Terminei o passo 3" não sobrevive à sessão com
  utilidade.
- **Log de trabalho concluído.** Histórico já guarda isso, consultável.
- **Dado bruto.** Memória é índice, não armazenamento.
- **Informação trivial ou óbvia.**
- **Qualquer coisa redescoberta em um comando.** Versão de pacote se consulta;
  não se memoriza.

---

## Padrão de índice

Quando o volume cresce, a camada permanente vira **índice de onde as coisas
estão**, não o conteúdo:

```markdown
# Memória (índice)

## Onde está cada coisa
- knowledge/historico/   registros por data, guias técnicos
- knowledge/projetos/    estrutura e decisões por cliente
- knowledge/<persona>/   identidade visual travada

## Marcos
- 2026-02  início da operação
- 2026-09  migração de plataforma

## Regra
Aqui só entra o que é durável e curto. Detalhe vai para knowledge/.
```

Isso mantém o orçamento pequeno sem perder o acesso ao detalhe. O agente sabe
onde procurar; não carrega tudo por precaução.
