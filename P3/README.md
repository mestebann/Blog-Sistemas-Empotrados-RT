# BLOG P3

## INTRODUCCIÓN

El objetivo de esta práctica ha sido el desarrollo de una máquina expendedora basada en Arduino UNO y en distintos sensores y actuadores. 

## RECURSOS UTILIZADOS

Para el software de la práctica he utilizado la librería Thread, a partir de la cual he usado dos threads principalmente. También he utilizado interrupciones hardware a través de un botón independiente y el botón del joystick. Además, he implementado Watchdog para evitar bloqueos. 

```python
ThreadController controller = ThreadController();
Thread ledThread = Thread();
Thread servicioThread = Thread();
```

- HILO 1 (ledThread): este primer thread lo he utilizado para la funcionalidad del parpadeo del LED 1, esta funcionalidad del parpadeo se realiza en la función `parpadeoLed()` en la que cuando acaba la función, activa el siguiente hilo.
  ```python
  servicioThread.enabled = true;
  ```

- HILO 2 (servicioThread): este segundo hilo es el que hace la funcionalidad del resto del programa, ya que se encarga tanto del apartado 'Servicio' como del apartado 'Admin'. Este hilo maneja dos funciones secuencialmente, primero detecta si hay o no presencia de cliente en `detectarCliente()` donde también mide la distancia del ultrasonido en la función `mediDistancia()`.
La segunda función que maneja este hilo es `servicio()`, donde se aplica toda la funcionalidad del apartado 'Servicio' y 'Admin'. Este hilo comienza ejecutando la función anterior hasta que en un momento dado (si se pulsa el botón más de 5 segundos) se pasa a la función `adminMenu()`:
  ```python
  if (adminActivo) {
    digitalWrite(LED1, HIGH);    
    analogWrite(LED2, 255);        
    
    adminMenu();

    wdt_reset();
    return;
  }
  ```
- INTERRUPCIÓN HARDWARE: ha sido ncesario utilizar interrupciones tanto en el botón del joystick como en un botón independiente. Los dos botones se manejan mediante interrupción PULLUP:
  ```python
  pinMode(JOY_BOTON, INPUT_PULLUP);
  pinMode(PIN_BOTON, INPUT_PULLUP);
  ```
  Además, el botón independiente se maneja maneja:
  ```python
  attachInterrupt(digitalPinToInterrupt(PIN_BOTON), isrBoton, CHANGE)
  ```
  Dentro de la ISR solo se hace lo estrictamente necesario:
  ```python
  void isrBoton() {
    bool estado = digitalRead(PIN_BOTON);

    if (estado == LOW) { // botón presionado
    botonPresionado = true;
    } 

    else { // botón soltado
      botonPresionado = false;
    }
    wdt_reset();
  }
  ```

- USO DEL WATCHDOG: he utilizado el watchdog en todo el programa para así evitar bloqueos ya que el uso de threads y sensores externos aumentaba el riesgo. El watchdog permite reiniciar la placa Arduino si alguna parte del código deja de responder. Primero activo el watchdog con un tiempo de 2 segundos:
  ```python
  wdt_enable(WDTO_2S);
  ```
  Un ejemplo del uso del watchdog en un bucle:
  ```python
  // Esperar 2 segundos
  long inicio = millis();
  while(millis() - inicio < 2000){
  wdt_reset();
  }
  ```

## PROBLEMAS Y SOLUCIONES

Durante la práctica he encontrado diversos problemas, a continuación expongo los distintos problemas que me han surgido y sus respectivas soluciones.

- BOTÓN INDEPENDIENTE: este primer problema fue un error mío de hardware, ya que conecté mal los cables al botón independiente, por lo que el botón no cambiaba de estado al pulsarlo. El error estaba en una confusión entre filas del botón. La solución fue por prueba y error en la colocación de los cables. Esto lo probé mediante un código sencillo que sólo detectase pulsación.

- JOYSTICK INVERTIDO: en varias partes del código leía
  ```python
  int joyX = analogRead(JOY_X);
  int joyY = analogRead(JOY_Y);
  ```
  Esto provocaba un movimiento incorrecto en los menús ya que me invertía los ejes. La solución fue simplemente modificar los ejes para que el movimiento en los menús fuese arriba/abajo en vez de izquierda/derecha. Solución:
  ```python
  int joyY = analogRead(JOY_X);
  int joyX = analogRead(JOY_Y);
  ```

## ESQUEMÁTICO


## VÍDEO









  
