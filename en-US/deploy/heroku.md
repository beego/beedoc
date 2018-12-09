---
name: Deployment to Heroku
sort: 5
---

# Heroku
 
## Deploy beego application to heroku

1. Add dependencies to vendor directory with [govendor](https://github.com/kardianos/govendor)
```
$ govendor init
```

2. Add heroku configurations to ``vendor/vendor.json`` file
```
{
  ...,
  "heroku": {
    "install" : [ "./..." ],
    "goVersion": "go1.11"
  },
  ....
}
```
There is a information about this configuration on heroku docs in [here](https://devcenter.heroku.com/articles/go-dependencies-via-govendor#build-configuration)

3. Download dependencies

```
$ govendor fetch github.com/astaxie/beego 
```
It will download the beego packages to ``./vendor`` directory 

```
$ git add vendor/
```
4. Configure default runmode

Default config must be ``runmode = prod`` in ``conf/app.conf``

5. Configure listen ports for heroku in ``main.go``
```go
func main() {
    log.Println("Env $PORT :", os.Getenv("PORT"))
    if os.Getenv("PORT") != "" {
        port, err := strconv.Atoi(os.Getenv("PORT"))
        if err != nil {
            log.Fatal(err)
            log.Fatal("$PORT must be set")
        }
        log.Println("port : ", port)
        beego.BConfig.Listen.HTTPPort = port
        beego.BConfig.Listen.HTTPSPort = port
    }
    if os.Getenv("BEEGO_ENV") != "" {
        log.Println("Env $BEEGO_ENV :", os.Getenv("BEEGO_ENV"))
        beego.BConfig.RunMode = os.Getenv("BEEGO_ENV")
    }

    beego.Run()
}
```

You can use below command to continue develop your app without change default runmode again
```
$ BEEGO_ENV=dev bee run
```
