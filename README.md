# Ruralingo: Arquitectura de IA Descentralizada para el Fortalecimiento Educativo en Zonas Rurales

Prototipo funcional de una plataforma de asistencia pedagógica para el aprendizaje de inglés y portugués, y la preservación del Wayuunaiki, diseñada para operar en redes locales sin conexión a internet en comunidades rurales de Colombia.

> [!IMPORTANT]
> El prototipo funcional se puede visualizar y probar en su versión Demo abriendo el siguiente enlace de GitHub Pages: [Ruralingo](https://jcvtao.github.io/ruralingo/)

Este repositorio constituye el componente de implementación de un proyecto de investigación desarrollado en el marco de la asignatura *Redes de Computadores* de la Universidad Nacional de Colombia (Sede Bogotá) durante el periodo académico 2026-1S.

El sustento teórico completo, incluyendo la formulación de la arquitectura de red, el análisis de viabilidad computacional, los protocolos de seguridad propuestos y la revisión de literatura, se encuentra documentado en el artículo de investigación asociado:

> **Vergara Tao, J. C.** (2026). *Ruralingo: Arquitectura de IA Descentralizada para el Fortalecimiento Educativo en Zonas Rurales*. Departamento de Ingeniería de Sistemas y Computación, Universidad Nacional de Colombia.
>
> 📄 **[Acceder al artículo completo](https://github.com/jcvtao/ruralingo/blob/main/ruralingo-articulo.pdf)**


## Descripción del problema

Las comunidades rurales colombianas enfrentan una brecha digital que va más allá de la falta de dispositivos: la dependencia de servicios centralizados en la nube hace que herramientas educativas basadas en inteligencia artificial sean inaccesibles en zonas donde la conectividad WAN es intermitente o inexistente. Esta condición afecta particularmente el aprendizaje de idiomas con proyección laboral (inglés y portugués) y la preservación de lenguas nativas como el Wayuunaiki, que cuenta con aproximadamente 300.000 hablantes en La Guajira colombo-venezolana y no dispone de recursos pedagógicos digitales locales.

Ruralingo propone desplazar el componente de inferencia de inteligencia artificial desde la nube hacia un nodo local ubicado en la propia institución educativa, eliminando la dependencia de enlaces WAN y garantizando continuidad del servicio dentro del perímetro escolar.


## Arquitectura propuesta

La arquitectura general del sistema se estructura bajo un modelo **cliente-servidor descentralizado**:

```
[Dispositivos móviles de estudiantes]
          │  HTTP/REST sobre TCP/IP (WLAN local)
          │  IEEE 802.11ac/ax · 2.4 GHz / 5 GHz
          ▼
  [Servidor Edge — nodo central de la escuela]
    ├── Punto de acceso inalámbrico (topología estrella)
    ├── Subred privada 192.168.1.0/24 (DHCP para clientes, IP estática para servidor)
    ├── Seguridad: WPA3-Personal (SAE), Client Isolation en AP, HTTPS local
    └── Microservicio de IA (Docker + Ollama)
              │
              └── Modelo de lenguaje pequeño (SLM): phi3 — GGUF 4 bits (~4–5 GB RAM)
```

### Componentes técnicos clave

| Componente | Especificación |
|---|---|
| Capa de red inalámbrica | IEEE 802.11ac/ax, bandas 2.4 GHz y 5 GHz, topología estrella |
| Protocolo de comunicación | HTTP/REST sobre TCP/IP |
| Direccionamiento | IP estática en servidor, DHCP en clientes (192.168.1.0/24) |
| Seguridad WLAN | WPA3-Personal con autenticación SAE, aislamiento de clientes |
| Capa de aplicación | HTTPS local con certificados autogestionados |
| Modelo de lenguaje | phi3 (Microsoft), formato GGUF, cuantización a 4 bits |
| Despliegue del SLM | Contenedor Docker sobre Ollama |
| Hardware objetivo | Servidor x86/ARM de bajo costo, ~8 GB RAM disponibles para el servicio |

La cuantización a 4 bits en formato GGUF reduce el modelo de sus requerimientos originales (~14–16 GB en precisión completa) a un rango operativo de 4 a 5 GB de RAM, haciéndolo compatible con hardware de grado escolar sin aceleración GPU especializada.


## Prototipo funcional

Este repositorio contiene una **Single-Page Application (SPA)** construida en un único archivo `index.html` que implementa la interfaz de usuario y la lógica de comunicación con el servidor Edge local. El prototipo sirve como demostración del flujo cliente-servidor propuesto en el artículo.

### Modos de operación

El prototipo opera en dos modos conmutables mediante un switch visible en la interfaz:

- **Modo Demo Remoto:** Las respuestas son generadas localmente en el navegador a partir de una base de conocimiento preprogramada. No se realiza ninguna petición HTTP. Este modo permite revisar la plataforma desde GitHub Pages sin necesidad de infraestructura local.

- **Modo Servidor Edge Local:** Al activar el switch, cada mensaje del usuario genera una petición `HTTP POST` al servidor Ollama corriendo localmente:

  ```
  POST http://localhost:11434/api/generate
  Content-Type: application/json

  {
    "model": "phi3",
    "prompt": "[system prompt del módulo]\n\nEstudiante: [mensaje]\n\nRespuesta:",
    "stream": false
  }
  ```

  Para garantizar un aprendizaje autónomo y sin mediación docente, el modelo es instruido a responder siempre en cuatro partes fijas: **Respuesta**, **Nota**, **Ejemplo** y **Reto**. Mientras la plataforma procesa la solicitud en el hardware local, la interfaz acompaña la experiencia del estudiante con el mensaje *"La IA está pensando..."*.

### Tecnologías del prototipo

- HTML5, CSS3, JavaScript (ES2021) — sin frameworks ni dependencias de producción
- Tailwind CSS (CDN) — para el sistema de diseño responsivo
- Fuente tipográfica: Inter (Google Fonts)
- Compatible con GitHub Pages sin configuración adicional


## Instalación y ejecución local con Ollama

### Requisitos

- Sistema operativo: Linux, macOS o Windows
- RAM disponible: mínimo 6 GB libres para el modelo phi3
- [Ollama](https://ollama.com) instalado

### Pasos

**1. Instalar Ollama**

- **Linux y macOS:**

  ```bash
  curl -fsSL https://ollama.com/install.sh | sh
  ```

- **Windows:** descargar el instalador `.exe` desde [ollama.com/download](https://ollama.com/download/windows) y ejecutarlo. Se instala como servicio en segundo plano; no requiere configuración adicional.

> [!NOTE]
> En Linux, el instalador también registra Ollama como servicio de systemd, por lo que `ollama serve` puede no ser necesario si el sistema lo inicia automáticamente al arrancar. Verificar con `systemctl status ollama`.

**2. Iniciar el servidor Ollama**

```bash
ollama serve
```

El servidor queda disponible en `http://localhost:11434`.

**3. Descargar el modelo phi3**

```bash
ollama pull phi3
```

La descarga ocupa aproximadamente 2.3 GB. Solo es necesaria una vez; el modelo queda almacenado localmente.

**4. Verificar el modelo**

```bash
ollama list
# Debe aparecer phi3 en la lista
```

**5. Abrir la aplicación**

Descargar o clonar este repositorio y abrir `index.html` directamente en el navegador, o acceder a la [versión publicada en GitHub Pages](https://jcvtao.github.io/ruralingo/). En la interfaz, activar el switch **"Modo Servidor Edge Local"** en la parte superior derecha.

> [!NOTE]
> Ollama habilita CORS para `localhost` por defecto desde su versión 0.1.24. Si se presentan errores de CORS, verificar que la variable de entorno `OLLAMA_ORIGINS` incluya el origen del navegador, o ejecutar `OLLAMA_ORIGINS="*" ollama serve` en entornos de desarrollo.

### Uso del Modo Servidor Edge Local

Con Ollama corriendo y el modelo descargado, el flujo completo en la interfaz es el siguiente:

1. **Mantener la terminal abierta** con `ollama serve` en ejecución durante toda la sesión. Cerrarla interrumpe el servicio.

2. **Abrir `index.html`** en el navegador (doble clic sobre el archivo, o desde la versión en GitHub Pages si se está en la misma red que el servidor).

3. **Activar el switch "Modo Servidor Edge Local"** en la esquina superior derecha de la interfaz. El banner superior cambiará de azul a verde y aparecerá el indicador *"Verificando..."* junto al encabezado del chat.

4. **Esperar la confirmación de conexión.** La interfaz realiza automáticamente un `GET http://localhost:11434/` para comprobar que el servidor responde. Si la conexión es exitosa, el indicador mostrará *"Edge conectado ✓"*. Si muestra *"Sin conexión"*, revisar que `ollama serve` esté activo en la terminal.

5. **Seleccionar un módulo** (Inglés, Portugués o Wayuunaiki) y escribir una pregunta o usar los botones de sugerencia.

6. **Primera inferencia (*cold start*).** La primera consulta de cada sesión puede tardar entre 15 y 60 segundos mientras phi3 carga en memoria. La interfaz muestra *"La IA está pensando..."* durante ese período; esto es normal y esperado. Las consultas siguientes responden en 3 a 15 segundos según el hardware.

> [!IMPORTANT]
> La pestaña del navegador y la terminal con `ollama serve` deben permanecer abiertas simultáneamente. Si se cierra la terminal, las peticiones del Modo Edge Local fallarán con un error de conexión visible en el chat.


## Módulos de aprendizaje

El prototipo incluye tres módulos pedagógicos. A continuación se detallan los contenidos contemplados para cada uno, organizados por nivel de complejidad creciente. En el Modo Demo, los contenidos disponibles directamente mediante los botones de sugerencia están marcados con **[Demo]**; los demás son accesibles mediante texto libre.

### Módulo 1 — Inglés 🇬🇧

Orientado a estudiantes rurales con escasa exposición previa al inglés. El rango de nivel contemplado es A1 hasta B1+/B2, de acuerdo con el Marco Común Europeo de Referencia para las Lenguas (MCER/CEFR). El enfoque privilegia vocabulario de uso cotidiano y estructuras gramaticales de alta frecuencia, con ejemplos contextualizados en la realidad colombiana.

#### Nivel A1 — Principiante

| Tema | Contenidos específicos |
|---|---|
| Saludos e introducciones **[Demo]** | Hello/Hi, Good morning/afternoon/evening, Nice to meet you, How are you? Presentaciones personales (nombre, origen, edad) |
| Alfabeto y fonética | Las 26 letras, pronunciación de vocales largas y cortas, diferencias con el español (V/B, H muda, W, Y, Z) |
| Números | Cardinales 1–100, ordinales 1°–20°, precios y cantidades |
| Colores **[Demo]** | Red, blue, green, yellow, orange, purple, white, black, brown. Regla: adjetivo antes del sustantivo |
| Vocabulario básico | Objetos del salón de clase, partes del cuerpo, animales comunes, días de la semana, meses del año |
| Verbo *to be* | Afirmativo, negativo e interrogativo; contracciones; usos con nombre, origen y edad |

#### Nivel A2 — Básico

| Tema | Contenidos específicos |
|---|---|
| Familia y relaciones | Mother, father, brother, sister, grandmother, grandfather, uncle, aunt, cousin; frases para describir la familia |
| Presente Simple | Rutinas diarias, tercera persona singular (-s), negaciones con *don't/doesn't*, preguntas con *do/does* |
| Presente Continuo | Estructura *am/is/are + -ing*, diferencia con el Presente Simple, reglas de ortografía del gerundio |
| Comidas y hábitos alimenticios **[Demo]** | Breakfast, lunch, dinner; vocabulario de alimentos (rice, chicken, fish, bread, fruit); frases en restaurante |
| Preposiciones de lugar **[texto libre]** | In, on, at, under, between, next to, behind, in front of; descripciones de ubicación en el salón |
| Artículos | *A/an* (indefinido) y *the* (definido); omisión del artículo en inglés vs. español |

#### Nivel B1 — Intermedio

| Tema | Contenidos específicos |
|---|---|
| Past Simple **[Demo]** | Verbos regulares (-ed) e irregulares de alta frecuencia (go→went, see→saw, eat→ate, speak→spoke, make→made, buy→bought); afirmativo, negativo (*didn't*) e interrogativo (*Did...?*) |
| Past Continuous | Was/were + -ing; uso para acciones en curso interrumpidas; contraste con Past Simple |
| Futuro **[Demo]** | *Will* (decisiones espontáneas, predicciones, promesas) vs. *Be going to* (planes previos, predicciones con evidencia); diferencias de uso con ejemplos |
| Verbos modales | *Can/could* (capacidad), *should* (consejo), *must/have to* (obligación), *may/might* (posibilidad) |
| Comparativos y superlativos | Adjetivos cortos (-er/-est), adjetivos largos (more/most), irregulares (good→better→best) |
| Vocabulario temático | Salud y partes del cuerpo, medios de transporte, el entorno natural y rural, tecnología básica |

#### Nivel B1+/B2 — Intermedio alto

| Tema | Contenidos específicos |
|---|---|
| Condicionales | Tipo 1 (*If + Present Simple, will*): situaciones reales o probables. Tipo 2 (*If + Past Simple, would*): situaciones hipotéticas |
| Voz pasiva | Presente y pasado simples en voz pasiva; cuándo y por qué se usa; transformación de oraciones activas |
| Discurso reportado | *Say* vs. *tell*; cambios en tiempos verbales y pronombres; preguntas reportadas |
| Phrasal verbs comunes | *Look up, turn on/off, give up, find out, carry out*; uso en contexto |
| Present Perfect | *Have/has + participio pasado*; contraste con Past Simple; marcadores temporales (*already, yet, ever, never, just*) |


### Módulo 2 — Portugués (Português Brasileiro) 🇧🇷

Diseñado para hispanohablantes colombianos que aprovechan la proximidad tipológica entre el español y el portugués. El nivel objetivo es **A1 hasta B1**, con énfasis en las diferencias de pronunciación y los *falsos cognatos* que generan mayor confusión en hablantes de español. Todos los ejemplos corresponden al portugués brasileño estándar.

#### Nivel A1 — Principiante

| Tema | Contenidos específicos |
|---|---|
| Saudações **[Demo]** | Olá/Oi, Bom dia/Boa tarde/Boa noite, Tudo bem?, Obrigado/Obrigada, Tchau/Até logo; diferencias de registro formal e informal |
| Verbo SER **[Demo]** | Conjugación completa: sou, é, somos, são; usos con identidad, origen, profesión y características permanentes |
| Verbo ESTAR **[Demo]** | Conjugación: estou, está, estamos, estão; usos con estados emocionales, ubicación temporal; diferencia con SER en ubicación de lugares fijos |
| Números | Um/uma, dois/duas, três–vinte; decenas hasta cem; comparación directa con los equivalentes en español |
| Família **[Demo]** | Pai/mãe, filho/filha, irmão/irmã, avô/avó, tio/tia, primo/prima; frases descriptivas sobre la familia |
| Artículos | O/a (definidos), um/uma (indefinidos); uso con sustantivos masculinos y femeninos |

#### Nivel A2 — Básico

| Tema | Contenidos específicos |
|---|---|
| Pronunciación brasileña **[Demo]** | Vogais nasais (ã, õ, em/en): *mãe, limões*; dígrafo *lh* (sonido "ll"): *filho, família*; el "r" inicial suena como "j" española: *Rio → "Jío"*; *d/t* antes de *i* → "dj/tch": *dia → "djia"*, *tia → "tchia"* |
| Presente do indicativo | Verbos regulares en -ar (falar), -er (comer), -ir (partir); conjugación completa; verbos irregulares comunes: ir, ter, fazer, querer, poder |
| Comida y alimentação | Café da manhã / almoço / jantar; vocabulario: arroz, feijão, frango, peixe, pão; *falso cognato*: *borracha* = neumático (no borracha); frases en restaurante |
| Pronombres personales | Eu, você, ele/ela, nós, vocês, eles/elas; uso de *você* en Brasil (equivalente a "tú/usted") |
| Falsos cognatos | *Borracha* (neumático), *polvo* (polvo de suelo), *embarazada* (no existe; *grávida* = embarazada), *exquisito* (extravagante), *borracho* (no existe; *bêbado* = borracho) |

#### Nivel B1 — Intermedio

| Tema | Contenidos específicos |
|---|---|
| Pretérito Perfeito Simples | Equivalente al Pretérito Indefinido del español; verbos regulares e irregulares: *fui, vi, fiz, tive, disse, pude*; marcadores temporales: *ontem, semana passada, há dois anos* |
| Pretérito Imperfeito | Acciones habituales en el pasado o en curso al momento de otra acción; contraste con el Perfeito Simples |
| Futuro com *ir* | *Vou/vai/vamos + infinitivo*: equivalente a "voy a + infinitivo" en español; uso más frecuente que el futuro sintético en el habla coloquial brasileña |
| Preposições e contrações | *Em + o = no, em + a = na, de + o = do, de + a = da, a + o = ao*; uso con verbos de movimiento y lugar |
| Vocabulario temático | Saúde (médico, hospital, sintomas), meio ambiente, trabalho e profissões, tecnologia e comunicação |
| Expressões idiomáticas | *Que saudade!, Que delícia!, Deixa pra lá, Dar um jeito, Tá bom/Tá certo*; usos contextuales |


### Módulo 3 — Wayuunaiki 🪶

El Wayuunaiki (también escrito *Wayuu* o *Guajiro*) es la lengua de mayor vitalidad entre las lenguas indígenas de Colombia, con comunidades activas en los departamentos de La Guajira (Colombia) y el estado de Zulia (Venezuela). A diferencia de los módulos de inglés y portugués, el objetivo de este módulo no es la adquisición progresiva de una lengua extranjera, sino la **preservación, documentación y difusión** de una lengua en riesgo de desplazamiento por parte del español.

#### Saludos y comunicación básica

> **Disponible en Demo**

| Wayuunaiki | Español | Pronunciación aproximada |
|---|---|---|
| **Anaswachi** | Buenos días | /a-nas-WA-chi/ |
| **Anasü** | Hola / Saludos | /a-NA-sü/ |
| **Jamaya wayuu?** | ¿Cómo está usted? | /cha-MA-ya WA-yu/ |
| **Anashajaashii** | Estoy bien | /a-nas-ha-CHÁ-shi/ |
| **Eein** | Sí | /e-ÉIN/ |
| **Jaaya** | No | /CHÁ-ya/ |
| **Müinjee** | Gracias | /mü-in-CHÉ-e/ |
| **Maatein** | De nada | /má-tein/ |
| **Waneepia** | Por favor | /wa-né-PIA/ |
| **Nnojotsü** | No sé | /nno-CHO-tsü/ |

#### Números (pütchi sümaa wayuunaiki)

> **Disponible en Demo**

| # | Wayuunaiki | Pronunciación |
|---|---|---|
| 1 | wanee | /wa-NÉ-e/ |
| 2 | piamüin | /pia-MÜ-in/ |
| 3 | apüinche | /a-PÜIN-che/ |
| 4 | pienchon | /pien-CHÓN/ |
| 5 | jayeechi | /cha-YÉ-chi/ |
| 6 | jaleechi | /cha-LÉ-chi/ |
| 7 | katsüinsü | /kat-SÜIN-sü/ |
| 8 | shikiimutsü | /shi-kii-MU-tsü/ |
| 9 | puesükateechi | /pue-sü-ka-TÉ-chi/ |
| 10 | pienchi | /PIÉN-chi/ |

#### Naturaleza y cosmovisión

> **Disponible en Demo**

La relación del pueblo Wayuu con el entorno natural no es simplemente léxica: los elementos naturales son concebidos como seres con agencia y personalidad propia. *Juyá* (la lluvia) es un ser espiritual masculino, padre simbólico del pueblo Wayuu; *Pülowi* representa el principio femenino asociado a la sequedad y a la vida animal; *Maleiwa* es el ser supremo creador. Esta dimensión cosmológica está presente en el vocabulario cotidiano.

| Wayuunaiki | Español | Contexto cultural |
|---|---|---|
| **Juyá** | Lluvia | Ser espiritual masculino; cuando llueve, "Juyá está pasando" |
| **Mma** | Tierra | Madre nutricia del pueblo |
| **Palaa** | Mar Caribe | El océano que define el territorio Wayuu |
| **Kai** | Sol | Guía y fuente de vida |
| **Müin** | Agua dulce | Recurso sagrado y escaso en La Guajira |
| **Pülowi** | Espíritu femenino | Dueña de animales; contrapartida de Juyá |
| **Maleiwa** | Ser Supremo | Entidad creadora de la cosmovisión Wayuu |
| **Kaa'ula** | Flamenco rosado | Ave totémica de La Guajira |
| **Wunu'u** | Planta medicinal | Conocimiento ancestral de curación |

#### Arte textil y colores

> **Disponible en Demo**

El tejido Wayuu (*wayuumutsü*) fue declarado **Patrimonio Cultural Inmaterial de la Humanidad** por la UNESCO en 2017. Cada pieza es tejida exclusivamente por mujeres y puede requerir entre 8 y 30 días de trabajo. Los patrones (*kanaas*) no son decorativos en sentido abstracto: codifican historias familiares, relaciones con el territorio y elementos de la cosmovisión. La mochila (*susu*) es el objeto textil de mayor reconocimiento internacional.

| Wayuunaiki | Color / elemento | Simbolismo |
|---|---|---|
| **Müna** | Rojo | Vida, sangre, fuerza |
| **Sulu'u** | Negro | Protección, profundidad |
| **Wasü** | Blanco | Pureza, paz |
| **Yoluja** | Amarillo | Sol, prosperidad |
| **Pirruya** | Verde | Naturaleza, esperanza |
| **Kaatuushikat** | Azul | El mar Caribe, el cielo |

Patrones *kanaas* más representativos: rombos (ojos de la araña: sabiduría y paciencia), líneas horizontales (ríos y caminos de La Guajira), zigzag (montañas de la Sierra Nevada de Santa Marta).

#### Familia y organización social

La sociedad Wayuu es **matrilineal**: la identidad, la herencia y la pertenencia al clan (*eirruku*) se transmiten por línea materna. Existen doce clanes principales, cada uno con un animal totémico: Epiayu, Uriana, Pushaina, Ipuana, Epieyuu, Uliana, Sapuana, Jarariyu, Pausayuu, Sijona, Pimienta y Urariyu.

| Wayuunaiki | Español | Nota |
|---|---|---|
| **Tü apüshii** | Mi familia / Mi clan materno | El clan materno es la unidad social fundamental |
| **Eirruku** | Clan (apellido materno) | Define la identidad colectiva |
| **Alijuna** | Persona no Wayuu | Término para referirse a personas ajenas a la comunidad |
| **Ouutsu** | Curandero / Chamán | Interpreta los sueños (*lapü*) y atiende enfermedades espirituales |
| **Lapü** | Sueño / Mensaje espiritual | Los sueños son mensajes del mundo espiritual en la cosmovisión Wayuu |
| **Yoluja** | Alma / Espíritu de los difuntos | Presente en la práctica del segundo velorio |

> [!NOTE]
> Debido a que el Wayuunaiki es una lengua aglutinante, con orden sintáctico VSO y fonemas sin equivalente en español, el asistente de IA se limita a un alcance estrictamente introductorio. La plataforma no pretende sustituir la transmisión comunitaria intergeneracional ni el aprendizaje con hablantes nativos, quienes son las autoridades lingüísticas legítimas.


## Consultas disponibles en Modo Demo

En el Modo Demo (sin Ollama), las siguientes preguntas activadas por botón garantizan una respuesta correcta mediante búsqueda directa por ID interno (no por coincidencia de palabras clave):

| Módulo | Botón | Lección |
|---|---|---|
| 🇬🇧 Inglés | Saludos en inglés | Greetings & Introductions |
| 🇬🇧 Inglés | El Past Simple | Past Simple (regular e irregular) |
| 🇬🇧 Inglés | Los colores en inglés | Colors + regla adjetivo-sustantivo |
| 🇬🇧 Inglés | El futuro: will y going to | Future tense (usos comparados) |
| 🇧🇷 Portugués | Saludos en portugués | Saudações em Português |
| 🇧🇷 Portugués | SER y ESTAR en portugués | Verbos SER e ESTAR completos |
| 🇧🇷 Portugués | Pronunciación del portugués | Vogais nasais, LH, R brasileño |
| 🇧🇷 Portugués | La familia en portugués | Família em Português |
| 🪶 Wayuunaiki | Saludos en Wayuunaiki | Saludos + contexto cultural |
| 🪶 Wayuunaiki | Números en Wayuunaiki | Números 1–10 con pronunciación |
| 🪶 Wayuunaiki | ¿Qué significa Juyá? | Naturaleza y cosmovisión Wayuu |
| 🪶 Wayuunaiki | El tejido y arte Wayuu | Arte textil, colores y patrones kanaas |

La entrada por texto libre activa respuestas adicionales (preposiciones, comidas, familia en inglés, números en portugués, frases básicas en Wayuunaiki, entre otras) mediante detección de palabras clave en español.


## Decisiones de implementación relevantes para la evaluación de Redes

- **Ausencia de `AbortSignal` en la petición Fetch:** La petición HTTP al servidor Ollama no incluye límite de tiempo. El modelo phi3, en su primera inferencia tras el inicio del servidor, puede tardar entre 15 y 60 segundos en responder dependiendo del hardware disponible (*cold start*). Interrumpir la petición en ese período produciría errores confusos para el usuario. El indicador *"La IA está pensando..."* comunica visualmente el estado de espera.

- **Separación de rutas de código para botones y texto libre:** Los botones de sugerencia utilizan `sendDirectResponse(id)`, que hace una búsqueda por identificador único en el array `MOCK`. El texto libre usa `getMockResponse(input, module)`, que aplica coincidencia de subcadenas sobre un array de palabras clave. Esta separación evita falsos positivos que ocurrirían si los botones usaran el mismo matcher (por ejemplo, la palabra clave `"one"` aparece como subcadena dentro de `"preposici`**`one`**`s"`).

- **Limpieza del historial al cambiar de módulo:** El área de chat se vacía completamente al cambiar entre Inglés, Portugués y Wayuunaiki. Esto libera memoria en el navegador y evita que el contexto de un módulo contamine la interacción en otro.


## Conclusiones del artículo de investigación

El artículo demuestra que la independencia funcional de la inteligencia artificial respecto de la nube es técnicamente viable en contextos escolares rurales, siempre que el procesamiento se traslade a una infraestructura local basada en Edge Computing. La combinación de una WLAN en estrella, un servidor Edge en el perímetro de la escuela y un esquema de direccionamiento privado reduce la dependencia de enlaces externos y mantiene la continuidad del servicio. La cuantización del modelo a 4 bits en formato GGUF, combinada con la contenedorización mediante Docker, constituye una estrategia adecuada para mitigar las restricciones de hardware típicas en equipos de bajo costo, sin sacrificar la calidad de inferencia necesaria para la interacción pedagógica.

Finalmente, el mantenimiento del corpus lingüístico —incluyendo materiales en Wayuunaiki— dentro de una subred privada local refuerza los principios de soberanía digital: los datos lingüísticos de la comunidad no salen del perímetro escolar, eliminando el riesgo de extracción o reutilización no autorizada por parte de plataformas externas.


### Trabajos futuros

De acuerdo con el artículo, las líneas de desarrollo subsecuentes incluyen:

- Desarrollo de la aplicación móvil nativa y definición detallada de su interfaz de usuario
- Pruebas piloto en una comunidad educativa real para medir latencia bajo concurrencia múltiple y estabilidad del servicio bajo condiciones operativas auténticas
- Ampliación del soporte multilingüe con validación comunitaria para otras lenguas nativas colombianas
- Evaluación del comportamiento del sistema frente a cargas reales y ajuste de parámetros de red y cómputo
