> EDICIÓN LIGERA: lee primero EMPIEZA_AQUI.txt. La base se descarga y construye durante la instalación; no viene en este ZIP.

# Oráculo Matemático Universal — versión 1

Aplicación de consola local para Python 3.11 o posterior. No predice sucesos.
La base incluida contiene vocabulario real de Open Multilingual Wordnet 1.4 y
extracciones Kaikki de Wikcionario español y Simple English Wiktionary.
La base entregada contiene **2.047.769 entradas grafía–idioma**, **1.836.771
grafías distintas** y **887 códigos de idioma/variedad**. Por defecto son elegibles
1.818.624 entradas sin espacios ASCII ni guiones bajos. No equivale a cobertura
completa de 887 idiomas. Recuentos medidos en `informe_verificacion.json`.

## Windows: instalación exacta

1. Descarga y extrae **todo** el ZIP en una carpeta de tu usuario, por ejemplo
   `Documentos\Oraculo`. No ejecutes desde dentro del ZIP.
2. Instala Python 3.11 o posterior de https://www.python.org/downloads/windows/.
   Activa el Python Launcher (`py`). Se recomienda Python 3.12 de 64 bits.
3. Abre `INSTALAR_WINDOWS.bat`. Necesita internet una vez para instalar las
   dependencias y el paquete de efemérides (aproximadamente 17 MB más dependencias).
4. Cuando termine, abre `INICIAR_WINDOWS.bat`.
5. Introduce tu nombre y nacimiento con formato `2000-03-20`.
   La hora natal es opcional. Si la conoces, indica zona IANA, por ejemplo
   `America/Mexico_City`; el programa aplica el horario histórico de esa zona.
6. Si omites hora natal, deja vacío el instante astronómico para usar el momento
   actual, identificado expresamente como instantánea de generación, no carta natal.
7. Elige A, B o C, repetición e historial. Cada Enter vacío devuelve una entrada.

Después de instalar, el programa genera palabras sin conexión. No envía nombre,
fecha, lugar ni historial. El corpus viene incluido. Usa Windows Terminal y una
fuente con cobertura Unicode si algunos caracteres aparecen como cuadros.

No se compiló ni probó un EXE ni se ejecutaron los BAT en Windows. Las pruebas
adjuntas se ejecutaron en Linux. La compatibilidad Windows se basa en Python,
SQLite y dependencias multiplataforma; requiere verificación final en tu equipo.

## Comandos

- `/idiomas`: códigos y recuentos. `/idioma spa` o `/idioma spa,eng,jpn`.
- `/idioma todos`: todos los idiomas. Algunos códigos de Wiktionary no son ISO.
- `/categoria n,v`: sustantivos y verbos según categoría principal.
  `/categoria todos`: elimina ese filtro. `a` adjetivo, `r` adverbio.
- `/tipo proper`: nombres propios marcados por Kaikki. `/tipo todos`.
  `unknown` no significa nombre común: OMW no distingue sistemáticamente nombres.
- `/frases si` incluye unidades de varias palabras; por defecto se excluyen las
  entradas con espacios ASCII o guiones bajos. No es segmentación universal.
- `/modo A`, `/modo B`, `/modo C`: cambia el motor. C solicita semilla.
- `/repeticion no`: comienza un nuevo ciclo sin repetición.
  `/repeticion si`: nuevo ciclo con repetición permitida.
- `/reiniciar`: reinicia explícitamente el ciclo y el contador del generador C.
- `/info`, `/perfil`, `/ultimo`: estadísticas, fórmulas de entrada y procedencia.
- `/historial`: últimas 20 generaciones guardadas en esta sesión.
- `/exportar csv` o `/exportar json`: exporta la sesión con historial activado.
- `/salir`: cierra. Ctrl+C también funciona.

El historial está desactivado por defecto. Al activarlo, se guarda JSONL local en
`historial/`. Incluye huella, letras latinas normalizadas del nombre y astronomía;
**no es anonimización**. No compartas esos archivos si contienen información que
quieres mantener privada. Borrar esa carpeta elimina los historiales guardados.
Sin historial no se conservan las salidas al cerrar; `/ultimo` existe en memoria.

## Matemática auditable

