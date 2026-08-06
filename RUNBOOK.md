# Runbook — cómo está hecha esta guía y cómo se le añade un curso

[🧭 Inicio](https://hgomezgonzalez.github.io/) · [Ver la guía publicada](https://hgomezgonzalez.github.io/guia-devops/)

Manual de mantenimiento de `index.html`. Escrito después de construir el sistema y añadir once cursos con
él, así que documenta lo que funcionó y también lo que se rompió por el camino.

**Índice**

1. [Por qué un solo archivo](#1-por-qué-un-solo-archivo)
2. [Arquitectura del sistema](#2-arquitectura-del-sistema)
3. [Añadir un curso: los 10 puntos de registro](#3-añadir-un-curso-los-10-puntos-de-registro)
4. [Cómo se escribe un curso](#4-cómo-se-escribe-un-curso)
5. [Verificación](#5-verificación)
6. [Despliegue](#6-despliegue)
7. [Errores que ya se cometieron](#7-errores-que-ya-se-cometieron)

---

## 1. Por qué un solo archivo

Todo vive en un `index.html` de ~3.600 líneas: HTML, CSS y JS en línea. Sin build, sin dependencias, sin
`node_modules`.

Suena primitivo y es justamente la ventaja. El caso de uso real es **abrirlo en el móvil cinco minutos
antes de una entrevista**, a veces sin buena conexión. Un solo archivo:

- carga de una y funciona offline una vez cacheado,
- se edita sin arrancar nada,
- se despliega con `git push`,
- no puede romperse porque una dependencia cambió.

El precio es un archivo grande. Se paga con disciplina: **estructura repetida y previsible**, que es de lo
que trata el resto de este documento.

---

## 2. Arquitectura del sistema

### 2.1 Sistema de diseño en `:root`

Todo el color y la tipografía salen de variables CSS. Nunca se escribe un hex suelto en un componente.

```css
:root{
  --bg; --surface; --surface-2; --surface-3;     /* superficies */
  --ink; --ink-soft; --ink-faint;                /* tinta */
  --line; --brand; --brand-ink;                  /* estructura */
  --azure; --docker; --k8s; --good; --warn;      /* semánticos y por tecnología */
  --sans; --mono; --maxw:960px; --radius; --shadow;
}
```

Tres temas: `prefers-color-scheme: dark` para la preferencia del sistema, más overrides manuales
`[data-theme=light|dark]`. **Los componentes se estilan siempre a través de los tokens**, nunca dentro de
la media query — si no, el tema manual no puede ganarle a la del sistema.

### 2.2 Navegación en cuatro capas

| Capa | Qué es |
|---|---|
| `nav.nav` | Barra fija: inicio, marca, acceso a chuleta, botón Enfoque, botón Menú |
| `#menu` | Overlay con las secciones agrupadas en 9 categorías, con buscador propio |
| `#dashboard` | La portada: los cursos por niveles, con buscador |
| `.seq-nav` | Anterior/Siguiente inyectados por JS al final de cada curso |

### 2.3 El dashboard y su buscador

Los cursos se agrupan en niveles que van de fundamentos a liderazgo:

```
Nivel 1  Fundamentos          Nivel 4  Infraestructura & Cloud
Nivel 2  Desarrollo           Nivel 5  Entrega (DevOps)
Nivel 3  APIs, integración    Nivel 6  Liderazgo & gestión
         & seguridad          Repaso · Assessments
```

Cada curso es una tarjeta:

```html
<a class="dash-card" data-k="palabras clave sin tildes" href="#id" style="--c:#COLOR">
  <span class="dc-n">12</span><b>Título</b><span>Subtítulo</span>
</a>
```

**El atributo `data-k` es el corazón del buscador.** Es un índice de sinónimos y jerga que permite
encontrar el curso por tema y no solo por nombre. Ejemplos reales del archivo:

```
kubernetes → "k8s pod pods orquestacion deployment service ingress cluster replicas escalar"
.NET       → "net dotnet c csharp asp aspnet ef core efcore entity framework linq nuget kestrel"
arquitectura → "hexagonal puertos adaptadores ddd clean onion cqrs strangler fig adr c4 bounded context"
```

Sin `data-k`, buscar "hexagonal" no encontraría nada, porque el curso se llama "Arquitectura de software".

El buscador normaliza con `NFD` y quita diacríticos, exige **todos los términos en cualquier orden**
(`"ef core"`, `"base datos"`), resalta la coincidencia con `<mark>`, oculta los niveles que quedan vacíos,
y **Enter salta al primer resultado**.

### 2.4 Modo Enfoque

`body.focus-mode` + el selector `:target` en CSS muestra **un solo curso a la vez**. Apagado, la página
vuelve a ser un documento continuo. Se persiste en `localStorage`. Sirve para estudiar sin distracción y
para que el móvil no tenga que renderizar 3.600 líneas.

### 2.5 Simulador

Mini-app JS sin backend. El banco es un array `Q` donde cada pregunta tiene esquema fijo:

```javascript
{
  cat:'Arquitectura',                    // categoría visible
  q:'...',                               // la pregunta
  hint:'...',                            // pista de por dónde empezar
  good:'...',                            // qué suma en la respuesta
  watch:'...',                           // qué te van a cuestionar
  level:'...',                           // cómo subir el nivel
  model:'...',                           // respuesta modelo
  sets:['all','tech','sysdesign'],       // para filtrar por tipo de práctica
  time:120                               // segundos sugeridos
}
```

Flujo: elegir set → pregunta con **cronómetro corriendo** y textarea → "Ver evaluación" revela la rúbrica
de tres criterios (`good` / `watch` / `level`) y la respuesta modelo → siguiente.

Sets disponibles: `all`, `quick` (corta a 5), `tech`, `sysdesign`, `lead`.

### 2.6 Los bloques de práctica

- **`#chuleta`** — tarjeta de bolsillo: pitch de 20 s y las métricas que siempre se mencionan.
- **`#guion`** — todos los "cómo lo cuentas tú" reunidos en un solo lugar.
- **`#entrenamiento`** — banco de preguntas en `<details class="qa">`, agrupado por bloques temáticos.
- **`#glosario`** — cada término del CV explicado en una frase. Su propósito: que ninguna palabra escrita
  en el CV pueda tomarte por sorpresa.
- **Assessments** — System Design, prueba de código, psicométricos y assessment center.

---

## 3. Añadir un curso: los 10 puntos de registro

Un curso no es solo una `<section>`. Hay **diez sitios** que tocar, y omitir uno deja el sistema
inconsistente de formas que no siempre saltan a la vista.

### 1 · Clase de color del tag

Junto a las demás `.t-*` (≈ línea 91):

```css
.t-gcp{background:#ea4335}
```

Elige un color que no choque con los vecinos de su nivel. Ejemplo real: GCP entró al Nivel 4, cuyos
vecinos eran azules y verde azulado, así que se le dio el rojo de Google.

### 2 · La sección del curso

Se inserta **en el orden lógico de la ruta**, no al final del archivo. Si el curso va después de Azure, la
sección va físicamente entre `</section>` de `#azure` y `<section id="pipelines">`.

### 3 · Entrada en el menú

En la categoría que corresponda dentro de `.menu-groups`:

```html
<a href="#gcp"><b>GCP / Google Cloud</b><span>Traducido desde Azure y AKS</span></a>
```

Si el tema no encaja en ninguna categoría, se crea una nueva con su punto de color.

### 4 · Tarjeta del dashboard

Cerrando el nivel que le toque, **con su `data-k`** (ver §2.3). Este es el punto que más se olvida y el
que rompe el buscador en silencio.

### 5 · Renumerar las tarjetas siguientes

Los `<span class="dc-n">N</span>` están escritos a mano. Si insertas en medio, hay que correr todos los
números posteriores. **Hazlo de atrás hacia adelante** para no colisionar:

```
23→24, 22→23, 21→22 …
```

### 6 · Array `SEQ`

Alimenta los botones Anterior/Siguiente. Se inserta en la posición del recorrido:

```javascript
['azure','Azure / AKS'],['gcp','GCP / Google Cloud'],['curso-devops','Curso DevOps'],
```

### 7 · Contador del buscador — **en DOS lugares**

Este es *el* punto que más fácil se escapa, porque el texto está duplicado:

```html
<p class="dash-count" id="dash-count">24 cursos · busca por tema: …</p>
```
```javascript
IDLE='24 cursos · busca por tema: …';
```

Si actualizas solo uno, el contador cambia al escribir y al borrar vuelve al número viejo.

### 8 · Preguntas del simulador

Dos por curso suele bastar. Se añaden al array `Q` con su esquema completo, y **hay que actualizar el
contador del botón**:

```html
<button class="sim-set" data-set="all">Completo (22)</button>
```

Ese número también está escrito a mano. Cuéntalo así:

```bash
grep -c "sets:\['all'" index.html
```

### 9 · Bloque en el banco de entrenamiento

Un `<h3>` con el tema y 4–6 `<details class="qa">` dentro de `#entrenamiento`.

### 10 · Sellos de despliegue

Dos, con la hora real de Bogotá: `#deploy-stamp` en el dashboard y `.deploy-stamp` en el pie.

```bash
TZ=America/Bogota date '+%Y-%m-%d %H:%M'
```

---

## 4. Cómo se escribe un curso

La estructura es **siempre la misma**. La repetición es deliberada: cuando todos los cursos se leen igual,
estudiar es más rápido y se nota enseguida lo que falta.

```
h2 con tag       → título del curso
p.sub            → por qué existe esta pieza y qué gap llena
.callout         → ANALOGÍA cotidiana
.pipe-scroll     → diagrama del recorrido (opcional pero muy útil)
.grid.cols-3     → los conceptos que debes saber explicar
<pre><code>      → el código, capa por capa
.cmp-wrap+table  → tabla comparativa (cuando aplica)
.callout         → el dato que demuestra profundidad
.callout         → "el círculo completo": cómo conecta con el resto de la guía
.youuse          → CÓMO LO CUENTAS TÚ
<details.qa>     → 5–8 preguntas típicas
```

### La analogía

Cada curso abre con una metáfora cotidiana. Es lo que te salva cuando te piden explicárselo a alguien de
negocio. Ejemplos del archivo:

| Curso | Analogía |
|---|---|
| Docker | Una lonchera sellada: la abras donde la abras, adentro está lo mismo |
| Arquitectura hexagonal | Una torre de control: no cambia aunque cambien las aerolíneas |
| GCP | Cambiar de aeropuerto, no de medio de transporte |
| .NET | El Spring Boot de Microsoft |

### La frase con métrica

**Todo curso cierra en un `.youuse`**: una frase en primera persona, lista para decir, que termina en una
cifra.

> *"Diseño con el dominio aislado: arquitectura hexagonal […] El resultado fue 500 ms bajo alto tráfico,
> cero downtime y un 60% menos de tiempo de despliegue."*

Si no puedes escribir esa frase para un curso, o no dominas el tema o no tienes experiencia real en él —y
ambas cosas son información útil.

### El guion honesto

Para tecnologías **no operadas en producción**, el `.youuse` debe decirlo con todas las letras. Es una
regla dura del proyecto, no una preferencia de estilo.

Los cursos de **React vs Angular** y **GCP** son los ejemplos: ambos reconocen explícitamente que esa
tecnología no se ha usado en un proyecto real, y convierten la respuesta en fortaleza mostrando el mapeo
con lo que sí se domina.

> Inventar experiencia se detecta en dos preguntas: basta con que pregunten por la consola, por una cuota
> o por un error típico.

### La técnica de homologación

Cuando el curso trata un stack ajeno pero análogo a uno conocido, el formato que funciona es:

1. Presentar cada concepto **junto a su equivalente** conocido.
2. Aislar las **3 o 4 diferencias reales** (de concepto, no de nombre).
3. Un **ejemplo lado a lado**: los mismos comandos en ambos, con el equivalente comentado en la línea.
4. Una **tabla maestra** de equivalencias, pensada para memorizar.

El curso de GCP es el ejemplo completo: 24 filas de equivalencias contra Azure y un despliegue lado a lado
`gcloud` / `az` que termina en el mismo `kubectl apply`, para mostrar qué es realmente portable.

### Componentes disponibles

No inventes CSS nuevo. Lo que ya existe:

| Clase | Para qué |
|---|---|
| `.callout` | Bloque destacado (analogía, dato de entrevista, círculo completo) |
| `.youuse` | La frase en primera persona, con borde verde |
| `.grid.cols-2` / `.cols-3` | Rejilla de `.card` |
| `.card` + `p.obj` | Tarjeta de concepto con su rol en una frase |
| `.pipe-scroll` + `.pipe` + `.step` | Diagrama de flujo horizontal, con scroll en móvil |
| `.cmp-wrap` + `table.cmp` | Tabla comparativa que desplaza dentro de su contenedor |
| `<pre><code>` + `.kw .str .cmt .num` | Código con resaltado manual |
| `<details class="qa">` | Pregunta plegable con respuesta |

---

## 5. Verificación

Nueve pasos. Los primeros cuatro atrapan casi todo.

```bash
# servir con cache-busting (el navegador cachea index.html con ganas)
python3 -m http.server 8000
# abrir http://localhost:8000/index.html?v=1
```

**1 · Estructura HTML** — anidamiento y etiquetas sin cerrar:

```bash
python3 - <<'PY'
from html.parser import HTMLParser
VOID={'br','hr','img','input','meta','link','source','area','base','col','embed','param','track','wbr'}
h=open('index.html',encoding='utf-8').read()
class P(HTMLParser):
    def __init__(s): super().__init__(convert_charrefs=False); s.st=[]; s.err=[]
    def handle_starttag(s,t,a):
        if t not in VOID: s.st.append((t,s.getpos()))
    def handle_endtag(s,t):
        if t in VOID: return
        if not s.st: s.err.append(f"sobra </{t}> l{s.getpos()[0]}"); return
        if s.st[-1][0]!=t:
            s.err.append(f"esperaba </{s.st[-1][0]}> l{s.st[-1][1][0]}, llegó </{t}> l{s.getpos()[0]}")
            for i in range(len(s.st)-1,-1,-1):
                if s.st[i][0]==t: del s.st[i:]; return
        else: s.st.pop()
p=P(); p.feed(h)
print("errores:",len(p.err),"· sin cerrar:",[t for t,_ in p.st])
for e in p.err[:6]: print("  ",e)
PY
```

**2 · JavaScript** — una coma mal puesta en `SEQ` o en `Q` rompe navegación, buscador y simulador a la vez:

```bash
python3 -c "
import re
h=open('index.html',encoding='utf-8').read()
for i,s in enumerate(re.findall(r'<script>(.*?)</script>',h,re.S)):
    open(f'/tmp/_s{i}.js','w').write(s)
" && for f in /tmp/_s*.js; do node --check "$f" && echo "OK $f"; done
```

**3 · Numeración del dashboard** — sin saltos ni repetidos:

```bash
grep -o 'href="#[a-z0-9-]*"[^>]*><span class="dc-n">[^<]*' index.html \
  | sed 's/href="#//;s/"[^>]*><span class="dc-n">/  →  /'
```

**4 · Contador del buscador** — que no queden dos números distintos:

```bash
grep -c "24 cursos" index.html      # debe dar 2
grep -c "23 cursos" index.html      # debe dar 0
```

**5 · Buscador** — probar que `data-k` funciona, en la consola del navegador:

```javascript
const i=document.getElementById('dashfilter');
['gcp','hexagonal','oauth','pods','ef core'].forEach(q=>{
  i.value=q; i.dispatchEvent(new Event('input'));
  console.log(q, [...document.querySelectorAll('#dashboard .dash-card:not([hidden])')]
    .map(c=>c.getAttribute('href')));
});
```

**6 · Navegación secuencial** — que el curso anterior apunte al nuevo y el nuevo al siguiente.

**7 · Modo Enfoque** — con el curso activo, debe mostrarse solo esa sección.

**8 · Simulador** — el set Completo trae el total nuevo y las preguntas muestran rúbrica y modelo.

**9 · Móvil y consola** — sin errores en consola y **sin scroll horizontal**: las tablas deben desplazarse
dentro de su `.cmp-wrap`, no arrastrar la página.

---

## 6. Despliegue

```bash
TZ=America/Bogota date '+%Y-%m-%d %H:%M'     # actualizar los dos sellos primero

git add index.html
git commit -m "feat: curso de <tema> en la guía de entrevistas"
git push origin main
```

GitHub Pages despliega solo. Tarda 1–3 minutos y **el navegador cachea**, así que la comprobación se hace
con polling y cache-busting:

```bash
for i in $(seq 1 30); do
  if curl -s -H 'Cache-Control: no-cache' "https://<usuario>.github.io/guia-devops/?v=$i" \
     | grep -q 'section id="<nuevo-id>"'; then echo "publicado"; break; fi
  sleep 10
done
```

Convención de commits: `feat: curso de <tema> en la guía de entrevistas`.

---

## 7. Errores que ya se cometieron

Esta sección vale más que el resto. Todo lo que sigue pasó de verdad.

**El contador del buscador en un solo sitio.** El texto está en el HTML y en la constante `IDLE` del JS.
Al actualizar solo el HTML, el contador se "arreglaba" al escribir y volvía al número viejo al borrar.

**Renumerar de adelante hacia atrás.** Al correr 18→19 antes que 19→20, se pisan los números. Siempre de
atrás hacia adelante.

**El botón Menú fuera de pantalla en móvil.** La barra tenía `overflow-x:auto` y el título completo
empujaba el botón de Menú fuera del viewport: había que arrastrar la barra para abrir el menú, cosa que
nadie descubre. Se arregló recortando el título con `text-overflow:ellipsis` y `overflow-x:hidden`. **Cada
vez que se añade algo a la nav hay que volver a comprobarlo** — pasó otra vez al añadir el enlace de
Inicio.

**Olvidar `data-k`.** El curso aparece en el dashboard pero es invisible para el buscador. No da error, y
solo se descubre buscando el tema a mano.

**Sobremuestrear en las verificaciones visuales.** Al revisar detalle fino, ampliar una imagen genera
artefactos que parecen defectos del contenido. Verifica a resolución nativa.

**Confiar en la recarga normal del navegador.** Al probar tras publicar, sin `?v=algo` se ve la versión
cacheada y parece que el despliegue falló.

**Añadir el curso al final del archivo.** Funciona, pero rompe el orden lógico de lectura y deja la
`seq-nav` incoherente con el dashboard.

---

## Añadir un curso — resumen ejecutable

```
[ ] 1.  Clase .t-xxx con color que no choque en su nivel
[ ] 2.  <section id="..."> en el orden lógico de la ruta
[ ] 3.  Entrada en el menú, en su categoría
[ ] 4.  Tarjeta del dashboard CON data-k
[ ] 5.  Renumerar las siguientes (de atrás hacia adelante)
[ ] 6.  Insertar en el array SEQ
[ ] 7.  Contador "N cursos" en HTML **y** en IDLE
[ ] 8.  2 preguntas en Q + actualizar "Completo (N)"
[ ] 9.  Bloque en #entrenamiento
[ ] 10. Los dos sellos de despliegue

[ ] Verificar: HTML · JS · numeración · buscador · seq-nav · enfoque · simulador · móvil
[ ] Publicar y confirmar en vivo con cache-busting
```
