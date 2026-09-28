# SlopMonster

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

![Los cinco personajes de SlopMonster alineados, uno por cada regla que puntúa](docs/img/hero.png)

**Convierte texto escrito por IA en texto que una persona publicaría.**

## 📖 Guía de uso

Guía completa (landing + paso a paso): **https://inematds.github.io/SlopMonster/guia/es/**

El texto de IA tiene un olor. `delve`, `seamless`, `unlock`, `it's not just a tool, it's a
journey`. El lector ya lo nota, y una página con ese olor es una página en la que deja
de confiar.

La mayoría de las herramientas que corrigen esto nacen de la misma investigación pública. Esta
añade las dos cosas que las demás omiten.

**Le da una puntuación de 0 a 5 a tu texto y puede detener tu build.** Sin opiniones, sin
conjeturas, solo patrones. Los desarrolladores lo llaman linter. Cualquiera puede llamarlo un
verificador que no te deja publicar.

**Un modelo rival hace la limpieza.** Si Claude escribió el borrador, GPT lo limpia. Un modelo
no oye su propio acento, del mismo modo que tú no oyes el tuyo.

Funciona en landing pages, READMEs, correos electrónicos y guiones. Cualquier cosa que una persona vaya a
leer y juzgar.

## El ciclo

```
1. PUNTUAR    tools/deslop.py     da una puntuación de 0 a 5, sale en rojo por debajo de 5. Patrones, sin opiniones.
2. REESCRIBIR tres pasadas         elimina el vocabulario → elimina las formas → vuelve a poner a una persona
3. LIMPIAR    tools/cleanse.sh    una familia de modelos diferente quita las marcas que dejó la primera
4. VOLVER A PUNTUAR tools/deslop.py solo se publica con 5/5
```

El puntuador tiene la primera y la última palabra, porque el puntuador es honesto y el modelo es
persuasivo. «Casi limpio» es la forma en que una página termina sonando igual que todas las demás páginas
de IA de internet.

## Recibos, no promesas

Una ronda real con siete frases literales de la página principal de Jasper.ai (29 ago 2026):

| | puntuación |
|---|---|
| Su texto, tal como se descargó | **3/5**, con `unlock`, `empower` y cuatro listas de tres |
| Después de una pasada por este ciclo | **5/5**, sentido intacto, tamaño dentro de 10%, nada inventado |

Cada comando y su salida exacta: [`examples/jasper-live-run.md`](examples/jasper-live-run.md).
El texto de ellos se cita con fines de crítica y sigue siendo suyo. La licencia MIT de abajo
cubre el código y la prosa de este repositorio.

## Inicio rápido

```bash
git clone https://github.com/inematds/SlopMonster && cd SlopMonster

# puntuar cualquier cosa
python3 tools/deslop.py --text "It's not just a tool, it's a game-changing journey."
# → score 3/5, señala las dos marcas, sale con código 1

# puntuar una página terminada (lee solo lo que el visitante VE)
python3 tools/deslop.py index.html

# puntuar un archivo markdown (omite fragmentos de código, bloques cercados y texto tachado)
python3 tools/deslop.py README.md

# limpiar un borrador con un modelo rival y volver a puntuar
tools/cleanse.sh rascunho.md > limpo.md
python3 tools/deslop.py --text "$(cat limpo.md)"

# ¿tus cifras son reales y puedes demostrarlo? evita que la regla de prueba detenga el build
python3 tools/deslop.py index.html --allow-proof

# ¿tocaste una regex? esto detecta un catálogo que quedó medio ciego en silencio
python3 tools/test_deslop.py
```

**El catálogo solo está en inglés.** El texto en otro idioma, incluido el portugués, recibe 5/5
porque el puntuador no puede leerlo, no porque esté limpio. El paso de limpieza con un modelo
rival funciona en cualquier idioma, pero la puntuación solo vale para texto en inglés.

Sin dependencias. El puntuador es Python puro, biblioteca estándar. El script de limpieza necesita
una CLI de IA (`codex` o `claude`), o ninguna, en cuyo caso imprime el prompt
para que lo pegues. El archivo `.github/workflows/slop.yml` es la puerta del build, listo para
copiarlo en tu propio repositorio.

### Instalar como skill de agente