Una entrada única es `(grafía NFC, código de idioma)`, no un significado.
Se conservan mayúsculas, acentos y alfabetos. No se aplica casefold a las palabras.
Los sentidos se deduplican; `provenance` conserva fuente y etiquetas originales.
Una grafía con varias categorías tiene como principal el mínimo lexicográfico
entre sus códigos POS. Esto es determinista pero simplifica la polisemia:
los filtros POS usan esa categoría principal, no todas sus posibles funciones.

La base tiene índices por idioma/categoría/tipo, grafía e identificador.
Al iniciar/cambiar filtros se construye una tabla temporal con posiciones densas
por grupo `idioma|POS`. Ese paso es O(N); cada Enter no recorre el diccionario.
La tabla temporal puede usar disco. Se necesita espacio libre para ella.

A: se elige grupo g con masa n_g y luego posición uniforme en [0,n_g).
Por tanto P(entrada)=1/N. No significa igual probabilidad por idioma ni por grafía
compartida entre idiomas. Se usa `secrets.randbelow`, que evita el sesgo de módulo.

B: perfil P = JSON canónico ordenado, UTF-8, de numerología, huella y astronomía.
Para cada grupo g:

    b_g = 1 + int_big_endian(SHA256(JSON(P) + '|' + g)[0:8]) mod 100
    w_g = b_g * multiplicador_simbolico(signo_solar, POS_g)
    Z = suma_g(n_g * w_g)
    P(grupo g) = n_g * w_g / Z
    P(entrada de g) = w_g / Z

Los multiplicadores están en `reglas_simbolicas.json`, editables entre 1 y 1000.
Por defecto: signos de fuego duplican verbos, tierra sustantivos, aire adverbios
y agua adjetivos; es una asociación artística arbitraria, no una ley astrológica.
Los grupos sin regla multiplican por 1. Sin astronomía válida todos multiplican
por 1. El archivo de reglas también forma parte del perfil auditado.

El módulo 100 es una **regla deliberada de pesos**, no el muestreador aleatorio.
No se afirma que esos pesos sean uniformes ni que tengan significado predictivo.
El hash transforma datos personales/astronómicos en una distribución simbólica;
no aporta azar. El azar nuevo procede de `secrets`. El perfil astronómico se
congela al iniciar: no se recalcula en cada Enter. Cambiarlo requiere nueva sesión.
Los numeradores y denominadores exactos quedan en `/ultimo` y el historial.

C: distribución uniforme como A; generador determinista especificado:
K=SHA256(semilla UTF-8); bloques HMAC-SHA256(K, contador de 16 bytes big-endian),
contador desde 0. Para randbelow(n), se toman k=ceil(bit_length(n)/8) bytes de
bloques nuevos; se rechaza x >= 256^k-(256^k mod n), y se devuelve x mod n.
El historial incluye versión de algoritmo, hash de base, filtros y semilla de C.
Las mismas versiones, base (incluidos IDs), filtros, semilla, comandos y política
de repetición reproducen la secuencia de palabras. Las fechas del historial no
son reproducibles. C no depende del perfil ni se presenta como impredecible.
Cambiar de modo consume una secuencia distinta; `/reiniciar` reinicia C.

Sin repetición: después de elegir una entrada, se eliminan TODAS las entradas
con la misma grafía NFC en los idiomas elegibles. Se intercambia la última
posición del grupo con la retirada y se reduce el tamaño. La probabilidad
registrada corresponde a la entrada antes de retirarla; la probabilidad de una
**grafía** homógrafa es la suma de sus entradas elegibles. Al agotarse, se detiene;
no reinicia en secreto. Cambiar filtros conserva el conjunto de grafías vistas.
Cambiar la política de repetición o `/reiniciar` inicia un ciclo nuevo explícito.

Numerología: NFKD y mayúsculas; se usan solo A–Z, A=1… I=9, J=1…, suma y reducción
repetida de dígitos hasta 0–9. Se muestra qué letras se usaron. No se conservan
11/22/33 como números maestros. Nombres sin letras latinas: número `null`, no se
inventa transliteración; la huella sí incluye el nombre Unicode completo NFC.
Camino de vida: suma de dígitos AAAAMMDD y reducción. Son convenciones simbólicas.
Huella: SHA256(JSON canónico [nombre NFC recortado, fecha ISO]). No es una clave
secreta ni vuelve anónimos los datos.

