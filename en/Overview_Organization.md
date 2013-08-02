# Organization

Beego requires itself and the user application to be installed into a GOPATH layout as prescribed by the go command line tool. (See “GOPATH Environment Variable” in the go command documentation)

### Example layout

Here is the default layout of a Beego application called sample, within a typical Go installation.

	$GOPATH                         GOPATH root
	  src                   		GOPATH src directory
	    github.com/astaxie/beego	Beego source code
	      ...
	    sample              App root
		    main.go         Main package files
		    controllers     App controllers
		    models          App domain models
		    views           Templates
			conf            Configuration files
	        	app.conf    Main configuration file
	      	static          Static assets
		        css         CSS files
		        js          Javascript files
		        img         Image files

This is a sample project structure, you have freedom to change these settings, for example, you can name controllers to "routers".