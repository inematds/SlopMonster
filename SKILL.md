---
name: slopmonster
description: Transforma texto escrito por IA em texto que uma pessoa publicaria. Aponta as marcas de IA, reescreve, limpa com um modelo rival, aponta de novo. Dispara em /slopmonster, "humaniza isso", "tira o slop disso", "isso parece IA?", "arruma esse texto", "humanize this", "de-slop this", "does this sound like AI", "fix this copy".
---

# SlopMonster

Pegue qualquer rascunho e faça ele parecer escrito por uma pessoa. Uma landing page, um
README, um e-mail, um roteiro. O alvo não é "passar num detector". Detector é ruído, e
correr atrás dele piora a prosa. O alvo é o instinto de um leitor que já viu mil parágrafos
de IA este mês.

O ciclo é sempre o mesmo, quatro passos, e o linter tem a primeira e a última palavra,
porque o linter é honesto e o modelo é persuasivo.

```
1. PONTUAR     python3 tools/deslop.py --text "…"    nota /5, sai em vermelho abaixo de 5
2. REESCREVER  três passadas, à mão ou por modelo     (ver abaixo)
3. LIMPAR      uma família de modelo DIFERENTE tira as marcas   tools/cleanse.sh
4. REPONTUAR   python3 tools/deslop.py de novo        só publica em 5/5
```

**O catálogo do linter é só em inglês.** Texto em português recebe 5/5 porque o linter não
lê a língua, não porque está limpo. Para texto em português, os passos 2 e 3 valem por
inteiro (as marcas têm equivalente direto em português), mas a nota dos passos 1 e 4 só
tem valor em texto em inglês.

## Passo 1: Pontuar

```bash
python3 tools/deslop.py pagina.html              # uma página pronta (pontua só o texto visível)
python3 tools/deslop.py pagina.html --view hero  # um elemento pelo id
python3 tools/deslop.py --text "cole um rascunho"
python3 tools/deslop.py pagina.html --allow-proof  # os números são reais e comprovados
```

Regex, sem opinião. Cinco grupos, um ponto cada: vocabulário de IA, construções de IA,
cadência de pontuação, ritmo de três, prova inventada. Abaixo de 5/5 ele sai com código
diferente de zero, então funciona como portão de build. "Quase limpo" é como uma página
acaba soando igual a todas as outras páginas de IA da internet.

Entrada vazia falha em vez de passar. Uma limpeza que estoura o tempo deixa um arquivo de
zero bytes, e um portão que carimba isso como LIMPO reporta slop como limpo exatamente
quando o pipeline quebrou.

Mexeu numa regex, rode `python3 tools/test_deslop.py`. O catálogo casa pela raiz da
palavra, e o atalho óbvio de stemming mata uma dúzia de palavras base em silêncio.

## Passo 2: Reescrever (três passadas)

Catálogo completo em `references/signs-of-ai-writing.md`. A versão curta:

1. **Mate o vocabulário.** `delve`, `seamless`, `robust`, `unlock`, `elevate`,
   `leverage`, `game-changing`, `journey`, `realm`… Troque por uma palavra mais simples, não
   por um sinônimo da mesma palavra.
2. **Mate as formas.** `not just X, but Y` é a marca mais barulhenta do inglês hoje. Também
   o reflexo do `rule-of-three`, o empilhamento de `em-dash`, pilhas de ressalva, parágrafos
   simétricos, o resumo de fechamento que ninguém pediu, e negrito abrindo cada bullet.
3. **Coloque uma pessoa de volta.** Tirar as marcas deixa um texto limpo e morto. Um número
   específico por afirmação. Tamanhos de frase que variam de verdade. Uma coisa que um autor
   cauteloso teria cortado. Uma aresta: uma contração, um fragmento, uma frase começando com
   "E".

## Passo 3: Limpar com um modelo rival

Um modelo é ruim em ouvir o próprio sotaque. Um modelo rival ouve na hora. Então a limpeza
roda numa **família de modelo diferente** da que escreveu o rascunho:

| Você está trabalhando em | Sotaque do rascunho | Limpar com |
|---|---|---|
| Claude Code / Claude | Anthropic | GPT-5.6 pela CLI codex. O `tools/cleanse.sh` faz isso |
| Codex / ChatGPT | OpenAI | Claude via `claude -p`, ou defina `DESLOP_WRITER=gpt` para o `cleanse.sh` |
| Gemini CLI | Google | Qualquer uma das CLIs. O `cleanse.sh` escolhe a que estiver instalada |
| Nenhuma CLI | n/a | O `cleanse.sh` imprime o prompt. Cole no chat da outra família |

```bash
tools/cleanse.sh rascunho.md > limpo.md                 # texto na saída, notas no stderr
tools/cleanse.sh rascunho.md > limpo.md 2> notas.txt    # guarda as notas também
```

A instrução que ele carrega (`prompts/cleanse.txt`): tire as marcas, mantenha cada fato,
mantenha o tamanho dentro de 10%, não invente nada.

Ele devolve **duas coisas**. O texto reescrito, depois uma linha `<<<SLOPMONSTER-NOTES>>>`,
depois até cinco bullets nomeando cada marca e sua correção. O script separa os dois, então
**stdout é texto e stderr é notas**, e o redirecionamento acima grava só prosa.

Leia as notas. A PASSADA 2 manda o modelo parar no último ponto real, então às vezes ele
apaga a sua linha de fechamento, e as notas são o único lugar onde ele avisa.

`WARNING no <<<SLOPMONSTER-NOTES>>> line` significa que o modelo ignorou o formato e a
resposta inteira veio como texto. Confira o final antes de publicar.

## Passo 4: Repontuar

Sempre. Um modelo de ponta é muito bom em tirar marcas e bem capaz de colocar outras novas
enquanto faz isso. Uma limpeza sem repontuar é cara ou coroa.

## A única regra dura

**Nunca inventar prova.** Nada de contagem de usuários, depoimentos, avaliações, nem
`trusted by 10,000 teams`, a menos que cada um seja verdade e você consiga mostrar. Se uma
afirmação precisa de um número que você não tem, escreva `[needs number]` e siga em frente.
O linter marca padrões de número mais substantivo de propósito: um falso positivo custa dez
segundos, um falso negativo é uma afirmação que você não sustenta. Especificidade ganha de
credibilidade emprestada de qualquer jeito.

## Formato de saída

Devolva primeiro o texto reescrito, inteiro. Depois uma lista curta `▎ o que mudou`, no
máximo cinco linhas, cada uma nomeando a marca e a correção. Nunca devolva só a análise.
Mantenha o sentido do autor exatamente: tirar o slop não é reescrever o argumento. Responda
na mesma língua e no mesmo registro do texto recebido.
