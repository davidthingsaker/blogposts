I use Ubuntu servers all the time, and there are some commands that everyone that uses them should know. I have compiled a list of what I think are the most useful commands a Ubuntu user can know. They most likely work on most other Linux systems but I haven't tested them. 

#### Before we get started

There are some useful things to know about the command line before you get started with the below. These aren't exactly commands but are used within commands. 

 * `/` is the root directory of your system. Its used in directory paths.
 * `~` is your home directory.
 * `\` will escape a character for you. For example whitespace, if you have a space in a directory name you will need to use this before the space. E.g. `Action\ Movies`

1) `ls`

Lets start with some basics, the `ls` command simply lists the current working directory structure. You will probably also want to know some of the `ls` option keys. The most useful I find are `-l`, '-a' and `-h` which do the following:

 * `a` - will show hidden files as well  
 * `l` - will show files in a list format 
 * `h` - will show the size of the files in human readable format.

You can also view the structure of a different directory by stating the directory after your options. For example 

```shell
ls -lah downloads/movies/
```

will list the files in the `downloads/movies/` directory. 

2) `cd`

No good doing anything on command line unless you can move about within the directory structure. `cd` will change directory. 

```shell
cd downloads/movies/
```

will move you into the movies directory.

3) `mkdir`

Sticking with the directory theme, `mkdir` will make a directory. 

```shell
mkdir movies
``` 

will make a directory called movies in your current working directory. You can specify to make a directory in a different location by concatenating the location in front of the new one.

```shell
mkdir downloads/movies
``` 

will make a directory called movies in the downloads directory.

4) `rm`

`rm` will remove your unwanted files or directories

```shell
rm index.html
```

will remove the index.html file in your current location. You may also need some of the flags for `rm`, namely `-f` and `-R`.

 * '-f' - will attempt to force the removal regardless of permissions and without prompting for confirmation
 * '-R' - will remove recursively, meaning it will remove a directory and its contents including other directories. 

```shell
rm -Rf /downloads/movies
```

will recursively remove the movies directory and all of its contents without prompting for confirmation. Now you will understand the jokes and/or scams trying to get you to run `rm -Rf /` on your system (dont run it). This will remove your entire directory structure given that `/` is your root directory. 

5) `mv`

This is another useful directory/file based command. `mv` will move something for you. 

```shell
mv index.html var/www/
```

will move the `index.html` file into your `var/www` directory. Another use of the `mv` command though is to also rename a file.

```shell
mv index.html index_old.html
```

will rename `index.html` to `index_old.html`.

6) `cp`

If you dont want to move something, but rather copy it then `cp` is the command you want. 

```shell
cp index.html var/www
```

will copy your `inde.html` file into the `var/www` directory. 

7) `cat`

`cat` will print out the contents of a file to your screen. 

```shell
cat ~/.ssh/.id_rsa.pub
```

will print out your public SSH key to the screen. 

8) `restart`

Moving on from directories now we have `restart` which I personally normally use in conjunction with `service`. The `restart` command will restart your system. Used with `service` it will just restart a service. Taking Nginx for example

```shell
service nginx restart
```

will restart the `nginx` service. You will need that one a lot when you are configuring Nginx

9) `reload`

However, sometimes you dont want to restart a service. Maybe you just want to reload the config. 

```shell
service nginx reload
```

will do just that. The difference being that `reload` will continue the service running, a `restart` will stop the service, then start the service again. Giving a momentary downtime. 

10) `df`

You probably wont need this one if you are just running a basic website or something on your server. However it can become quite useful if you have multiple sites or a small disk. `df` will tell you how much space is being used and how much is free on your disk. Similar to the `ls` command above, you will likely want the `-h` flag for humanly readable. 

```shell
df -h
```

11) `top`

`top` will bring up a screen with your currently running process, how much memory and CPU they are using among other things. This can be useful if you are having issues and need to diagnose some problems like memory leaks. 

12) `wget`

This is used to get the contents of a url. It has many uses, i mainly use it for 2 specific reasons. Either I simply want to download the contents or I want to run the script that is at that location. 

```shell
wget http://example.com/file.iso
```

this will download the iso from that location.

```shell
wget http://example.com/sendNotification.php
```

this will run the `sendNotification.php` script. Which could be run on say a cron job. 

13) `apt-get`

`apt-get` is a essential tool on a Ubuntu server. There are three parts to it, `install`, `remove` and `update`. This is what you will use to install all your packages like Nginx or MySQL. 

```shell
apt-get update
```

this will update the list of repositories in the apt get lists so you are install the most up-to-date stable releases. 

```shell
apt-get install nginx
```

this will install the `nginx` package.

```shell
apt-get remove nginx
```

this will remove the `nginx` package. 

14) `sudo`

When it comes to installing things on your system, its likely you will need to be a root user. Or at least have root level permissions for the system. `sudo` will ensure you are running your commands with those privileges. This will in most cases then ask for your password. 

```shell
sudo apt-get install nginx
```

15) `chmod`

Talking of permissions, there may come a time when you need to change these. `chmod` will change the permissions levels for the given file. Its too in depth to explain Linux permissions here, however I will give a brief overview.

Permissions on a Ubuntu (Linux) system are based on a numbering system. With 3 classes of user. The `owner`, the `group` and the `world`. The `owner` is the user than owns the file/directory. The `group` is the group of users that the directory belongs to and the `world` users are everyone else. 

There are then 3 levels of permissions, read (`r`), write (`w`) and execute (`x`). There are fairly self explainatory. Each of these permissions is given a point level and those points added together for each user and then concatenated give the file/directory its permission level. 

* 4 points for `r`
* 2 points for `w`
* 1 point for `x`

So take `index.php` for example. If I want `r`, `w` and `x` for the owner, `r` and `w` for the group and just `r` for everyone else my persmissions would be as follows:

```
4 + 2 + 1 = 7 # owner
4 + 2 = 6 # group
4 = 4 # world
```

Concatenate those numbers together you get `764` so running

```shell
chmod 764 index.php
```

will give those permissions for those users for the index.php file.

Another useful note here is the `-R` command is used for recursion through directories. 

16) `chown`

This will chnage the owner of the file / directory.

```shell
chown john app/
```

This will make john the owner of the app directory. 

17) `chgrp`

This will change the group of the file / directory.

```shell
chgrp www-data app/
``` 

This will change the `app` directory group to www-data

18) `adduser`

Not much good chnaging permissions for users if you dont have any. The `adduser` command will do just that. Some people like to user `useradd` which technically does the same thing, but `adduser` will guide you through adding the user with passwords, names etc and create the home directory for you. `useradd` wont, you will need to do that manually.  

```shell
adduser john
```

19) `usermod`

You will need this one to add an existing user to a group. 

```shell
user mod www-data john
```

this will add the user john to the group www-data.

20) `deluser`

This unsurprisingly will remove a user.

```shell
deluser john
```

Will remove the user john from the system. 



