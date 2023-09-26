---
title: "systemd - Service"
date: 2023-09-25T16:20:13+02:00
draft: false
---

# Services under systemd

It's been some time since my last post, I figured why not have a look at systemd? We'll write a simple
service just to get a look at the basic functionality. For this purpose, let's first write a Bash script
that our service can execute:

```
#!/bin/bash

while true
do
        echo Current time: $(date)
        sleep 5
done

```

This will, in an endless loop, echo the current date, including the time, then sleep for 5 seconds.
Save this as as sysdservice.sh (or whatever you want to name it).
Make our script executable with 'chmod +x sysdservice.sh'. Go ahead and run the script to try for yourself!

Now let's get to the part we are here for, our systemd service! First of all, our service will go into
/etc/systemd/system/, so let's create one from scratch:

```
sudo touch /etc/systemd/system/echotime.service

```

As you can see, services are named servicename.service! Our service file will be completely empty, since we just
created it without adding anything. Let's start by adding the three basic sections of a service and show some possible
parameters under each of them:

```
[Unit]
Description=Print the current date to the console
After=network.target

[Service]
ExecStart=/home/ev0/sysdservice.sh
Type=simple

[Install]
WantedBy=multi-user.target
```

In our example, the Unit section has the Description and After parameters. The Description parameter
is just a short description of the service, the After parameter specifies when our service can execute!
If you put, for example, the network service under the After parameter, then our service will only execute after
the network service is up and never before that.

The Service section (which is the only section strictly needed!) features the ExecStart parameter,
which is the command that will get executed by the service. Technically, our service could have only consisted of

```
[Service]
ExecStart=/home/ev0/sysdservice.sh
```

and it would have run fine! The Type parameter specifies the type of process to spawn.

The Install section has, in our example, only one paramter, WantedBy. It means that the service should execute
when a specific target during the boot process is reached. The most common value is multi-user.target, meaning
the system is fully booted.

There are a lot of parameters available, for a complete overview please refer to the man pages on your system
or, if you prefer the online version, found [here](https://www.freedesktop.org/wiki/Software/systemd/?ref=linuxhandbook.com).
[Some](https://www.freedesktop.org/software/systemd/man/systemd.service.html)
[more](https://www.freedesktop.org/software/systemd/man/systemd.exec.html)
[links](https://www.freedesktop.org/software/systemd/man/index.html) that directly lead you to the relevant man pages.

Now let's actually run our service instead of invoking our script manually. First we start our system and
then check it's status:

```
sudo systemctl start echotime
sudo systemctl status echotime
```

You'll see an output like this:

```
● echotime.service - Print the current date to the console
     Loaded: loaded (/etc/systemd/system/echotime.service; disabled; preset: enabled)
     Active: active (running) since Mon 2023-09-25 17:02:52 CEST; 1s ago
   Main PID: 21481 (sysdservice.sh)
      Tasks: 2 (limit: 19007)
     Memory: 544.0K
        CPU: 3ms
     CGroup: /system.slice/echotime.service
             ├─21481 /bin/bash /home/ev0/sysdservice.sh
             └─21483 sleep 5

Sep 25 17:02:52 ev0-MS-7C56 systemd[1]: Started Print the current date to the console.
Sep 25 17:02:52 ev0-MS-7C56 sysdservice.sh[21481]: Current time: Mo 25. Sep 17:02:52 CEST 2023
```

We can see some information about whether our service is loaded and/or active, it's process ID, it's description and
the output of our service itself in the end. If you wait for a minute and execute the status command again, the output
will be longer, since our service is executing every five seconds. If you are just interested in the output of our
service, you can take a look at the continuous output by running the command

```
sudo journalctl -u echotime -f
```

If we want to stop our service, simply run

```
sudo systemctl stop echotime
```

and check the status again, the Active parameter will just show 'inactive (dead)' and most of the information will
be missing, since the process isn't active it won't have a PID, memory consumption and so on. Now, we could always
start and stop our service like this manually, but if we want to start it automatically when our system boots, we have
to enable it with:

```
sudo systemctl enable echotime
```

Disable the service with, you guessed it, 'sudo systemctl disable echotime' and if you edit the service file you can
run 'sudo systemctl daemon-reload'.

And that's already it for our very basic introduction to services under systemd. As you have seen on the man pages,
there are a lot of possibilites to configure your services for more complex use cases but this little tutorial
was a gentle first step to get you going.
