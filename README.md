PAV - P5: síntesis musical polifónica
=====================================

Obtenga su copia del repositorio de la práctica accediendo a [Práctica 5](https://github.com/albino-pav/P5) 
y pulsando sobre el botón `Fork` situado en la esquina superior derecha. A continuación, siga las
instrucciones de la [Práctica 2](https://github.com/albino-pav/P2) para crear una rama con el apellido de
los integrantes del grupo de prácticas, dar de alta al resto de integrantes como colaboradores del proyecto
y crear la copias locales del repositorio.

Como entrega deberá realizar un *pull request* con el contenido de su copia del repositorio. Recuerde que
los ficheros entregados deberán estar en condiciones de ser ejecutados con sólo ejecutar:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  make release
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A modo de memoria de la práctica, complete, en este mismo documento y usando el formato *markdown*, los
ejercicios indicados.

Ejercicios.
-----------

### Envolvente ADSR.

Tomando como modelo un instrumento sencillo (puede usar el InstrumentDumb), genere cuatro instrumentos que
permitan visualizar el funcionamiento de la curva ADSR.

Per tal d'observar la corba ADSR hem generat 4 instruments amb difrents ADSR, tenint en compte el model del instrument seno que farem més endevant.

* Un instrumento con una envolvente ADSR genérica, para el que se aprecie con claridad cada uno de sus
  parámetros: ataque (A), caída (D), mantenimiento (S) y liberación (R).


![alt text](/img/Seno_generico_sin_efectos.PNG)
![alt text](/img/ADSR_Seno_generico_sin_efectos.PNG)


Hem generat el fitcher .wav amb el programa synth i la corba ADSR desitjada, la mostrem en un editor d'audio i veiem que alhora de fer la seva representació temporal s'observa molt be la corba ADSR, mirant com l'amplitud de la nota no es constant al llarg del temps, sino que passa per diferents fases d'intensitats. Observem un atac rapid, una caiguda moderada, un llarg manteniment i una lliberació normal. 



* Un instrumento *percusivo*, como una guitarra o un piano, en el que el sonido tenga un ataque rápido, no
  haya mantenimiemto y el sonido se apague lentamente.
  - Para un instrumento de este tipo, tenemos dos situaciones posibles:
    * El intérprete mantiene la nota *pulsada* hasta su completa extinción.

    ![alt text](/img/adrseno_final_biel.PNG)
    ![alt text](/img/ADSR.PNG)

    Observem que com l'atac ha estat més rapid i la caiguda ha estat lenta, ja no es nota que es va apagan poc a poc, no te el manteniment i la lliberació també es molt lenta, ja que seria com si deixesim la corda apretadea i aixi sol es lliuraria realment al començar la seguent nota i sortir de l'anterior, hem exentuat la cua per tal de que es veigues, ja que sino no s'ens veia que arribava a la seguent casi casi.

    * El intérprete da por finalizada la nota antes de su completa extinción, iniciándose una disminución
	  abrupta del sonido hasta su finalización.

    ![alt text](/img/seno_percusivo_sin_efectos.PNG)
    ![alt text](/img/ADSR_seno_percusivo_sin_efectos.PNG)

    Observem que en l'ataca ràpid la caiguda es lenta, ja que la nota es va apagant lentament i de cop la deixem anar, aixi deixant molt poc temps per la lliberació d'aquesta. S'observa com la nota s'acaba més rapid sense arribar a tocar la seguent.


  - Debera representar en esta memoria **ambos** posibles finales de la nota.
* Un instrumento *plano*, como los de cuerdas frotadas (violines y semejantes) o algunos de viento. En
  ellos, el ataque es relativamente rápido hasta alcanzar el nivel de mantenimiento (sin sobrecarga), y la
  liberación también es bastante rápida.

  ![alt text](/img/seno_plano_sin_efectos.PNG)
  ![alt text](/img/ADSR_SENO_PLANO.PNG)
  

  Si observem, viem que la corba ADSR compleix a la perfecció amb lo estipulat, el atac és relativament rapid, la caiguda més o menys rapida, el manteniment es molt llarg i la lliberació és prou lenta. Això ho vam fer perquè ens semblava més versemblant amb el so d’un violí, en què les notes se solapen, és a dir, una no s’apaga fins que comença la següent.


Para los cuatro casos, deberá incluir una gráfica en la que se visualice claramente la curva ADSR. Deberá
añadir la información necesaria para su correcta interpretación, aunque esa información puede reducirse a
colocar etiquetas y títulos adecuados en la propia gráfica (se valorará positivamente esta alternativa).

### Instrumentos Dumb y Seno.

Implemente el instrumento `Seno` tomando como modelo el `InstrumentDumb`. La señal **deberá** formarse
mediante búsqueda de los valores en una tabla.

- Incluya, a continuación, el código del fichero `seno.cpp` con los métodos de la clase Seno.

```c
    #include <iostream>
    #include <math.h>
    #include "seno.h"
    #include "keyvalue.h"

    #include <stdlib.h>

    using namespace upc;
    using namespace std;

    //El modo sencillo de generar un sonido periodico de frecuencia
    //variable es almacenar un periodo del mismo en una tabala y 
    //recorrer esta a una velocidad adecuada para conseguir
    //el pitch deseado

    //Constructor
    seno::seno(const std::string &param) 
      : adsr(SamplingRate, param) {
      bActive = false;
      x.resize(BSIZE);

      /*
        You can use the class keyvalue to parse "param" and configure your instrument.
        Take a Look at keyvalue.h    
      */

      //Usando funciones de la libreria keyvalue.h analizamos
     //la cadena de parámetros de interes para el instrumento, en este caso
     //solamente la longitud de la tabla N, ya que los parametros ADSR se procesan 
     //en otra parte
      KeyValue kv(param);
      int N;

      if (!kv.to_int("N",N))
        N = 40; //default value
  
      //Se crea la tabla con un periodo de sinusoide
      tbl.resize(N);
      float phase = 0, step = 2 * M_PI /(float) N;
      index = 0;
      for (int i=0; i < N ; ++i) {
        tbl[i] = sin(phase);
        phase += step;
      }
    }

  //Cada vez que el programa encuentra un comando MIDI en el fichero score, invoca al
  //método command(comando, la nota, la velocidad)
  //si el comando es 9 (NoteOn) se declara activo el instrumento
  //inicializa curva ADSR y un contador index que recorre la tabla,
  //de amplitud A. Si cmd = 8 o 0 (NoteOff o EndNote) el metoo inicia la fase release de la curva
  //o fiinaliza el sonido, respectivament
  void seno::command(long cmd, long note, long vel) {
    //NoteOn instrumento Activo
    if (cmd == 9) {		//'Key' pressed: attack begins
      bActive = true;
      adsr.start();
      index = 0;
      A = vel / 127.;
      //Sabiendo la correspondencia de La central definimos F0 para otras notas y definimos los saltos
      float F0 = 440.00 * pow(2, (note - 69.00)/12.00) / SamplingRate;
      step = tbl.size() * F0;
    }
    //Fase release de la curva
    else if (cmd == 8) {	//'Key' released: sustain ends, release begins
      adsr.stop();
    }
    //Finaliza el sonido
    else if (cmd == 0) {	//Sound extinguished without waiting for release to end
      adsr.end();
    }
  }

  //La sintesis se realiza aquí
  //La nota puede encontrarse en 3 situaciones Finalizada, Inactiva, Activa:
  const vector<float> & seno::synthesize() {
    //Finalizada: si la curva ha llegado a su nivel final
    if (not adsr.active()) {
      x.assign(x.size(), 0);
      bActive = false;
      return x;
    }
    //Inactiva si ya ha sido marcada como inactiva
    else if (not bActive)
      return x;
    //Activa, se aplica la envolvente ADSR, aquí es donde se recorre la tabla
    for (unsigned int i=0; i<x.size(); ++i) {
      // Primera aproximación: redondeando el indice requerido para acceder a valores de la tabla
      float new_index = round(index * step);
      x[i] = A * (tbl[new_index]);
      index += 1;
      if (new_index == tbl.size()) index = 0;
    }
    adsr(x); //apply envelope to x and update internal status of ADSR

    return x;
  }
   ```


- Explique qué método se ha seguido para asignar un valor a la señal a partir de los contenidos en la tabla,
  e incluya una gráfica en la que se vean claramente (use pelotitas en lugar de líneas) los valores de la
  tabla y los de la señal generada.

    L’instrument seno funciona de la manera següent: es pren un període del senyal sinusoidal i es guarda a la taula amb N mostres. A continuació, aquest període registrat a la taula es pot recórrer més de pressa o més a poc a poc, i això és el que farem per generar notes més agudes o més greus, respectivament.

    ![alt text](/img/seno.jpeg)

- Si ha implementado la síntesis por tabla almacenada en fichero externo, incluya a continuación el código
  del método `command()`.

### Efectos sonoros.

- Incluya dos gráficas en las que se vean, claramente, el efecto del trémolo y el vibrato sobre una señal
  sinusoidal. Deberá explicar detalladamente cómo se manifiestan los parámetros del efecto (frecuencia e
  índice de modulación) en la señal generada (se valorará que la explicación esté contenida en las propias
  gráficas, sin necesidad de *literatura*).

***Tremolo***

  Hem generat dues gràfiques de Trèmolo, una amb una profunditat de modulació molt gran, per tant, hi haurà variacions molt grans en el volum del so i la seva amplitud, i, en canvi, una freqüència de modulació petita, és a dir, que hi haurà canvis cada més temps, no són gaire seguits els canvis.

  ![alt text](/img/tremolo.png)

  En el segon cas que es mostra a continuació, les variacions seran més versemblants i així l’amplitud de la sinusoide no es veurà tan afectada:

  ![alt text](/img/tremolo2.png)

  ***Vibrato***

  Pel que fa al *vibrato*, sabem que varia la freqüència fonamental de la nota en funció dels paràmetres d’*Intensitat* i de la *freqüència de modulació*. A la següent imatge podem observar com totes dues comencen alhora, però no oscil·len a la mateixa freqüència, ja que el vibrato la va canviant:

  ![alt text](/img/vibrato.png)

  ![alt text](/img/vibrato2.png)

  En aquesta segona imatge podem veure com, mentre que en la segona gràfica la freqüència es manté constant, a la de dalt la freqüència va variant, comprimint i estirant la freqüència fonamental d’una mateixa nota. Aquest efecte pot resultar agradable en certs estils de música i instruments, sempre que no ens passem canviant la freqüència. En l’exemple de la guitarra o el baix, queda molt bé en certs moments moure una mica el dit per la corda per canviar la tensió produïda i generar vibrato, però si ens desplacem dos trasts avall, ja sona malament, perquè és clarament una altra nota. De fet, moltes guitarres porten un pont flotant o semiflotant per produir l’efecte de vibrato, que queda molt bé si se sap balancejar correctament el pont, canviant la tensió de les cordes i, per tant, la seva afinació en diversos estils i cançons.



- Si ha generado algún efecto por su cuenta, explique en qué consiste, cómo lo ha implementado y qué
  resultado ha producido. Incluya, en el directorio `work/ejemplos`, los ficheros necesarios para apreciar
  el efecto, e indique, a continuación, la orden necesaria para generar los ficheros de audio usando el
  programa `synth`.

### Síntesis FM.

Construya un instrumento de síntesis FM, según las explicaciones contenidas en el enunciado y el artículo
de [John M. Chowning](https://web.eecs.umich.edu/~fessler/course/100/misc/chowning-73-tso.pdf). El
instrumento usará como parámetros **básicos** los números `N1` y `N2`, y el índice de modulación `I`, que
deberá venir expresado en semitonos.

- Use el instrumento para generar un vibrato de *parámetros razonables* e incluya una gráfica en la que se
  vea, claramente, la correspondencia entre los valores `N1`, `N2` e `I` con la señal obtenida.

  Mitjançant les explicacions incloses a la pràctica i l’article de John M. Chowning, i basant-nos en l’instrument seno, hem produït un instrument que anomenem seno_vibrato, que en realitat es correspon amb l’instrument produït mitjançant Síntesi FM.

  ![alt text](/img/sintesisFM.jpeg)

- Use el instrumento para generar un sonido tipo clarinete y otro tipo campana. Tome los parámetros del
  sonido (N1, N2 e I) y de la envolvente ADSR del citado artículo. Con estos sonidos, genere sendas escalas
  diatónicas (fichero `doremi.sco`) y ponga el resultado en los ficheros `work/doremi/clarinete.wav` y
  `work/doremi/campana.work`.
  * También puede colgar en el directorio work/doremi otras escalas usando sonidos *interesantes*. Por
    ejemplo, violines, pianos, percusiones, espadas láser de la
	[Guerra de las Galaxias](https://www.starwars.com/), etc.

  Basant-nos en l’article de John M. Chowning i en el nostre codi, hem buscat la relació entre N1/N2 i l’envolupant ADSR per a cadascun dels instruments que se’ns demana (campana i clarinet).

  ***Clarinete***

  Aquí veiem, segons l’article, l’envolupant ADSR de l’instrument clarinet. A partir d’això, definim els nostres paràmetres. També, segons l’article, la relació N1/N2 ha de ser 3/1, i així ho hem deixat.

  ![alt text](/img/clarinete.png)

  A la següent gràfica, observem l’envolupant ADSR d’una campana. També veiem que la relació N1/N2 ha de ser 1/1.4; aquesta sí que la seguim, perquè, a més, en no ser una relació entera, generem una relació inharmònica com la de les campanes.

  ![alt text](/img/bell.png)


### Orquestación usando el programa synth.

Use el programa `synth` para generar canciones a partir de su partitura MIDI. Como mínimo, deberá incluir la
*orquestación* de la canción *You've got a friend in me* (fichero `ToyStory_A_Friend_in_me.sco`) del genial
[Randy Newman](https://open.spotify.com/artist/3HQyFCFFfJO3KKBlUfZsyW/about).

- En este triste arreglo, la pista 1 corresponde al instrumento solista (puede ser un piano, flautas,
  violines, etc.), y la 2 al bajo (bajo eléctrico, contrabajo, tuba, etc.).
- Coloque el resultado, junto con los ficheros necesarios para generarlo, en el directorio `work/music`.
- Indique, a continuación, la orden necesaria para generar la señal (suponiendo que todos los archivos
  necesarios están en directorio indicado).

También puede orquestar otros temas más complejos, como la banda sonora de *Hawaii5-0* o el villacinco de
John Lennon *Happy Xmas (War Is Over)* (fichero `The_Christmas_Song_Lennon.sco`), o cualquier otra canción
de su agrado o composición. Se valorará la riqueza instrumental, su modelado y el resultado final.
- Coloque los ficheros generados, junto a sus ficheros `score`, `instruments` y `efffects`, en el directorio
  `work/music`.
- Indique, a continuación, la orden necesaria para generar cada una de las señales usando los distintos
  ficheros.


Fent servir el programa synth, hem generat la cançó de Toy Story assignant els instruments que se’ns demanaven. He fet tres versions, totes amb el mateix instrument greu:

La primera té el clarinet fet amb síntesi FM com a solista.

La segona té un instrument percutit com a solista.

La tercera és per oblidar, amb la campana com a solista; suposo que és perquè, en generar inharmònics, no està en el to de l’altre instrument.

Els comandos que hem utilitzat són els següents:

  synth toy_story_clarinete.orc ToyStory_A_Friend_in_me.sco toy_story_percutido.wav

  synth toy_story_clarinete.orc ToyStory_A_Friend_in_me.sco toy_story_clarinete.wav



> NOTA:
>
> No olvide escuchar el resultado generado y comprobar que no se producen ruidos extraños o distorsiones.
> Sobre todo, tenga en cuenta la salud auditiva de quien será encargado de corregir su trabajo.
