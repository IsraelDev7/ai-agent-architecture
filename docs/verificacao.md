# Verificação: contra o sucesso presumido

## O padrão mais caro

O agente executa, não confere, e reporta sucesso. O usuário descobre o erro
depois — e a confiança no sistema não volta fácil.

Chamo de **sucesso presumido**: reportar o que a operação *deveria* ter feito em
vez do que ela *fez*.

É mais caro que falha explícita. Falha explícita é corrigida na hora. Sucesso
presumido se acumula silenciosamente até alguém descobrir que três relatórios
estavam errados.

---

## Quatro variações, todas vistas em produção

| Variação | Por que engana |
|---|---|
| Reportar a intenção do comando | A intenção não é o resultado |
| Confiar em `exit code 0` | Muita ferramenta sai 0 com falha parcial |
| Aceitar HTTP 200 | Página de rate limit devolve 200 com corpo plausível |
| Calcular em vez de medir | O cálculo ignora o que o sistema fez de fato |

### `fire and forget` depois de responder

Em ambiente serverless o container pode ser congelado antes das promessas
terminarem. A resposta sai, o trabalho não acontece, e nada registra o problema.

### `mode: 'no-cors'`

A resposta é **opaca por especificação**. O código não consegue saber se
funcionou. Qualquer tratamento de erro ali é teatro.

### `catch` marcando sucesso como reserva

O erro vira confirmação. É a inversão exata do que um `catch` existe para fazer.

### `console.log` no lugar de persistência

Log expira, não se consulta e não se agrupa. Não é registro, é esperança.

---

## O caso que ensinou a medir

Ao mixar trilha sonora sob um vídeo, apliquei a regra consagrada de produção de
áudio: **música 18 a 24 dB abaixo da voz**. Número correto, fonte confiável,
aplicado corretamente.

Medi o resultado:

```
antes da mixagem:  -26,1 LUFS
depois da mixagem: -26,1 LUFS
```

Idêntico. A música tinha ficado **inaudível**.

A regra pressupõe voz. O vídeo não tinha fala nenhuma — o "leito" não era
secundário, era o conteúdo principal. O cálculo estava certo e a **premissa**
estava errada.

Nenhuma inspeção do código pegaria isso. Só a medição.

A correção: trilha como leito principal, ambiente acima com ganho, e
normalização para o alvo da plataforma (`-14 LUFS`). Depois, medição de novo —
RMS por janela de um segundo, confirmando audibilidade ao longo de todo o clipe.

**Lição generalizada:** quando existe medida objetiva, medir não é opcional. O
cálculo teórico valida a fórmula, não o resultado.

---

## Protocolo

Antes de afirmar que algo funcionou:

1. **Existe medida objetiva?** Meça. Loudness, bytes, contagem de linhas, código
   de status, hash.
2. **O resultado é visual?** Renderize e inspecione com visão. Não confie no
   comando que gerou.
3. **A ferramenta pode falhar parcialmente?** Confira o conteúdo, não só o exit
   code.
4. **A resposta veio de rede?** Valide o corpo, não o status. Rate limit e
   interstício devolvem 200 com HTML plausível.
5. **Não deu para verificar?** Diga isso. "Executei mas não consegui confirmar"
   é uma resposta honesta e útil. "Pronto!" sem verificação, não.

---

## Validar corpo, não status

Rotina de recuperação de página bloqueada é o exemplo canônico. Três rotas que
devolvem HTTP 200 e **não** são a página:

- **Cache do Google** (morto desde 2024) devolve 200 e dezenas de KB — é um
  interstício de busca com redirecionamento em JS.
- **Caches AMP** devolvem um stub de ~300 bytes com meta-refresh apontando de
  volta para a URL bloqueada. Tratar como sucesso cria laço infinito.
- **Corpos de rate limit** têm vários KB de HTML. Checagem por tamanho passa.

Heurísticas que funcionam: piso de bytes por rota, detecção de meta-refresh cujo
destino é o host original, e títulos de interstício conhecidos.

---

## Verificação é barata; retratação não é

Medir custa um comando. Descobrir uma semana depois que o número estava errado
custa a confiança no sistema inteiro — e obriga a reconferir tudo que veio
daquela fonte.