**Claude Code:** copia esta carpeta a `~/.claude/skills/slopmonster/` y escribe `/slopmonster`,
«quita el slop de esto» o «de-slop this». **Codex y otros agentes:** indica al agente que use el
`SKILL.md`. Son instrucciones en markdown puro, nada específico de Claude.

## La limpieza sabe qué modelo escribió

La regla: la limpieza se ejecuta con una **familia de modelos diferente** de la que escribió el borrador.
Las familias diferentes tienen acentos diferentes, y a un modelo le cuesta oír el suyo.

| Trabajas en | Acento del borrador | Lo que hace `cleanse.sh` |
|---|---|---|
| Claude Code | Anthropic | llama a **GPT-5.6** mediante tu CLI `codex`, con tiempo de espera y sandbox de solo lectura |
| Codex / ChatGPT | OpenAI | con `DESLOP_WRITER=gpt` llama a **Claude** mediante `claude -p` |
| Gemini CLI | Google | la CLI rival que esté instalada |
| ninguna CLI rival | n/a | imprime el prompt completo para que lo pegues en el chat de la otra familia |

Se niega a enviar un borrador de vuelta a su propia familia. Que el modelo corrija su propio examen es exactamente lo que este paso existe para evitar.

Después vuelve a puntuar, siempre: un modelo de primera es muy bueno para quitar marcas y también puede
añadir otras nuevas mientras lo hace.

## Qué detecta el puntuador

Cinco grupos. Si caes en uno, pierdes un punto. Por debajo de 5/5, el comando sale en rojo, así que
un build puede detenerse.

Cuatro grupos quitan el acento de IA. El quinto pregunta si la línea vende algo.

Cada antes y después de abajo es una línea real del sitio de Ridgeline Roofing. El texto tachado es lo que
decía el primer borrador. La negrita es lo que se publicó. El registro completo:
[`examples/ridgeline-roofing.md`](examples/ridgeline-roofing.md).

Cada ejemplo de esta página está en `formato de código` o tachado. No es decoración. Un
literal no es texto de venta, así que `deslop.py` omite ambos cuando lee un archivo `.md`.

![Regla 1, vocabulario de IA: delve, leverage, seamless, unlock](docs/img/rule-1-vocab.png)

**1. Vocabulario de IA.** Palabras que aparecen mucho más en texto de IA que en texto
humano.

Hay dos listas y funcionan de maneras distintas.

La primera lista busca por raíz. Así, `elevate` también detecta `elevates`, `elevated` y
`elevating`. Esto importa más de lo que parece. Las páginas de venta se escriben en tercera persona.
`Acme elevates your workflow` es la forma más común de la palabra, y la búsqueda exacta la pasaba
por alto.

La segunda lista contiene palabras que también tienen un sentido legítimo en el día a día. `crafted`,
`harness`, `landscape`, `journey`. Estas se detectan palabra por palabra. Así, «we craft
furniture by hand» sigue limpio, y solo se detecta el uso de marketing.

La corrección es una palabra más simple. No un sinónimo más elegante para la misma idea.

> ~~We leverage industry-leading materials to deliver unparalleled protection.~~
> **We source materials from manufacturers who test for wind, hail and sun.**
> `leverage`, `deliver`, `unparalleled`. Ninguna dice nada, y cada una cuesta una línea.

![Regla 2, construcciones de IA: "not just a tool, it's a journey"](docs/img/rule-2-phrases.png)

**2. Construcciones de IA.** Este grupo detecta formas de frase, no palabras aisladas.

Una forma es un molde que puedes llenar con cualquier cosa. `not just X, but Y` es la más
ruidosa del inglés actual. Cuando la ves, ya no puedes dejar de verla.

Hay 17 formas en la lista. Cosas como `that's where X comes in`, `say goodbye to` y
`whether you're X or Y`, acumulaciones de matices como `could potentially`, y preguntas que el
autor responde por su cuenta.

Se comprueban las formas corta y larga, «it's» e «it is». El registro formal no es un disfraz
ingenioso. Es el patrón de lo que escribe un modelo.

Las formas importan más que las palabras, porque una página puede pasar una prueba de vocabulario y
aun así parecer escrita por una máquina.

> ~~Not just a roof, but peace of mind.~~
> **A written scope and a fixed number before anyone climbs a ladder.**
> La forma promete una revelación y entrega una abstracción.