## Astronomía y límites

Skyfield + JPL DE421: posiciones aparentes geocéntricas en eclíptica/equinoccio
verdaderos de la fecha, longitud, latitud y distancia; Sol, Luna, Mercurio, Venus,
Marte y baricentros de Júpiter/Saturno/Urano/Neptuno. Fase lunar = diferencia de
longitudes eclípticas; iluminación calculada geométricamente por Skyfield.
No se usa el mes sinódico promedio del prototipo como efeméride.

DE421 cubre aproximadamente 1900–2053; fuera de su intervalo se informa que la
astronomía no está disponible. No se inventan posiciones. El paquete utiliza
escalas temporales incorporadas; futuras revisiones de segundos intercalares
requieren actualizar dependencias. No se ofrece precisión observacional absoluta.

Simbolismo astrológico implementado: índice tropical floor(longitud solar/30),
0 Aries, 1 Tauro, …, 11 Piscis, incluido en P. No equivale a las constelaciones
astronómicas. No se calculan casas, ascendente ni una carta natal completa.
El lugar es solo una etiqueta local: posiciones geocéntricas no dependen de él.
Hora inexistente por DST se rechaza; para hora duplicada se elige fold 0 o 1.
Las asociaciones de B son convencionales; no hay influencia humana demostrada.

## Ampliar la base

Cierra la aplicación y haz copia de `data/lexico.sqlite` antes de importar.
Descarga JSONL/JSONL.GZ de https://kaikki.org/dictionary/rawdata.html después de
revisar tamaño y licencia. En una terminal abierta dentro de esta carpeta:

    .venv\Scripts\python.exe importar.py kaikki archivo.jsonl.gz --url URL_ORIGINAL --license LICENCIA_VERIFICADA

Para OMW 1.4:

    .venv\Scripts\python.exe importar.py omw omw-1.4.zip

No se extraen scripts ni se ejecuta contenido descargado. El importador Kaikki
lee registros línea a línea; no carga todo el archivo. Importa `word` de entradas,
no todas las flexiones enumeradas en `forms`; muchas flexiones ya tienen entrada
propia en Wiktionary. No genera palabras nuevas. Usa pycountry para unificar códigos
ISO de dos letras con tres letras cuando existe correspondencia exacta.

El importador conserva fuente/hash/licencia y permite reimportación idempotente.
Si se interrumpe, pueden quedar lotes completos; reimportar el mismo archivo los
completa sin duplicarlos. No importes con otra instancia abierta. No existe
actualización destructiva automática ni borrado de vocabulario antiguo.

Limitaciones pendientes: cobertura universal imposible, todas las disciplinas
no verificadas, nombres geográficos/históricos no clasificados sistemáticamente,
clasificación temática universal no implementada.
La versión acepta más corpus pero no afirma haber integrado todos los disponibles.

## Pruebas

    .venv\Scripts\python.exe -m unittest -v test_motor test_integracion
    .venv\Scripts\python.exe verificar.py

`pruebas.txt` contiene el resultado real de tests. `informe_verificacion.json`
contiene integridad SQLite, conteos, SHA256, latencia, 10.000 extracciones sin
repetición y prueba diagnóstica de frecuencias. Una prueba estadística no demuestra
seguridad criptográfica; esta depende del generador del sistema y su implementación.

Código nuevo: licencia MIT, véase `LICENCIA_CODIGO.txt`. Los corpus tienen sus
propias licencias y atribuciones: `licencias/`, `FUENTES_Y_LICENCIAS.md`, `fuentes.json`.

Reconstrucción opcional: `python reconstruir_datos.py` descarga los tres archivos,
comprueba sus hashes contra `fuentes.json`, importa en una base temporal y solo
reemplaza la existente tras verificar integridad. Cierra antes la aplicación.
Si una fuente cambia, se detiene sin reemplazar la base. El script requiere
espacio para descargas y base temporal además de la base instalada.
En esta edición ligera, el instalador reconstruye la base automáticamente.
