# linux notes

## first steps in the terminal

We can check we are using bash as a shell with `echo "${BASH_VERSION}"` If we do not see any output, we can enter a bash session by just typing the command `bash`

A useful basic command is `echo` This command outputs text to the screen. We wrap what we want to output to the screen in single or double quotes. If we are just typing text, we can use single quotes. These quotation marks do not make the text a string as they do in other programming languages. In bash, the quotations are related to expansions.

With the echo command, we specify an argument which is what we would like to output `echo 'hello world!'` The command along with the arguments makes a complete bash command.

The echo command finishes with a newline. We can remove this with the `-n` option like so `echo -n 'hello world!`

Another option we can use is `-e` which enables escape characters such as `\n` or `\t` An example of this is `echo -e 'bash is\ngreat!` This will use `\n` as a new line.

We can combine options - a quick way to do this is to put them all with one hyphen like so `echo -en 'hello\nworld!`

Another useful command is `ls` which lists the contents of a directory. We can use the `-a` option which will list all of the contents including hidden directories and folders. We can use the `-l` option to list more details about the contents of a directory and we can combine these two options using `ls -la` We can list the contents of a directory in time order using the `-t` option or in reverse order using the `-r` option.

When using `ls` we can specify which directory we want to list the contents of like so `ls -al /home/user/other`

We can use `--color=` to specify what we want bash to do with colours. The values we can use are `never`, `auto` and `always`. The `--` work with whole words whereas `-` works with single letters. We could use `--color=always` to use colours.

### displaying and changing directories

In a shell, we are always in a directory. We can print the current working directory using `pwd` We can change our directory using the `cd` command. When we use the `cd` command, we need to specify the directory we would like to change to as its argument `cd /home/user/Documents` The current working directory can be specified with `.` or we can go up the directory tree one level using `cd ..`

We can use *absolute* or *relative* paths. An *absolute* path defines the complete path and therefore works from anywhere. Absolute paths begin with `/` unless we use `~` to mean the user's home directory. An example of an absolute path is `/home/l337/Documents/other`

A *relative* path is resolved according to our current working directory. We could just type the name of a directory in the current working directory `cd Documents/other` or we could use the `.` like so `cd ./Documents/other`

We could combine `..` with relative paths to go up one level and then somewhere else `cd ../Downloads` We can also move up more than one level by using more `../` for example `cd ../../billybob/Documents`

### combining commands

We can combine commands in bash using `;` like so `cd /home/user/Documents; ls -al`

## user management

On a linux system, there are different types of user. There is a *root* user who has the highest privileges and therefore complete control of the system. This user is given the user id of 0. There are *regular* users who do not have the same privs as the root user, but they can be given elevated privs on a temporary basis using the *sudo* command which is explored in more detail later in this section. There are also *service* users. Services have user accounts so we can logically seperate them from the rest of the system. This is a good idea when it comes to security. An example is the `www-data` user account which the `apache2` service runs under. If an attacker compromises the machine via the web-server, they land on the machine as a restriced service user.

All users have a *primary group* which is usually named the same as the username. Users can have additonal groups - the number of groups is not limited. Each user has a unique id and each group has a unique id.

### /etc/passwd /etc/shadow and /etc/group

User information is contained in the `/etc/passwd` file. The info is the username, uid, gid, user description (full name), home directory path and default shell. The password hash is not stored in this file - it used to be - there is an x where the hash would be. We could add a password hash here, but that is not good for security - it is a technique we can use as attacker's if `/etc/passwd` is writeable.

The password hashes are stored in the `/etc/shadow` file. This file should only be accessible to the root user or users with elevated privs as it contains sensitive data. In addition to the password hashes, it stores the date of the last password change and password expiry dates.

The hashing algorithm used for the password hash can be determined from the first part of the hash. The hash will generally be of the following format `$type:$salt$hashedpassword` If we see * or ! instead of a hash, it means that password logins have been disabled for that user. The ! specifically means that the password for that user has been locked and therefore cannot be used. We could still be able to switch to that user or use key based authentication, however.

After the hash, we will see the date the password was last changed (the number specifies the number of days from *January 1st 1970*), the minimum password age (which is usually set to 0 which means there is no minimum password age), the maximum password age (how long before the password has to be changed - this is set to 99999 by default), and the password change warning number which is how long before the password expires that the user is notified that they need to change their password.

The `/etc/group` file contains data about the additional groups. This is a useful file to look at when we are enumerating a machine and want to know which groups are on it as it lists all the additonal groups along with the names of users assigned to it. This file is usually readable by all users.