![Regla 3, cadencia de puntuación: dos guiones largos en una frase](docs/img/rule-3-punctuation.png)

**3. Cadencia de puntuación.** Dos guiones largos dentro de una misma frase.

Un guion largo en un párrafo es puntuación. Tres son un tic. Los modelos usan el guion largo a una tasa de
tres a cinco veces la de una persona.

La comprobación solo mira dentro de una ventana de 220 caracteres, y ese límite hace un trabajo real. El
texto de interfaz no tiene punto final. Los elementos de menú, botones y etiquetas se
encadenan, así que una división ingenua de frases trata toda la página como una sola frase y la
regla se activa en todas partes. Un puntuador que grita «que viene el lobo» se desactiva, así que la ventana se queda.

El punto y coma también cuenta, pero solo por encima de un umbral de tres, proporcional al tamaño de la
página. Dos puntos y coma en un documento técnico largo son estilo, no una marca.

> ~~Our team — trained, certified and local — is ready to help.~~
> **Thirty-eight on the crew, factory-trained for every material we install.**
> Los guiones largos ocultaban que la frase no tenía información.

![Regla 4, ritmo de tres: "faster, smarter, and better"](docs/img/rule-4-rhythm.png)

**4. Ritmo de tres.** Tres elementos seguidos. `faster, smarter, and better`.

Tres adjetivos son ritmo, no argumento. Un tricolon es retórica. Tres en una página son
una máquina. Tricolon es solo el nombre elegante de una lista de tres elementos.

Esta comprobación es estrecha a propósito, y solo dos formas la activan. Con la coma de Oxford,
necesita tres palabras sueltas. Sin ella, el tercer elemento tiene que ser una frase corta que
cierre la oración.

La precisión es el punto. «Inspection, repair and replacement for homes and commercial
buildings» son tres cosas reales que hace un techador, y pasa limpio. Marcarlo sería gritar «que viene el lobo», y la siguiente persona desactivaría el puntuador.

> ~~Trusted, reliable and built to last.~~
> **Six nails per shingle, every shingle.**
> Una especificación gana a tres adjetivos siempre. Nadie inventa una línea así,
> porque el texto inventado no sabe eso.

![Regla 5, ventas y marketing: la reescritura basada en Krug, Priestley y Hormozi](docs/img/rule-5-conversion.png)

**5. Ventas y marketing.** Las primeras cuatro reglas quitan el robot. Esta hace la
pregunta más difícil. ¿La línea vende algo?

El texto limpio que no dice nada sigue siendo una página muerta. Aquí funcionan dos cosas.

**La regla estricta: nunca inventar pruebas.** Nada de recuentos de clientes, testimonios o
reseñas que el negocio no haya conseguido. El puntuador marca cualquier número junto a un
sustantivo de personas, como `10,000+ happy users`. Se activa fácilmente a propósito. Una falsa alarma
cuesta diez segundos. Un error deja en tu sitio una afirmación que no puedes respaldar. Si
la cifra es real y puedes demostrarlo, `--allow-proof` la rebaja a advertencia e imprime los aciertos.

La prueba falsa es un fallo de ventas antes que un fallo de escritura. Nadie compra en una
página que descubrió mintiendo.

> ~~Loved by 10,000+ happy homeowners.~~
> **Project names and photography are placeholders. Swap in your own jobs before this goes live.**
> Di que el espacio está vacío. Suena a confianza, no a debilidad.

> ~~The area's most trusted roofing experts.~~
> **Roofing, and only roofing, since 2001.**
> «Más confiable» no se puede comprobar, así que el lector le resta valor. Una fecha no admite discusión.

**De dónde salen las líneas.** Cuatro fuentes. Si una frase no puede indicar su fuente, no
entra en la página.

1. **Lo que el oficio hace de verdad.** La fuente más sólida, por mucho. «Seis clavos por teja»
   es una especificación real con un modo de falla real detrás.
2. **Lo que el cliente ya teme.** Que el precio va a cambiar. Que el patio quedará destruido.
   Que le venderán un techo entero para un problema de tapajuntas.
3. **Lo que la competencia no dirá.** Una negativa llega más lejos que una promesa. «No hacemos
   superposiciones» te posiciona y descalifica al cliente equivocado en una sola línea.
