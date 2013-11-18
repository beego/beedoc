# FAQ

1. Cannot find template files, configuration files or nil pointer error?

	This is because you use `go run main.go` to run the App, where `go run` puts your App in temporary file, and beego loads these information in current directory; therefore, you have to use `go build` and then use `./app` to run the App, or simply use bee tool `bee run app`.

1. Can beego be used in product environment?

	A lot of big users are using beego in product environment, including CDN system of SNDA, 360 search API service for Youdao, Bmob mobile cloud API service, weico back-end API service, etc. All of them are high-concurrency and high-performance.
	
1. Will beego breaks current APIs in future update?

	beego keeps stability from version 0.1 to now, most of Apps are able to update beego without any modification.
	
1. Will beego have enough power to go further?

	Many people wrong about that open source project may not have enough power to keep going. For now, beego has 5 developers in the develop group, you should believe, follow and join us.