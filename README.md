<div style="text-align: justify">
<center>

![Build Status](https://drive.google.com/uc?export=view&id=1D30zy4k_GYXKbG4EveSlM7d-czKEbK6b)


</center>

# Creación de una extensión para VS Code

## Instalación de herramientas y paquetes

Para crear la extensión se hace uso de algunas herramientas y paquetes, entre ellos mencionamos:

* VS Code
* Node
* Yeoman
* Generator-code

### VS Code

Para instalar VS Code es necesario descargar los archivos de instalación de la página oficial [https://code.visualstudio.com/#alt-downloads](https://code.visualstudio.com/#alt-downloads), se selecciona el instalador de acuerdo al sistema operativo, en este caso se utilizó Windows 64 bit.
<center>

[![Build Status](https://drive.google.com/uc?export=view&id=1LVcmvuomxEFWML5AiCvdXFTKsfXtrY0i)](https://code.visualstudio.com/#alt-downloads)


</center>
<center>

Figura 1. Fuente: [https://code.visualstudio.com/#alt-downloads](https://code.visualstudio.com/#alt-downloads)
</center>

Al completar la descarga se extraen los archivos del .zip comprimido se ejecuta el archivo Setup y se siguen el asistente de instalación. Una vez completada la instalación podremos ejecutar VS Code.

### Node
Node.js es un entorno de ejecución de JavaScript para descargarlo es necesario visitar la página [https://nodejs.org/es/](https://nodejs.org/es/) donde descargaremos el instalador de acuerdo al sistema operativo del computador donde se instalará, en este caso Windows 10 (x64), obtenemos un archivo con extensión .msi y al ejecutarlo abrirá el asistente para la instalación una vez concluido el proceso podemos comprobar la versión instalada desde el cmd con el comando:

```
node -v
```


<center>

![Build Status](https://drive.google.com/uc?export=view&id=1BdBo5_23u4eR4iSVHXX9dlyWYYyjvAs6)


</center>
<center>

Figura 2. Fuente: Elaboración propia
</center>

Al instalar Node tendremos acceso a su repositorio de paquetes NPM que nos permitirá instalar y utilizar los siguientes paquetes.


### Yeoman y Generator-code

Yeoman es una aplicación que permite crear la estructura básica de los ficheros y directorios para el desarrollo de un proyecto web, Generator-code es un generador para Yeoman que permite generar la estructura para crear una extensión para VS Code.

Para realizar la instalación de Yeoman y Generator-code abrimos el cmd y ejecutamos el siguiente comando:


```
npm install -g yo generator-code typescript
```

Una vez finalizada la instalación estamos listos para crear la estructura de la extensión, para esto en el cmd ejecutamos el siguiente comando:


```
yo code
```

Este comando inicia la generación de la estructura del proyecto y agrega las dependencias necesarias para su ejecución. Este proceso requiere parametros iniciales, los parametros utilizados son los siguientes.

* **What type of extension do you want to create?** Utilizando las fechas se selecciona *New Extension (TypeScript)* indicando que la extensión de desarrollará utilizando TypeScript.
* **What's the name of your extension?** Establecemos el nombre *Line Gapper*
* **What's the identifier of your extension?** Necesitamos un identificador para la extensión y este será *gapline*
* **What's the description of your extension?** La descripción es opcional  y en caso de que la extensión sea publicada esta será la descripción que se mostrará *gapline extension*
* **Initialize a git repository?** Debido a que este proyecto no estará vinculado con un repositorio de git seleccionamos *No*
* **Bundle the source code with webpack?** No necesitamos vincular el código fuente por lo que seleccionamos *No*
* **Which package manager to use?** Nos pregunta que manejador de paquetes utilizaremos, este será *npm* que hemos instalado con anterioridad.


<center>

![Build Status](https://drive.google.com/uc?export=view&id=1RT57XTWhu7ZCfCwwRoIDBRdCSoEvQsRL)


</center>
<center>

Figura 3. Fuente: Elaboración propia
</center>

Una vez finalizada la creación de la estructura del proyecto podemos comenzar a editarla para que funcione como deseamos.

# Creación de la extensión Line Gapper
## extension\.ts

Comenzaremos editando el fichero **extension.ts** este se localiza dentro del la carpeta **src** en la estructura de la aplicación, en este fichero eliminamos el contenido e insertamos el siguiente código:

~~~typescript
'use strict';
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
    let disposable = vscode.commands.registerCommand('extension.gapline', () => {
        var editor = vscode.window.activeTextEditor;
        if (!editor) { return; }
        var selection = editor.selection;
        var text = editor.document.getText(selection);
        vscode.window.showInputBox({ prompt: 'Lineas?' }).then(value => {
            let numberOfLines = +value;
            var textInChunks: Array<string> = [];
            text.split('\n').forEach((currentLine: string, lineIndex) => {
                textInChunks.push(currentLine);
                if ((lineIndex+1) % numberOfLines === 0) {textInChunks.push('');}
            });
            text = textInChunks.join('\n');
            editor.edit((editBuilder) => {
                var range = new vscode.Range(
                    selection.start.line, 0, 
                    selection.end.line,
                    editor.document.lineAt(selection.end.line).text.length
                );
                editBuilder.replace(range, text);
            });
        })
    });
    context.subscriptions.push(disposable);
}

export function deactivate() { }
~~~

## tsconfig\.json

En el fichero *tsconfig.json* debe tener el elemento **strict** en **false**, de no existir el elemento dentro del json se debe agregar. 

~~~json
"compilerOptions": {
    //Agregar el atribituo strict en falso
    "strict": false,  
~~~

## package\.json

En el fichero *package.json* es necesario cambiar algunos valores para obtener la funcionalidad deseada. 

En el elemento **activationEvents** es necesario cambiar *helloWorld* por *gapline*.

~~~json
//Fichero package.json original.
"activationEvents": [
        "onCommand:gapline.helloWorld"
	],
~~~
Luego de realizar el cambio.
~~~json
//Fichero package.json editado.
"activationEvents": [
        "onCommand:extension.gapline"
    ],
~~~
Tambien en el elemento **command** y **title** colocando el command el identificador de la extensión, en este caso **extension.gapeline** y en title colocamos el título que damos a la extensión **GapLine**

~~~json
"contributes": {
		"commands": [
			{
                //Command y title original
				"command": "gapline.helloWorld",
				"title": "Hello World"
			}
		]
	},
~~~

Luego de realizar el cambio

~~~json
"contributes": {
        "commands": [
        {
            //Command y title editado
            "command": "extension.gapline",
            "title": "GapLine"
        }
        ]
    },
~~~

# Anatomía de la extensión
## extension\.ts

Para analizar y explicar el código utilizado en la extensión repasaremos primero las funciones básicas que utilizaremos en la extensión, el fichero extension.ts exporta dos funciones **activate** y **deactivate**.

### Función activate
La función **activate** es ejecutada cuando el **Activation event** (Evento de activación) es disparado.

~~~typescript
export function activate(context: vscode.ExtensionContext) {
   //Se coloca el código que ejecutará la función.
}
~~~

#### **Activation events**
Estos eventos indican que cual será el disparador de la extensión que iniciará si ejecución, el evento es configurado en el fichero *package.json*. El evento utilizado para esta extensión es el evento **onCommand**.

~~~json
"activationEvents": [
    /*Se utiliza el evento onCommand para ejecutar la extensión al momento de incovar el comando*/
        "onCommand:extension.gapline"
    ],
~~~

**Evento onCommand**
Este evento es disparado cuandoquiera que el que el comando es invocado.

Existen otros activation events como:

- **onLanguage:** Se dispara cuando al abrir un fichero se detecta un lenguaje especifico.

- **onDebug:** Al iniciar el debugger se dispara este evento.

- **workspaceContains:** Se determina un patrón para los ficheros y al abrir una carpeta con un fichero que concuerde con el patrón se dispara el evento.

- **onView:** Se dispara cuando una vista especificada se expande.

- **onFileSystem:** Es disparado cuando un fichero de un determinado esquema es abierto.

- **onWebviewPanel:** Cuando VS Code restaura la vista a un tipo de vista determinado.

- **onUri:** Disparado cuando el Uri para la extensión es abierto

- **onCustomEditor:** Cuando VS Code necestia crear un tipo de vista especifico para el edito.


### Función deactivate
Es utilizada cuando es necesario ejecutar alguna operación al deshabilitar o desinstalar la extensión, en las extensiones que no requieren este tipo de operación esta función se puede omitir.

~~~typescript
export function deactivate() { 
    //Se coloca el código que es ejecutará en esta función.
}
~~~

## Strict Mode
El modo estricto habilita el rango de validaciones de tipos que garantiza la corrección de tipos en la ejecución de la extensión.

~~~typescript
//Habilita el modo estricto.
'use strict';
~~~

## Importación de módulos

~~~typescript
// Importa el módulo vscode y le asigna  el alias vscode
import * as vscode from 'vscode';
~~~

## Función activate


~~~typescript
/*La función activate es ejecutada cuando se dispara el evento que activa la extensión*/
export function activate(context: vscode.ExtensionContext) {
    /*Registra comando que implementa la funcionalidad de la extensión, se debe verificar que el valor entre comillas (') sea el mismo valor que el definido en el fichero package.json*/
    let disposable = vscode.commands.registerCommand('extension.gapline', () => {
        /* Obtiene la instancia del editor de texto de la ventana de VS Code activa*/
        var editor = vscode.window.activeTextEditor;
        /*Se valida que el que se haya obtenido la instancia de un editor de texto, en caso de no obtenerlo el return termina el ejecución de la función*/
        if (!editor) { return; }
        /*La variable selection obtiene el objeto de los elementos seleccionados dentro del editor*/
        var selection = editor.selection;
        /*La variable text toma el valor del texto contenido por selection*/
        var text = editor.document.getText(selection);
        /*Se muestra el mensaje para obtener el número de lineas con la que la extensión trabajará y determina la función que se ejecutará para procesar la respuesta ingresada por el usuario*/
        vscode.window.showInputBox({ prompt: 'Lineas?' }).then(value => {
            /*La variable numberOfLines obtiene el valor de value, que es el valor que el usuario ingresa*/
            let numberOfLines = +value;
            /*Se define textInChunks como un arreglo de cadenas vacio*/
            var textInChunks: Array<string> = [];
            /*La variable text es dividida por cada salto de línea creando un arreglo, cada elemento del arreglo se procesa y currentLine va tomando el valor de cada línea durante el recorrido del arreglo completo.*/
            text.split('\n').forEach((currentLine: string, lineIndex) => {
                /*Cada línea que se procesa se agrega al final del arreglo textInChunks*/
                textInChunks.push(currentLine);
                /*lineIndex tiene el indice de la línea que se está procesando, se le suma uno porque los indices comienzan desde cero. Se compara el resto de la división y si el resto es cero se agrega una línea en blanco al arreglo*/
                if ((lineIndex+1) % numberOfLines === 0) {textInChunks.push('');}
            });
            /*text concatena todos los elementos del arreglo unidos entre se por un salto de línea*/
            text = textInChunks.join('\n');
            /*Implementa la edición del editor de texto según el siguiente bloque de código*/
            editor.edit((editBuilder) => {
                /*Se establece un nuevo rango para modificar el editor de texto*/
                var range = new vscode.Range(
                    /*Número de línea y columna inicial*/
                    selection.start.line, 0, 
                    /*Número de la línea final*/
                    selection.end.line,
                    /*Número de la columna final*/
                    editor.document.lineAt(selection.end.line).text.length
                );
                /*En el editor de texto se reemplaza todo lo que está en el rango establecido por el texto de la variable text */
                editBuilder.replace(range, text);
            });
        })
    });
    /*Agrega todos los recursos utilizados para ser descartados y limpiados al finalizar la ejecución de la extensión*/
    context.subscriptions.push(disposable);
}

~~~

## Función deactivate
Para esta extensión en particular no es necesario realizar ninguna operación en la función deactivate.

~~~typescript
export function deactivate() { }
~~~ 

# Ejecución de la extensión
Una vez concluidos los cambios en los ficheros podemos ejecutar y probar la funcionalidad de a extensión.

Teniedo abierto el proyecto de la extensión en VS Code presionamos la tecla **F5** para iniciar la ejecución, esto abrirá una nueva ventana de VS Code que tendrán la extensión instalada para utilizarla.

<center>

![Build Status](https://drive.google.com/uc?export=view&id=17jMivsqpo2lHlNfrqzs7CVeTtXCuOgHI)


</center>
<center>

Figura 4. Fuente: Elaboración propia
</center>

En la nueva ventana se necesita abrir un fichero de texto plano que contenga multiples líneas, de preferencia sin líneas vacias para notar mejor como funciona la extensión. Una vez abierto el fichero es necesario seleccionar todo el texto en el editor y pulsamos **Ctrl + Shift + P** abrir la paleta para seleccionar extensión, buscamos la extensión por el title que configuramos en el fichero *package.json*.

<center>

![Build Status](https://drive.google.com/uc?export=view&id=1ysghg2DLxMu74cR8sPmYjx9elqDWGJLC)


</center>
<center>

Figura 5. Fuente: Elaboración propia
</center>

En la paleta se debe seleccionar GapLine y nos mostrará un mensaje preguntando la cantidad de líneas en las que se agregará la línea en blanco en nuestro editor.

<center>

![Build Status](https://drive.google.com/uc?export=view&id=1SffBdGwJxksoWTL9v8_-fZ19muMl98Ia)


</center>
<center>

Figura 6. Fuente: Elaboración propia
</center>

Al confirmar el número de líenas que utilizaremos la extensión ejecutará el código que establecimos y reemplazará en el editor el texto con el nuevo formato de líneas en blanco como lo podemos apreciar a continuación.

<center>

![Build Status](https://drive.google.com/uc?export=view&id=1xAvq8xbAuusqXeBhS51XIr40m34NLrQI)


</center>
<center>

Figura 7. Fuente: Elaboración propia
</center>

</div>

# Referencias

- Microsoft, 2021. Yocode - vscode. Recuperado el 12 de mayo de 2021 de [https://vscode.readthedocs.io/en/latest/extensions/yocode](https://vscode.readthedocs.io/en/latest/extensions/yocode/)

- Microsoft, 2021. Your First Extension. Recuperado el 12 de mayo de 2021 de [https://code.visualstudio.com/api/get-started/your-first-extension](https://code.visualstudio.com/api/get-started/your-first-extension)

- Microsoft, 2021. Extension Anatomy. Recuperado el 14 de mayo de 2021 de [https://code.visualstudio.com/api/get-started/extension-anatomy](https://code.visualstudio.com/api/get-started/extension-anatomy)


- Microsoft, 2021. Activation Events. Recuperado el 14 de mayo de 2021 de [https://code.visualstudio.com/api/references/activation-events](https://code.visualstudio.com/api/references/activation-events)


- Microsoft, 2021. TypeScript: TSConfig Reference - Docs on every TSConfig option. Recuperado el 16 de mayo de 2021 de [typescriptlang.org/tsconfig#strict](typescriptlang.org/tsconfig#strict)

