---
layout: post
title: "Setting up a Raspberry Pi for hacking and development"
date: 2016-08-10 23:02
comments: true
categories: Raspberry-Pi, programming
---

For the last several years I have been writing scripts to simplify and automate the setup for Raspberry Pi single board computers using Raspian Linux (a Raspberry Pi specific port of Debian GNU/Linux). It's about time I distilled what I have learnt into some notes.

Most of this material can also be found in my Pi [setup scripts](https://github.com/alecthegeek/CCHS_Raspian_for_IoT)

# Keeping the OS up to date.

Use the standard Debian `apt-get` utility. For example

`apt-get update && apt-get upgrade -y && apt-get dist-upgrade -y ; apt-get autoremove ; apt-get clean`

## How to get a reminder to update the OS

There is a little know feature in apt-get that allow you to run triggers after `apt-get update` runs successfully. It's very poorly documented and only runs after an update, not the actual upgrade. None the less you can use this hook to create a timestamp to be checked each time the user logs in.

Create a file in `/etc/apt/apt.conf.d/` with a name similar to `15update-stamp`. Place the following content in it

    APT::Update::Post-Invoke-Success {"touch /var/lib/apt/periodic/update-success-stamp 2>/dev/null || true";};

Each time the user logs in you can compare the current date to timestamp of the marker file `/var/lib/apt/periodic/update-success-stamp` to see how long since the last `apt-get update` was run.

Something in the user's `.profile` like this:

```shell
# Check apt-get is run every few days
no_of_days=7 # How often we want to prompt

no_of_seconds=$(($no_of_days*24*60*60))

if [[ ! -e /var/lib/apt/periodic/update-success-stamp ]] ||
      $(( ( $(date +%s) - $(stat -c %Y /var/lib/apt/periodic/update-success-stamp) ) > $no_of_seconds )) ; then
     echo "It has been at least $no_of_days days since the last update"
     echo 'Please run "sudo apt-get update && sudo apt-get -y upgrade"'
echo fi
```

# Rename the Pi user account

I find this a convenience, rather than creating a second account. However caution is required as the following four files must be all updated consistently: `/etc/group`; `/etc/passwd`; `/etc/sudoers`; and `/etc/shadow`. Failure will often mean your OS image is useless and you need to copy Raspian to the SD card again. The following shell script will help

```shell
for i in /etc/group /etc/passwd /etc/sudoers /etc/shadow ; do
          sed -rie '/\b'pi'\b/s/\b'pi'\b/'NEW_NAME'/g' $i
done
```
You then need to immediately rename the home directory with `mv /home/pi /home/NEW_NAME`

You need privilege when doing this and I'm pretty paranoid about running a whole script under `sudo` so I run this as part of a larger setup script.

```shell
cat <<'EOF' | sudo bash -s pi NEW_NAME /home/pi
 for i in /etc/group /etc/passwd /etc/sudoers /etc/shadow ; do
      sed -ri -e '/\b'$1'\b/s/\b'$1'\b/'$2'/g' $i
 done
 mv $3 /home/$2
EOF
```

# Renaming the host

This is fairly easy. Just update the contents of `/etc/hostname` and `/etc/hosts` then reboot the Pi.

# Force the Pi to check the filesystem (and repair if required) on _every_ reboot

Add the text `fsck.mode=force fsck.repair=yes` to the end of `/boot/cmdline.txt`

N.B. This file should be checked after every update. The following sed script will do this

```shell
sudo sed -i -e 's/ fsck\.\(repair\|mode\)=[^ ]*//;
                s/$/ fsck.mode=force fsck.repair=yes/' /boot/cmdline.txt
```

# Consider installing some Python package management tools

```shell
sudo apt-get install python-setuptools &&
sudo easy_install pip &&
sudo pip install virtualenv virtualenvwrapper
```

It makes installing other packages (e.g [ino](http://inotool.org/) for Arduino) a lot easier

# Using Node.js?

Many node packages are very fussy about which version of Node they use on the Pi. Consider installing [nvm](https://github.com/creationix/nvm) 1st. The `curl` installer method works fine on the Pi.

# Wish you could have something like Dropbox on the Pi?

Follow these [instructions](https://github.com/makerdayprojects/makerdayprojects.github.io/wiki/SetupNotes#cloud-storage-for-raspberry-pi)

# Install Go programming language

```shell
INSTALL_VERSION=1.6.3
OS=linux
ARCH=armv6l

if type go > /dev/null 2>&1 && [[ "$(go version 2>/dev/null)" =~ $INSTALL_VERSION ]] ;then
  echo $(go version)  already installed at $(go env GOROOT)
  exit 2
fi

[ -d /usr/local/go ] && sudo rm -rf /usr/local/go

wget -O -  https://storage.googleapis.com/golang/go${INSTALL_VERSION}.${OS}-${ARCH}.tar.gz | sudo tar -xzC /usr/local  -f -


echo '# Setup for golang' |sudo tee /etc/profile.d/golang.sh  > /dev/null
echo 'PATH=$PATH:/usr/local/go/bin'|sudo tee -a /etc/profile.d/golang.sh > /dev/null

source /etc/profile.d/golang.sh
```

I also like to install the Go toolchain in /usr/local/go/bin so I run this as well

```
sudo -i GOPATH=/tmp GOBIN=$(go env GOROOT)/bin go get -u golang.org/x/tools/cmd/...
```