### creating and securing users

We can use the `useradd` command to create a new user. An important flag is `-m` which specifies that the user should have a home directory. Service users do not need home directories. The `-d` flag lets us specify a path for the home directory of the user. The `-s` flag lets us specify a default shell. An example command to create a new user with a home directory and bash as a shell is `sudo useradd -m -s "/bin/bash" dmouse`

We can specify the groups a new user belongs to. We can specify a *primary* group using the `-g` flag. We can specify additional *secondary* groups by using the `-G` flag.

Once we have created a new user, we need to give them a password if we want them to be able to log into the system. The `passwd` command lets us do this. The `-S` flag shows us the password status. The `-d` flag will delete the password. The `-n` flag will let us specify the minimum password age whilst the `-x` flag lets us specify the maximum password age - both these values will be in days. The `-l` flag will lock the password whilst the `-u` flag will unlock the password.

When we look at the status of the password, we might see `P` which means a regular password is being used and is not locked. If we see `L` then the password is locked. Then we will see the date the password was changed. We then see the minimum, maximum, and warning data. We can see the password status for other users by specifying their name like so `sudo passwd -S dmouse`

We can set a password for a user with the command `sudo passwd dmouse` and then specify the new password when we are prompted to do so.

After the maximum password age the user will be forced to change their password. The minimum password age value will specify how long the user needs to wait after changing their password before they can change it again.

When we look at the status of a password, a `P` means the password is normal whilst an `L` means it has been locked. The date is when the password was last changed. The next two numbers are the minimum and maximum. The next number specifies how many days before the maximum is reached that a warning will be given.

If a user is operating `passwd` with elevated privs password policies can be ignored whereas a low privileged user will find that password policies are enforced.

### modifying users

We can *modify* users using the `usermod` command. The `-c` flag lets us change the user description (full name) whilst the `-s` flag lets us change the default shell.

We could use the `-d` flag to change the home directory along with the `-m` flag to move the existing home directory to the new location. We could also use the `-l` flag to change the linux username.

> [!CAUTION]
> Changing the home directory or linux username can break apps which use the original values

When we change the default shell using the `usermod` command, we can use any shell - it does not have to be in the `/etc/shells` file. If a user is changing their own default shell, they will need to use a shell in `/etc/shells` and they need to use the `chsh` command with the `-s` flag set.

We can use the `-g` flag to change the *primary* group, the `-G` flag to change *secondary* groups and the `-aG` flag to add *secondary* groups.

If we are changing the *secondary* groups with the `-G` flag, we need to specify all of the current groups if we want to keep them - if we do not specify an existing group when we use the `-G` flag the group will be removed from the user.

We can use the `deluser` command to delete a user from a group by specifying the user and then the group.

### deleting users

We can *delete* users with the `userdel` command. The only argument we need to pass to it is a username. We can use the `-r` flag to recursively remove the contents of the home directory and mail for the user from `/var/mail`. The `-f` flag will do the same as the `-r` flag but it will enforce the deletion of the user even if they are currently logged in. It might also delete any groups with the user's name.

## group management

### groups

Groups let us more easily manage users which need similar access rights. We can give them the access rights via a group rather than individually changing the rights for each user. Groups therefore simplify user management.

Groups can enhance resource sharing as we can use them to manage access to files and directories.

Groups let us strengthen system security. Groups can be given access to a specific command and they can let us restrict or give access to resources.

Each user has a primary group and they can also belong to secondary groups. The primary groups are stored in `/etc/passwd` and they determine the default ownership of files when they are created. We can check the names of groups along with their corresponding `gid` numbers by looking at `/etc/group`

We can check which groups our current user belongs to by using `groups` We can check which groups a different user belongs to by specifying their username like so `groups billybob`

There are some built-in groups which are important. The `root` group has admin privs and therefore members of it have *complete control over the system*.

Members of the `sudo` aka `wheel` group have access to commands on a temporary basis with elevated privs via the use of the `sudo` command.

Members of the `adm` group can read log files.

Members of the `lpadmin` aka `lp` group can manage printers and print jobs.

Members of the `www-data` group can access web content. This group is used for web server processes such as Apache or Nginx.

Members of the `plugdev` group can manage pluggable devices such as pen drives and external harddrives.

### managing group members

We can change the *primary* group of a user with `sudo usermod -g new_group lauren` or we can add a new *secondary* group to a user with `sudo usermod -aG new_secondary_group lauren`

