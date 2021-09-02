# bot
## Twitter Bot
# El día que me reemplazó un Bot
https://medium.com/@alejandrocrosa/el-d%C3%ADa-que-me-reemplaz%C3%B3-un-bot-adba86e45e87
Alejandro Crosa

Jul 7, 2017·5 min read


Tuve uno de esos días donde te discuten cosas insólitas y sin argumentos en Twitter, me canse y decidí armar un bot que me reemplace unos días, y así no tener que lidiar con conversaciones ridículas y repetitivas. Vacaciones de Twitter!

Tendría que armar un bot que hable como yo, tuitee cosas parecidas y responda usando palabras que suelo utilizar (ademas de imágenes de memes), en un par de horas del fin de semana baje todo mi historial de tuits, los procesé removiendo usuarios, urls, y demás, finalmente pude armar un pequeño dataset de unos ~20.000 tuits.
Como mi idea era armar una red neuronal y entrenarla con esos datos (estoy aprendiendo ML), agregue también algunos posts que había escrito a ese corpus de tuits y algún que otro texto que encontré por ahi que había escrito anteriormente.
Después alquilé un servidor de Amazon con un buena placa de video (p2.xlarge GPU) y deje corriendo todo el finde un script que analizaba esos tuits entrenando el modelo que finalmente aprendería mi estilo de hablar.
El script además imprimía pequeñas frases a medida que la red neuronal aprendía de mis textos, con lo cual cada tanto podia meterme e ir viendo pequeños avances en el aprendizaje. Al principio imprimía cosas sin sentido, pero de apoco comenzaba a aprender patrones entre las letras y de a poco comenzaba a formar palabras que tenían sentido. Después de unas horas comenzaba a escribir frases que tenían mas sentido, hasta que luego de dos días podia escribir frases que tranquilamente hubiese escrito yo mismo (aunque a veces escribía sin sentidos).

Algunas de las primeras frases del bot
Finalmente llegó el lunes y arme un script que se conectaba a Twitter y al recibir mensajes de otros usuario usaba ese modelo entrenado (el Alejandro Bot) y respondía como si fuese yo.
La experiencia ademas de ser super divertida, me ayudo a darme de cuenta de varias cosas:
No es muy descabellado pensar que con un data set muchísimo mas grande y un entrenamiento mucho mayor le resultaría muy difícil a la gente diferenciar entre un Bot o una persona real.
Ser reemplazado por un Bot puede ser catastrófico si el bot tuitea algo que sea violento, misógino o directamente inapropiado usando tu identidad. Algun que otro tuit lo tuve que borrar porque me pareció inapriopiado. De hecho un usuario bloqueo al bot pensando que era una persona.
Responder tuits es una actividad muy poco estructurada, pero tranquilamente se podrían generar Bots que automaticen tareas que tienen mas estructura, por ej. pagar cuentas, responder preguntas básicas, recolectar información por nosotros (asistentes virtuales, et al.)

Las respuestas de los Bots no son super precisas, pero a veces como el cerebro de los humanos ajusta ante errores, llegan a interpretar cosas a partir de una frase inconexa.
Dejar un Bot haciendo una tarea que a veces no nos gusta (responder tuits) es liberador y me encantaría en el futuro automatizar otros aspectos de mi vida de manera mas controlada, creo que ahi hay muchísimo futuro e industrias enteras.
¿Como se hace un bot?
Hay varias formas de hacer Bots, pero acá voy a explicar como hice el mío y el de Ricardo Fort (@ricardobort).
Mi Bot usó una red neuronal entrenada con mis tuits, pero generar eso es costoso ya que lleva mucho tiempo y recursos, una instancia de Amazon con un buen GPU sale aproximadamente $1 por hora , si consideras que un modelo puede tardar varios días en entrenarse completamente no es un gasto trivial. Si bien es muchísimo mas caro usar una maquina con GPU en Amazon, el tiempo de iteración te permite ver el resultado del modelo muchísimo mas rápido en vez de tener que esperar días para ver que onda lo que entrenaste, y una vez que tenés el modelo guardado en disco podes pasar el código a una maquina mas barata (no hace falta el GPU para generar las secuencias rápidamente), con un m2 para generar las frases es suficiente.
Para entrenar el Bot use Tensorflow, gracias a TFLearn que es una abstracción que utiliza Tensorflow a bajo nivel. En este caso particular utilicé una red neuronal que permite predecir secuencias, es decir, dado un aprendizaje anterior y sugiriendo un valor nuevo, predice el siguiente con cierta probabilidad. En este caso el valor sugerido es una secuencia de caracteres (una frase). Lo increíble es que la red no predice frases, sino que al haber aprendido la relación entre las letras, predice la secuencia de caracteres que luego va a formar una frase. Mind blown.

El codigo de TFLearn permite simplificar muchísimo la declaración de la red neuronal
El entrenamiento no fue muy grande (~1 día ½)y el set de datos es muy muy pequeño con lo cual los resultados fueron buenos a veces — pero no perfectos — claramente es fácil darse cuenta que no estas hablando con una persona, muchas frases terminan siendo inconexas. Probablemente una forma de entrenamiento mejor hubiese sido entrenar la red basándola en conversaciones reales, en lugar de frases aisladas.
Acá esta el código del Bot (version neuronal) básicamente la idea es correr el script bot-train.py por unos días y eso va a generar el modelo y guardarlo en el disco (model.ftl). A medida que lo entrenas podes ver el uso del GPU con el comando `nvidia-smi` y con htop el uso de CPU.
El script bot-predict.py se conecta a tuiter con las credenciales que ingreses en el archivo .twitter y responde basado en ese modelo.
La instancia de Amazon que use fue con el AMI (imagen pre-configurada) que tenia ya instalados Python, Tensorflow, etc. La pueden encontrar acá: “Deep Learning AMI Amazon Linux Version” https://aws.amazon.com/marketplace/pp/B01M0AXXQB
¿Hacemos un bot de Fort?
Después de ese experimento se me ocurrió usar el set de datos de tuits de Ricardo Fort ya que el siempre tuiteo de una manera muy general y particular.
Para eso baje todo el archivo de tuits e hice lo mismo que con mi Bot, el problema es que la cantidad de texto de él era mucho menor (solo ~11k tuits), con lo cual termine usando algo mucho mas barato de generar (no hace falta entrenamiento de días) usando “Markov Chains”, encontré una librería de Python que las genera de forma fácil con lo cual solo tuve que modificar el script del Bot original mínimamente. Los resultados usando Markov son relativamente buenos y muchísimo mas rápidos.
Aca esta todo el código de los tres ejemplos que es bastante simple, de hecho es el primer codigo de Python que escribo.
