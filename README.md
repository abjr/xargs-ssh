xargs-ssh
=============

This simple Linux bash script runs multiple ssh commands in parallel by leveraging the parallel processing power of xargs. I first learned of this trick on [Jordan Sissel's blog](http://www.semicomplete.com/blog/articles/week-of-unix-tools/day-5-xargs.html).

There are already a number of tools available to run ssh in parallel, such as
* [pssh](http://www.theether.org/pssh/)
* [sshpt](http://code.google.com/p/sshpt/)
* [ClusterSSH](http://clusterssh.sourceforge.net/)
* [Capistrano](http://capistranorb.com/)
* [Fabric](http://fabfile.org/)
* [Func](https://fedorahosted.org/func/)
* [MCollective](https://puppetlabs.com/mcollective/)

These tools all have numerous advantages and features that this simple script doesn't. The single advantage that xarg-ssh has over these alternatives is that it's just a bash script that you can run without having to install anything new on either server or client.

At the heart of this script is this command
````bash
xargs -a $serverlist -I"SERVER" -P${threads} -n1 sh -c "ssh SERVER $cmd"
````
* $serverlist is a file containing a newline-separated list of hostnames.
* ${threads} is the maximum number of threads you want it to run simultaneously at any given time.
* $cmd is the command to be run on each server.

If this script is a bit much for you, you could distill it down to just that one line. I recommend wrapping the ssh part in a command substitution in some sort of printf/echo statement if the output matters.

Dependencies:
-------------
This script only works on Linux. The following tools are needed to run xargs-ssh.bash:
* bash
* openssh-client (also known as "ssh")
* xargs
* sed
* awk
* tput
* find
* POSIX shell (also known as "sh")
* printf

These are all tools that are commonly found on any GNU/Linux system.  

The versions of xargs and mktemp on OSX (the BSD versions) behave very differently to their Linux counterparts. I could adapt this script to work on OSX, but since I personally intend to use this only on Linux (I don't even own a Mac), I probably won't.  

In case you want to do it, here are some tips to help you on your way:  
Counting the cores on OSX is done like this:
````bash
sysctl hw.ncpu | awk '{print $2}
````
BSD mktemp requires a bit more effort to create a random file in /tmp/, but that probably doesn't even matter: BSD xargs doesn't have an option to read input from a file. The way to go here is to pipe your data straight to xargs.

Usage:
------
Either pipe a line-separated list of servers to xargs-ssh, or provide a file with the list with the -f option. You may optionally specifiy a command to run with the -c option and enable easily parsable output with the -s option.
If server list is not provided, no -f option neither pipe, the script runs the command interactively as each server is typed (end with Ctrl-D).

<table>
  <tr>
    <th>Option</th><th>argument</th><th>Description</th>
  </tr>
  <tr>
    <td>-f</td><td>filename</td><td>specifies the input file. Use -f filename to read from a file. If no file is specified, stdin is used.</td>
  </tr>
  <tr>
    <td>-c</td><td>command</td><td>lets you specify a command to run on each server. By default this command is "uptime".</td>
  </tr>
  <tr>
    <td>-s</td><td>n/a</td><td>enables script mode, which keeps the output for each server on one line, new lines in the output are separated by "\n":</td>
  </tr>
  <tr>
    <td>-h</td><td>n/a</td><td>prints help message that explains how to use the script.</td>
  </tr>
</table>

Examples:
---------
Basic usage of the script using a list of servers in a file:
````bash
./xargs-ssh.bash -f /home/kenny/serverlist.txt
````
An example of piping data to the script:
````bash
for i in {1..4}; do echo server$i; done | ./xargs-ssh.bash
````
These two commands would have the same output:
<pre>
###### server3 ######
 11:31:22 up 34 days, 17:09,  2 users,  load average: 0.00, 0.00, 0.00

###### server4 ######
 11:31:22 up 153 days, 22:44,  1 user,  load average: 0.00, 0.00, 0.00

###### server2 ######
 11:31:22 up 34 days, 17:13,  8 users,  load average: 0.12, 0.11, 0.05

###### server1 ######
 05:28:35 up 76 days, 12:29,  0 users,  load average: 0.00, 0.00, 0.00
</pre>

Specify a custom command and enable script mode:
````bash
./xargs-ssh.bash -f /home/kenny/serverlist.txt -c "hostname --fqdn && uptime" -s
````
The output of each individual ssh reduced to one line of output, where new lines in the output are separated by "\n":
<pre>
server4: server4.customer.local\n 11:31:22 up 153 days, 22:44,  1 user,  load average: 0.00, 0.00, 0.00
server1: server1.customer.local\n 05:28:35 up 76 days, 12:29,  0 users,  load average: 0.00, 0.00, 0.00
server2: server2.customer.local\n 11:31:22 up 34 days, 17:13,  8 users,  load average: 0.12, 0.11, 0.05
server3: server3.customer.local\n 11:31:22 up 34 days, 17:09,  2 users,  load average: 0.00, 0.00, 0.00
</pre>
