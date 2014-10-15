# Bee Watch

[![Build Status](https://drone.io/github.com/beego/beewatch/status.png)](https://drone.io/github.com/beego/beewatch/latest)

Bee Watch es un debugger interactivo para el lenguaje de programación Go.

## Características

- Usa los tres niveles `Critical`, `Info` y `Trace` para cambiar el comportamiento del debugger.
- `Display()` (muestra) los valores de las variables e `Printf()` (imprime) con formatos personalizados.
- Interfaz de usuario Web muy amigable para modo **WebSocket** o modo easy-control (control fácil) **command-line**.
- Llama a `AddWatchVars()` para monitorear variables y mostrar su información cuando el programa llama a `Break()`.
- Archivo de configuración personalizable(`beewatch.json`).

## Instalación

Bee Watch en un Proyecto Go habilitado vía "go get", puedes ejecutarlo con el sisguiente comando para autoinstalarlo:

	go get github.com/beego/beewatch

**Atención** Este proyecto solo puede ser instaldo por códidgo de fuente por ahora.

## Incio rápido

### Uso

	package main

	import (
		"time"

		"github.com/beego/beewatch"
	)

	func main() {
		// Inicia con el nivel por defecto: Trace,
		// o usa `beewatch.Start(beewatch.Info())` para ver el nivel.
		beewatch.Start()

		// Algunas varialbes.
		appName := "Bee Watch"
		boolean := true
		number := 3862
		floatNum := 3.1415926
		test := "fail to watch"

		// Agrega variavles para ser vistas, deben ser de tamaño par.
		// Nota que tienes que pasar punteros.
		// Por ahora solo soporta tipos de datos básicos.
		// En este caso, 'test' no será visto.
		beewatch.AddWatchVars("test", test, "App Name", &appName,
			"bool", &boolean, "number", &number, "float", &floatNum)

		// Muestra algo.
		beewatch.Info().Display("App Name", appName).Break().
			Printf("valor booleano es %v; número es %d", boolean, number)

		beewatch.Critical().Break().Display("float", floatNum)

		// Cambia algunos valores, deben ser de tamaño par.
		appName = "Bee Watch2"
		number = 250
		// Aquí verás algo interesante.
		beewatch.Trace().Break()

		// Múltiple testeo de goroutines.
		for i := 0; i < 3; i++ {
			go multipletest(i)
		}

		beewatch.Trace().Printf("Espera 3 segundos...")
		select {
		case <-time.After(3 * time.Second):
			beewatch.Trace().Printf("Debug terminado")
		}
	
		// Cerrar debugger,
		// No es útil cuando haces debug un servidor web que no tiene un fin.
		beewatch.Close()
	}

	func multipletest(num int) {
		beewatch.Critical().Break().Display("num", num)
	}

### Connect

El debugger de Bee Watch es iniciado automáticamente en [http://localhost:23456](http://localhost:23456) cuando usas el modo **WebSocket**, puedes cambiar el puerto y otras parámetros de configuración editando `beewatch.json`(copia de la configuración por defecto desde la carpeta de fuente de Bee Watch).

Tu navegador debe soportar WebSocket. Bee Wathc ha sido testiado con Chrome, Safari y Firefox en Mac y Windows.

![Demo de Bee Watch](https://github.com/beego/beewatch/blob/master/tests/images/demo_beewatch.png?raw=true)

## Ejemplos y documentación de la API

[Go Walker](http://gowalker.org/github.com/beego/beewatch)