We can delete a user from a group using `sudo deluser lauren adm` but if this tool has not been installed we can do it with `sudo usermod -g g1,g2 billybob` In this case we want to delete billybob from g3 but not from g1,g2 so we need to specify the secondary groups he is in which we want him to remain in.

### creating, modifying and deleting groups

We can create our own groups and give them a custom `gid` - this is usually above 1000 but we can use any number. We might use a range of numbers for the `gid` of custom groups. We must use a *unique* `gid`.

We can use `sudo groupadd -g 2001 apps` The `-g` flag lets us specify the `gid`.

We can modify existing groups using `sudo groupmod -n cool_apps -g 2002 apps` The `-g` flag is the same as before and the `-n` flag lets us change the name of the group.

> [!CAUTION]
> Modifying group ids can negatively impact access to resources for users in the group

To delete a group, we can use `sudo groupdel cool_apps` This will not work if the group which we are trying to delete is the *primary group* of *any user*.

## automating and scheduling tasks using cron jobs

### cron jobs intro

Cron jobs let us automate tasks and schedule them. This means we can get tasks to repeatedly run on our system following a pre-defined time schedule.

Here are two examples of when we might want to set up a cron job:

- We want to run a backup script everyday at 03:00
- We want to execute an update script every hour of every day

We could of course achieve this using a different tool such as `systemd` but cron jobs still have a place in the unix world.

Cron jobs work on all *unix* systems not just *linux* systems - this makes them more *flexible* than `systemd`.

We can automatically send *output* from cron jobs to email.

Cron jobs are built into lots of products already - such as websites which use scheduled tasks - so it is useful to know about them.

### cron variants

There are different implementations of cron - they tend to support different features.

#### vixie-cron

`vixie-cron` is the default implementation of cron which ships with *ubuntu* - it is known as `cron`

#### anacron

`anacron` is also used on *ubuntu* systems - if the system is shut down the jobs scheduled by `anacron` will continue to work as they have been defined when the system reboots.

#### cronie

`cronie` is used on *centos* - it is a fork of `vixie-cron` and it has `anacron` built into it whereas on *ubuntu* systems these are seperate

### cron daemon

The *cron daemon* aka `crond` is the background process which manages scheduled tasks. It executes tasks at specified intervals and checks every minute if something needs to be executed.

`crond` reads *cron tab* files. These files can be *user specific* such as `/var/cron/tabs` and `/var/spool/cron/crontabs` or they can be system wide such as `/etc/crontab`

> [!IMPORTANT]
> `/etc/crontab` must be owned by `root` and no other users or groups should be allowed *write* access to it

On *debian* systems `crond` will also check in `/etc/cron.d` though in general we should not be setting up cron jobs using this file. We find sometimes that third-party apps put cron jobs into `etc/cron.d`

#### creating user specific cron jobs

We can edit the *crontab* file using `crontab -e` This will open an editor for the user to edit their own cron jobs.

> [!IMPORTANT]
> We should only use `crontab -e` to set up or edit *user specific* cron jobs

The *crontab* file for the user will be opened - each user can specify their own tasks which they want to have executed in the background at scheduled times.

> [!NOTE]
> The `root` user can also create their own *user specific* cron jobs

We can temporarialy change the `EDITOR` *environmental variable* for editing the *crontab* file using `EDITOR=nano crontab -e`

Once cron jobs have been set up by a user, a *crontab* file under their username will be created in `/var/spool/cron/crontabs` which is a directory only accessbile to the `root` user.

All user specific *crontab* files are kept in the one directory `/var/spool/cron/crontabs` but we should not really access this directory - we certainly should not edit the files in it directly.

> [!IMPORTANT]
> We should use `crontab -e` whenever we want to create a *user specific* cron job - *do not* edit the *crontab* file directly

If we want to just have a look at the contents of our *crontab* file we can use `crontab -l` which will list the cron jobs in it.

##### crontab syntax

We can define some *environment variables* in the *crontab* file. We could for example set the `SHELL` environment variable because the default shell used is `/bin/sh` and we might want to use `/bin/bash` Another environment variable we could set is the `PATH` because by default `cron` will use `/usr/bin:/bin`

The actual cronjob syntax specifies time and the command to run as shown below (in this example we are using `*` to be a wildcard to mean every so the command will run every minute of every hour of every day of every month on every day of the week):

|minute|hour|day|month|day of the week|command|
|---|---|---|---|---|---|
|*|*|*|*|*|echo "hello" > /home/user/test.txt|

An example of how we can enter a command to run as a cron job is:

