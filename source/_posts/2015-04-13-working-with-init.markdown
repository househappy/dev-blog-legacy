---
layout: post
title: "working with init"
date: 2015-04-13 10:05:01 -0700
comments: true
categories:
published: false 
---
##Create a linux startup service and init.d script from scratch, using chef and debian linux. 

By C. Thomas Clark https://github.com/cthomaspdx

As a Devops engineer at Househappy.org, I have been tasked to help maintain up-time for our servers and services they provide. If a server does have to reboot unexpectedly it would be nice to have some kind of assurance that when our servers come back online they automatically start their services. 

Case in point: Our cloud server provider linode, very subtle notified us that our servers were due for an update and needed to be rebooted inorder for the update to take effect. Of course they formatted the email to look almost identical to a lot of the other email noise they send us, and I failed to recognize that this was serious and need my urgent attention. So time passed and the weekend rolled around, and all of our pager services started notifing us of our impending doom. One of the major problem with this is that many of our homegrown services with out init.d script failed to start back up after reboot. I would like to not have to worry about that happening again, so I decided to do my research. 

Understanding "Sys-v" style init scripts and runlevels 

The `/etc` dir contains a set of directories named `/rc?.d` where `?` is their runlevel integers 1-6.  A runlevel is basically the state of the system. i.e. `/etc/rc0.d` is the Halt state or shutdown of the system; while runlevel 1 is "single user" or "rescue mode" where only the most essential services are ran. Runlevel 6 `/etc/rc6.d ` is the reboot runlevel and is the same as halt `rc0.d` but with a reboot call at the end. So this leaves us to play with runlevels 2-5 for multi-user system start up configurations. 

You can check which runlevel you are on by runing `/sbin/runlevel`  the second character it returns is the runlevel. 

Take a look at what a the directory `/etc/rc2.d` for runlevel 2 contains. Run `ls -la /etc/rc2.d` 
I can see that it contains a bunch of symlinks to `/etc/init.d/service_name` these symlinks have prefixes of S or K (you may not have an example of K##service in the rc2.d) followed by two intergers 01, 02...99 and are excuted in alphabetical first, so K(kills) are run before S(starts), then numeric order the lowest number being the first service to start. Higher level services depend on other services to be running so the depencies need to start first and in turn a lower number number following the S or K in the symlink. So when the run level changes or on boot, init sends the signal `start` to the init script in order.

     S01vboxadd -> ../init.d/vboxadd
     S02vboxadd-service -> ../init.d/vboxadd-service
     S13rpcbind -> ../init.d/rpcbind
     S14nfs-common -> ../init.d/nfs-common
     S16chef-client -> ../init.d/chef-client
     S16datadog-agent -> ../init.d/datadog-agent
     S16elasticsearch -> ../init.d/elasticsearch
     S16rsyslog -> ../init.d/rsyslog
     S16sudo -> ../init.d/sudo

 Here is a great article about runlevels if you would like more detail. https://www.debian-administration.org/article/212/An_introduction_to_run-levels

 So with an understanding of how the symlink are called let's take a look at how they are generated. 
 If we open up any `/etc/init.d/ ` init script, we can see that they contain a header `### BEGIN INIT INFO`. 
 This one is for elasticsearch.
 


    ### BEGIN INIT INFO
    # Provides:          (name of the service)
    # Required-Start:    (services that should start before this does.)
    # Required-Stop:     (services that should stoped after this service is stopped.)
    # Default-Start:     2 3 4 5 (these are the runlevels in which it should be started)
    # Default-Stop:      0 1 6  (these are the runleves in which it should be stopped(halt, safe-mode and reboot).)
    # Short-Description: Starts elasticsearch
    # Description:       Starts elasticsearch using start-stop-daemon
    ### END INIT INFO


    ### BEGIN INIT INFO
    # Provides:          elasticsearch
    # Required-Start:    $network $remote_fs $named
    # Required-Stop:     $network $remote_fs $named
    # Default-Start:     2 3 4 5
    # Default-Stop:      0 1 6
    # Short-Description: Starts elasticsearch
    # Description:       Starts elasticsearch using start-stop-daemon
    ### END INIT INFO`


run `update-rc.d <basename> default` and the symlinks in the rc?.d files will be created according to INIT INFO and you are off to the races. 

For more info check `man update-rc.d` 


##Chef setup 

Here is an example of an init.d scrip we added for our geoip sevice.

Add a a template for the init.d script.

Here is a link to a nice init.d script template https://github.com/fhd/init-script-template

Add the template to the `/etc/init.d/` directory.

    template "/etc/init.d/geoip" do
      source 'init.d.geoip.erb'
      mode 0755
    end

Then start the service.

    service 'geoip' do 
      supports :status => true, :start => true, :stop => true, :restart => true
      action [:enable, :start]
    end

Service will update-rc.d with `action :enable`











