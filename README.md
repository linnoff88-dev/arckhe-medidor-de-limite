# medidor-de-limite

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="48" />
  <p><strong>Arckhé AI</strong></p>
</div>


[![Quem comeu seu limite](guia/assets/banner.jpg)](https://inematds.github.io/medidor-de-limite/guia/)

**ðŸ‡§ðŸ‡· [PortuguÃªs](README.md) Â· ðŸ‡ºðŸ‡¸ [English](README.en.md) Â· ðŸ‡ªðŸ‡¸ [EspaÃ±ol](README.es.md)**

> Seu limite acabou na quarta? Veja quem comeu.

## O que Ã©

Um medidor que lÃª os registros que o Claude Code e o Codex jÃ¡ gravam no seu computador e mostra para onde foi o limite da sua assinatura. Serve para quem roda muitas sessÃµes, robÃ´s e timers (`claude -p`, `codex exec`) e bate o limite da semana sem saber quem gastou. Ele mostra um ponteiro verde, amarelo ou vermelho, os projetos e rotinas que mais gastam e as execuÃ§Ãµes repetidas que dava para juntar. Precisa sÃ³ de Python 3; nÃ£o usa internet, chave nem API, e nÃ£o apaga nem pausa nada.

## ðŸ“– Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/medidor-de-limite/guia/**

## ComeÃ§o rÃ¡pido

```bash
git clone https://github.com/inematds/medidor-de-limite.git
cd medidor-de-limite
python3 medir.py relatorio
```

```bash
python3 medir.py calibrar --salvar
python3 medir.py html --saida medidor.html
```

## O que ele lÃª

| Ferramenta | Onde | O que soma |
|---|---|---|
| Claude Code | `~/.claude/projects/**/*.jsonl` (inclui subagentes) | `usage` de cada resposta: `input_tokens`, `cache_read_input_tokens`, `cache_creation_input_tokens`, `output_tokens`. A mesma resposta aparece em vÃ¡rias linhas: conta uma vez sÃ³ (pelo `message.id`). |
| Codex | `$CODEX_HOME/sessions` e `~/.codex*/sessions` (uma pasta por conta) | `total_token_usage` dos eventos `token_count`, em diferenÃ§a. No Codex o `input_tokens` jÃ¡ inclui o cache lido; o medidor separa. |

Agrupa por **dia**, **projeto** (a pasta onde a sessÃ£o rodou; a primeira pasta dentro de `~/projetos`, `~/projects`, `~/code`â€¦), **sessÃ£o** e **rotina**: sessÃµes sem pessoa no teclado (Claude `entrypoint=sdk-cli`, ou seja `claude -p` e bots pelo SDK; Codex `codex exec`) agrupadas pela primeira mensagem, com nÃºmeros e datas trocados por `#`. Rotina Ã© o que rodou 2 vezes ou mais na janela.

## Duas medidas, sem esconder nada

- **Tokens processados**: tudo o que passou pelo modelo (novos + cache lido + cache escrito + saÃ­da). Ã‰ o nÃºmero grande, e quase todo ele Ã© cache lido.
- **Peso**: os mesmos tokens com o peso que o preÃ§o pÃºblico dÃ¡ a cada tipo (Claude: novos 1, cache escrito 1,25, cache lido 0,1, saÃ­da 5; Codex: novos 1, cache lido 0,1, saÃ­da 8). O ponteiro usa o peso, porque ler cache custa bem menos que escrever resposta. Os pesos estÃ£o no config e vocÃª pode trocar.

## O teto Ã© seu, nÃ£o do plano

O plano **nÃ£o publica o limite em tokens**. Por isso o teto Ã© calibrado com os seus prÃ³prios registros:

- `python3 medir.py calibrar` procura nos registros a mensagem "You've hit your weekly limit Â· resets â€¦". Achou: soma o peso desde o inÃ­cio daquela semana (7 dias antes do reset) atÃ© o corte, e esse Ã© o teto. TambÃ©m guarda o dia e a hora do reset, para o ponteiro medir a semana corrente.
- NÃ£o achou: `python3 medir.py calibrar --estourou-em "2026-10-26 18:00" --salvar` (estimativa por cima: 7 dias antes do corte) ou `--teto N` Ã  mÃ£o.
- Uso no site, no app ou em outra mÃ¡quina conta no limite, mas nÃ£o estÃ¡ nos registros daqui. O teto calibrado jÃ¡ embute isso, desde que o seu jeito de usar nÃ£o mude muito.
- O ponteiro da semana precisa que a janela cubra desde o Ãºltimo reset: com `--dias` menor, ele avisa e volta para a projeÃ§Ã£o.
- O Codex quase nunca grava o corte nos registros; o teto dele costuma ficar Ã  mÃ£o.

O ponteiro fica **verde** abaixo de 70% do teto, **amarelo** de 70% a 90% e **vermelho** acima, sempre com o rÃ³tulo escrito.

## Repetidos

Duas ou mais sessÃµes de robÃ´ com o pedido inicial parecido (similaridade â‰¥ 0,85, ajustÃ¡vel) que comeÃ§aram **no mesmo dia, a menos de 60 minutos uma da outra**. Timer diÃ¡rio que roda uma vez por dia nÃ£o entra. Para cada grupo o medidor mostra duas economias:

- **pelo menos** (n âˆ’ 1) Ã— a menor execuÃ§Ã£o: o custo fixo de abrir cada sessÃ£o (instruÃ§Ãµes, ferramentas, contexto);
- **atÃ©** o total menos a maior execuÃ§Ã£o, se as execuÃ§Ãµes faziam o mesmo trabalho.

O medidor sÃ³ sugere. Quem decide juntar ou pausar o timer Ã© vocÃª (`systemctl --user list-timers`, `crontab -l`).

## Exemplo real (mÃ¡quina do INEMA, 08/11/2026)

Em `exemplos/`: `relatorio.txt`, `relatorio.json`, `medidor.html` e prints em 1440 px e 390 px. Ao vivo: https://inematds.github.io/medidor-de-limite/exemplos/medidor.html. Gerado com `--anonimo`: pastas de projetos privados viram `privado-N` e os pedidos delas nÃ£o aparecem.

- Claude Code: 4,3 bilhÃµes de tokens processados em 7 dias (96% cache lido), 1.059 sessÃµes, 999 de robÃ´s e timers.
- Codex: 189 milhÃµes de tokens, 1.039 sessÃµes, todas `codex exec`.
- Ponteiro do Claude: **amarelo, 81,9% do teto** na semana que comeÃ§ou segunda 21h; no ritmo dos dias com uso, a semana fecha em 103%.
- Uma sÃ³ pasta respondeu por 60% do peso dos 7 dias.
- 91 grupos de repetidos (699 execuÃ§Ãµes); juntar cada grupo economiza pelo menos 8,6% do peso.

## OpÃ§Ãµes

```bash
python3 medir.py relatorio --dias 14
python3 medir.py relatorio --json resumo.json
python3 medir.py html --anonimo --publicos publicos.txt --saida medidor.html
python3 medir.py relatorio --codex-home ~/.codex-2
python3 medir.py relatorio --janela 30 --similar 0.9
```

Config em `~/.config/medidor-de-limite/config.json` (tetos, pesos, `raizes` das pastas de projeto, `publicos`, `codex_homes`).

## O processo, do inÃ­cio ao fim

| # | Passo | Comando | Quem faz |
|---|---|---|---|
| 1 | Baixar | `git clone https://github.com/inematds/medidor-de-limite.git` | ðŸ§  vocÃª |
| 2 | Primeira leitura (7 dias) | `python3 medir.py relatorio` | âš™ï¸ script |
| 3 | Achar o teto na semana que estourou | `python3 medir.py calibrar --salvar` | âš™ï¸ script (vocÃª confere) |
| 4 | Ver o ponteiro e as barras | `python3 medir.py html` e abrir `medidor.html` | âš™ï¸ script |
| 5 | Ler o top 10 e os repetidos | â€” | ðŸ§  vocÃª |
| 6 | Juntar ou pausar timers repetidos | `systemctl --user list-timers` | ðŸ§  vocÃª (o medidor nunca mexe) |
| 7 | Repetir toda semana (opcional) | timer seu que roda o passo 4 | âš™ï¸ timer |

## Testes

```bash
python3 -m unittest discover tests
```

Com o pytest instalado (`pip install pytest`), `python3 -m pytest -q` tambÃ©m roda.

Sem rede, com registros JSONL sintÃ©ticos: soma de usage (sem duplicar e sem as mensagens de erro), Codex com cache dentro do input, rotina pela primeira mensagem, repetidos Ã— timer diÃ¡rio, projeÃ§Ã£o e cor do ponteiro, calibragem pela semana do corte.

## Privacidade

SÃ³ leitura. NÃ£o imprime o conteÃºdo das conversas: sÃ³ nÃºmeros e o inÃ­cio do pedido (atÃ© 80 caracteres, sem e-mail, URL nem caminho pessoal). Nada sai do seu computador.

---

Feito pelo [INEMA.CLUB](https://inema.club) Â· conteÃºdo aberto e gratuito Â· licenÃ§a MIT

---

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="32" />
  <p><strong>Arckhé AI</strong></p>
</div>