```bash
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/local/sbin:/sbin:/usr/sbin:/bin:/usr/bin

* * * * * ping -c 1 google.com >> test.txt
```

The command can be specified in the *crontab* file as above. If we want to run something more complicated, we can specify a path to a *shell script* in the command section.

```bash
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/local/sbin:/sbin:/usr/sbin:/bin:/usr/bin

* * * * * /home/user/backup.sh >> test.txt
```

###### time format

We can specify a value directly - this one will execute the command at 08:30 every day.

```bash
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/local/sbin:/sbin:/usr/sbin:/bin:/usr/bin

30 8 * * * ping -c 1 google.com >> test.txt
```

We can specify multiple values - this one executes the command at the specified minutes past every hour.

```bash
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/local/sbin:/sbin:/usr/sbin:/bin:/usr/bin

0,15,30,45 * * * * ping -c 1 google.com >> test.txt
```

We can specify a range of values - this one executes the command on every hour including 5:00 and 17:00.

```bash
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/local/sbin:/sbin:/usr/sbin:/bin:/usr/bin

0 5-17 * * * ping -c 1 google.com >> test.txt
```

We can specify every so many minutes or hours etc - this one executes the command every ten minutes.

```bash
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/local/sbin:/sbin:/usr/sbin:/bin:/usr/bin

*/10 * * * * ping -c 1 google.com >> test.txt
```

The *day of the week* value acts as a filter. If we want to specify *Sunday* we can use either 0 or 7 with the other days of the week being given the respective numeric values - this example runs the command only on Fridays at 08:00.

```bash
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/local/sbin:/sbin:/usr/sbin:/bin:/usr/bin

0 8 * * 5 ping -c 1 google.com >> test.txt
```

##### managing cron output

In the previous examples we always *redirected* the output to a file. If we do not do this, `cron` will attempt to send the output to the email address of the user. If a *mail transfer agent* has not been installed, the command output will be discarded.

We can check the results of cron jobs by using `journalctl -u cron.serice` - we can look at just the end of this log file by using `journalctl -u cron.serice -f`

###### installing a mail transfer agent

We can use:

```bash
sudo apt-get update
sudo apt install mailutils
```

This will trigger a configuration wizard for *postfix* - we can either configure it to send mail to local users or via the internet.

> [!TIP]
> If *postfix* is already installed and we want to change its configuration we can use `sudo dpkg-reconfigure postfix`

If we configure *postfix* to use *local only* we will see the cron job output in our mail which is found in `/var/mail`

If we configure it to use the internet we will need to specify an email account for the output to be sent to. We can do this using the `MAILTO` *environment variable* in the *crontab* entry.

```bash
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/local/sbin:/sbin:/usr/sbin:/bin:/usr/bin
MAILTO="dduck@duckyduck.com"

0 8 * * * /home/user/check.sh
```

#### creating system wide cron jobs

We can also create *system wide* cron jobs which can sometimes be more useful than *user specific* cron jobs. The advantage to these cron jobs is that we can establish them as any user on the system rather than just the currently logged in user. An example of when this could be useful is if we need to establish a cron job as a different user - for example *www-data* - and we do not want to have to log in as them in order to set it up. It should be noted that only the *root* user should be allowed to create *system* cron jobs. The *system wide* cron jobs are defined in a regular file which can be edited by *root* directly - it is found at `/etc/crontab`

> [!NOTE]
> If the permissions on `/etc/crontab` are changed so other users or groups have *write* access to it, the commands in it *will not run*

The syntax for *system wide* cron jobs is slightly different to *user specific* cron jobs since we need to specify which user we want the task to execute as.

|minute|hour|day|month|day of the week|user|command|
|---|---|---|---|---|---|---|
|*|*|*|*|*|root|/root/update.sh|

> [!NOTE]
> Other users can *read* `/etc/crontab`

### cron jobs best practice

- Schedule tasks evenly
- Avoid peak hours where possible
- Avoid executing the same task multiple times - `flock` can help with this
- Log errors and check them frequently
- Add error checks to automated scripts
- Run tasks with the *least privilege* possible
- Secure scripts and commands by using correct *permissions* - maybe make them *read only*
- Do not store sensitive data in *crontab* files
- Test scripts before adding as cron jobs
- Ensure scripts work with the `PATH` variable which `cron` uses - or we can set our own as already mentioned

> [!TIP]
> If you want to learn more about how attackers can abuse cron jobs to elevate their privileges on a compromised machine, please take a look at [abusing cron jobs](https://puzz00.github.io/linux_cron.html)


