---
name: Reload
sort: 3
---

# Reload

## What is reload?

What is reload? If you use nginx you probably know that nginx supports reload. It can use old process to serve old requests and use new process to serve new requests which mean it can upgrade the system and change the parameters without stop service. So reload and recompile are different concepts. Recompile is restart process by monitoring the changes and recompiling the files. `bee run` is this kind of tool.

## Is reload necessary?

Many people would ask should HTTP application support reload? The answer is: yes. Upgrade without stopping service is our goal. Many people would say the service may broken. But this is high availability issues, don't confuse of them. Actually it is a predictable issue and we should prevent it from the problem caused by reload. Are you still trobling with the upgrading after the midnight? 
Let's get started with reload!

## How does beego support reload?

The basic principle of reload is: fork a subprocess from the main process, then subprocess exec related program. So what does happen in this process? We know process forking will inherit all the handles, dada and stacks from main process to subprocess. But in the handles there is a `CloseOnExec` which is while execute `exec` if no excuse all the copied handles will be close. But what we want is subprocess can reuse `net.Listener` handle of main process. After executing `exec` function the process is "dead", system will replace the old code with new code, discard the old data and stack and reallocate new data and stack. The only thing left is the process id. For system this is the same process but indeed it's not anymore.

So the first step we need to do is let subprocess inherit main process's handle. We can assign file by param of `os.StartProcess` and write the needed handles into it.

The second step is we want the subprocess can listen from this handle. Go supports `net.FileListener`, listen by handle. But the subprocess need to know the FD, so we need to set FD in evnironment variable while start subprocess.

The third step is we want the old process keep serving and finishing their requests but the new requests use the new process. There are two concerns: the first one is the old process keep serving but how can we know we have the old process? So we need to record every requests we recieved, then we know the unfinished requests. The second one is let the old process stop serving new requests but the new process do that. They are all listening on the same port, so theoretically it's random. So we just need to close the old process after the serving. It will throw the error while `l.Accept`.

Above are the three issues we need to solve. You can check the code to see what's really going on there.

## The reload demo

1. Write the code. Get method in Beego's Controller:

		func (this *MainController) Get() {
			a, _ := this.GetInt("sleep")
			time.Sleep(time.Duration(a) * time.Second)
			this.Ctx.WriteString("ospid:" + strconv.Itoa(os.Getpid()))
		}

2. Open two terminals and input:

  In one terminal input: `ps -ef|grep application_name`

  In the other teminal input: `curl "http://127.0.0.1:8080/?sleep=20"`

3. Reload:

	`kill -HUP process ID`

4. Open a terminal and input: `"http://127.0.0.1:8080/?sleep=0"`

We can see such result: The first request wait for 20s, but it's run by the old process. After reloading, the first is still running and it will output the old process ID. But the second request will output the new process ID.

