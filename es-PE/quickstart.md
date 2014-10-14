# Iniciando

## Instalation

Beego contiene ejemplos de aplicaciones para ayudarte a aprender y usar el framework beego.

Necesitarás tener un Go 1.1 instalado para este trabajo.

Necesitarás instala Beego y la herramienta de desarrollo [Bee](http://beego.me/docs/install/bee.md: 
	
	$ go get github.com/astaxie/beego
	$ go get github.com/beego/bee


Por conveniencia, deberás agregar `$GOPATH/bin` a tu variable de entorno `$PATH`.

¿Quieres empezar a ver ya como funciona? Entonces lo siguiente:

	$ cd $GOPATH/src
	$ bee new hola
	$ cd hola
	$ bee run

Usuarios de Windows：

    > cd %GOPATH%/src
    > bee new hola
    > cd hola
    > bee run hola

Estos comandos te ayudan a:

1. Installa beego dentro de tu `$GOPATH`
2. Instalar la herramienta de desarrollo Bee en tu computadora.
3. Crear una nueva aplicación llamada `hola`.
4. Inicia la compilación.

Una vez corriendo, abre tu navegaro en [http://localhost:8080/](http://localhost:8080/).

## Ejemplo simple

El siguiente ejemplo imprime `Hola Mundo` en tu navegador, muestra cuan fácil es construir una applicación web con beego.

	package main
	
	import (
		"github.com/astaxie/beego"
	)
	
	type MainController struct {
		beego.Controller
	}
	
	func (this *MainController) Get() {
		this.Ctx.WriteString("hola mundo")
	}
	
	func main() {
		beego.Router("/", &MainController{})
		beego.Run()
	}

Save file as `hola.go`, build and run it:

	$ go build -o hola hola.go
	$ ./hola

Abre [http://127.0.0.1:8080](http://127.0.0.1:8080) en tu navegador y verás `hola mundo`.

¿Qué está pasando en el escenario del ejemplo de arriba?

1. Importamos el paquete `github.com/astaxie/beego`. Como sabemos, Go inicializa los paquetes y ejecuta init() en cada paquete ([más detalles](https://github.com/Unknwon/build-web-application-with-golang_EN/blob/master/eBook/02.3.md#main-function-and-init-function)), por lo que beego inicializa la la aplicación `BeeApp` en este momento.
2. Define el controlador. Definimos una estructura llamada `MainController` con un campo anónimo `beego.Controller`, entonces `MainController` tiene todos los métodos que `beego.Controller` tiene.
3. Define algunos métodos RESTful. Dado el campo anónimo de arriba, `MainController` ya tiene `Get`, `Post`, `Delete`, `Put` y otros métodos, estos métodos serán llamados cuando el usuario envíe una consulta correspondiente (por ejemplo, el método `Post` es llamado para manipular las consultas usando POST. Por lo tanto, después de sobrecargar el método `Get` en `MainController`, todas las consultas GET usarán ese método en `MainController` en lugar de `beego.Controller`.
4. Define la función main. Todas las aplicaciones en Go usan `main` como su punto de entrada tal como C.
5. Registra las rutas. Esto le dice a beego que controlador es responsable para una petición específica. Aquí nosostros registramos MainController` para `/`, por lo que todas las peticiones a `/` serán manipuladas por MainController`. Se consciene que el primer argumento es la ruta y el segundo es un puntero al controlador que quieres registrar.
6. Corre la aplicación en el puerto 8080 por defecto, preciona `Ctrl+c` para salir.

Los siguientes archivos `.bat` con accesos directos para ususarios de Windows:

Crea archivos `step1.install-bee.bat` y `step2.new-beego-app.bat` bajo `%GOPATH%/src`.

`step1.install-bee.bat`:

	set GOPATH=%~dp0..
	go build github.com\beego\bee
	copy bee.exe %GOPATH%\bin\bee.exe
	del bee.exe
	pause

`step2.new-beego-app.bat`:

	@echo Set value of APP same as your app folder
	set APP=coscms.com
	set GOPATH=%~dp0..
	set BEE=%GOPATH%\bin\bee
	%BEE% new %APP%
	cd %APP%
	echo %BEE% run %APP%.exe > run.bat
	echo pause >> run.bat
	start run.bat
	pause
	start http://127.0.0.1:8080

Haz click en esos dos archivos en orden y tendrás un inicio rápido en un tour en beego. Y solo tendrás que correr `run.bat` en el futuro.