4. **Las líneas que ya eran buenas.** «From first call to final nail» llegó escrita en el
   wireframe y superó todas las reescrituras. Se quedó.

**El trabajo citado detrás de la reescritura:**

| Quién | Qué propone |
|---|---|
| **Steve Krug**, *Don't Make Me Think* (2000) | cada línea que el lector tiene que descifrar es una línea que se salta |
| **Daniel Priestley**, orden del pitch | empieza por el problema y la idea clave, nunca por el producto |
| **Alex Hormozi**, el lado de la oferta | dolor identificado, especificidad comprobable, pruebas que realmente tienes |

Estos tres, junto con el referente de la categoría, se convierten en cinco principios de trabajo, cada uno con un
antes y un después reales: [`references/principles.md`](references/principles.md).

## Qué poner en lugar de las marcas

Que esté limpio no significa que sea bueno. Cinco principios deciden qué dice la línea en su lugar: el
*Don't Make Me Think* de Krug, el orden del pitch de Priestley, que empieza por el problema, y el
argumento de Hormozi, desde el lado de la oferta, de que la especificidad vence a los superlativos. Cada
uno con un antes y un después reales: [`references/principles.md`](references/principles.md).

## Un ejemplo completo

El sitio de Ridgeline Roofing: de un wireframe en Lorem ipsum a un sitio publicado, con el
antes → después de cada título, las seis marcas detectadas en los primeros borradores y la pasada de
verificar o señalar en cada cifra. Este es el archivo que enseña:
[`examples/ridgeline-roofing.md`](examples/ridgeline-roofing.md).

> ~~The area's most trusted roofing experts.~~
> **Roofing, and only roofing, since 2001.**
> «Más confiable» no se puede refutar, así que el lector le resta todo el valor. Una fecha no admite discusión.

## La única regla estricta

**Nunca inventar pruebas.** Nada de recuentos de usuarios, testimonios o reseñas que no hayas
conseguido. Si una afirmación necesita una cifra que no tienes, escribe
`[needs number]` y sigue adelante. El beneficio de una cifra inventada es menor que el beneficio de
una especificidad real, y es el único error irreversible.

Y esta skill nunca prometerá «engañar a los detectores de IA». Los detectores son ruido. El objetivo es
el instinto de un lector humano.

Sobre este archivo: `python3 tools/deslop.py README.md` da **5/5**, pero con una salvedad
honesta. Los ejemplos en inglés están marcados como literales, en `código` o tachados, y el
puntuador omite ambos. La prosa que los rodea está en portugués, y el catálogo no lee portugués.
La versión original en inglés de este README pasaba el propio puntuador sin nada suavizado.

## En qué se basa

Todas las fuentes están en [`references/sources.md`](references/sources.md):
*Signs of AI writing* de Wikipedia (WikiProject AI Cleanup) como catálogo canónico,
más cuatro humanizadores de código abierto con licencia MIT: `blader/humanizer`,
`harshaneel/humanize`, `lguz/humanize-writing-skill`, `haidrrrry/humanize-ai-writing`.
Los principios de reescritura vienen de Krug, Priestley y Hormozi. Los repositorios para evadir
detectores se omiten a propósito.

## Mapa del repositorio

```
SKILL.md                            la skill del agente, todo el ciclo como instrucciones
tools/deslop.py                     el puntuador. biblioteca estándar, sin deps, sale en rojo por debajo de 5/5
tools/test_deslop.py                suite de regresión. ejecútala después de tocar cualquier regex
tools/cleanse.sh                    limpieza con modelo rival, enrutada automáticamente, con tiempo de espera
.github/workflows/slop.yml          la puerta del build, lista para copiar
prompts/cleanse.txt                 la instrucción exacta que recibe el modelo de limpieza
references/signs-of-ai-writing.md   el catálogo completo: 2 niveles de vocabulario, 8 formas, cadencia, ritmo, pruebas
references/principles.md            los cinco principios de reescritura, cada uno con un par real
references/sources.md               cada fuente en la que se basa esto
examples/ridgeline-roofing.md       sitio completo, cada línea de antes → después
examples/jasper-live-run.md         ronda real sin editar: 3/5 → 5/5 en una página real
```

MIT. La misma que los humanizadores en los que se basa.
