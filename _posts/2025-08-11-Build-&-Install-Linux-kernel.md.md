## Configuring Your Development System


### Installing required packages
~~~
sudo apt-get install build-essential vim git cscope libncurses-dev libssl-dev bison flex

sudo apt-get install git-email
~~~

### Set your username and email for git(If you havent set this already)

git config --global user.name yourname

git config --global user.email youremail

### setting SMTP

~~~
git config --global sendemail.smtpserver smtp.gmail.com
git config --global sendemail.smtpserverport 587
git config --global sendemail.smtpencryption tls
git config --global sendemail.smtpuser your.email@gmail.com
~~~

note: Don't add your app password here

Then lets test the configuration

~~~
mkdir git-email-test
cd git-email-test
git init
echo "Test content" > testfile.txt
git add testfile.txt
git commit -m "Test commit for git send-email"
~~~

Then lets check this

~~~
git format-patch -1 HEAD
0001-Test-commit-for-git-send-email.patch
git send-email 0001-Test-commit-for-git-send-email.patch --to=youremail@gmail.com

~~~

Now it will ask the password. If you not enable 2-step verification then enter your email password.

If you enable it then you have to put the app password for git-send-email. To get this use below link.

https://security.google.com/settings/security/apppasswords

Enter the App name as **"git-send-email"** and click generate. Then google will show you a 16-character password. Copy and paste as the password. Then you will receive an email on TEST commit for git send-email. 

Add the ss of the test email

## Cloning the Linux Mainline

It is good to creating a workspace in root not in under home directory.

~~~
sudo mkdir /linux_work
sudo chown $USER:$USER /linux_work
cd /linux_work
~~~

Note: If you don't run **sudo chown $USER:$USER /linux_work** the directory owned by root. That means you'll have to use sudo for every action. It's bad specially when you are working with Git. In the next step we'll clone the linux mainline with **git clone**, and using Sit with sudo can mess up file permissions and security.  

~~~
git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git linux_mainline
~~~

Now you have the linux kernel source code 😎

## Reviewing a patch

~~~
cd /linux_work/linux_mainline
git format-patch -1 a035d552a93bb9ef6048733bb9f2a0dc857ff869
nano 0001-Makefile-Globally-enable-fall-through-warning.patch
~~~

Note: a035d552a93bb9ef6048733bb9f2a0dc857ff869 this is the patch ID. You can replace it with any commit you wan to review.

## Building and Installing the kernel

### Cloning the stable kernel

Earlier we cloned the mainline kernel. Now we will clone the latest stable version.

Go to home directory and clone the repo.

~~~
git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git linux_stable
~~~

Check the latest stable version. 

https://www.kernel.org/

~~~
cd linux_stable
git branch -a | grep linux-6
​git checkout linux-6.16.y
~~~

### Copying the configuration

The linux_stable directory now has the  kernel we are going to work on. This is the unmodified linux kernel. But our linux distro (ubuntu in my case) already comes with the a full kernel configuration. So we'll copy that configuration into our new kernel.

~~~
ls /boot
~~~
You can find the latest kernel configuration file and copy it.
~~~
cd ~/linux_stable
cp /boot/config-6.11.0-26-generic .config
~~~

Now you have the ubuntu kernel configuration in your new kernel.

## Compiling the Kernel


### Installing Required Build Packages

~~~
sudo apt update
sudo apt install build-essential libncurses-dev bison flex libssl-dev libelf-dev dwarves
sudo apt install libdw-dev libiberty-dev
~~~

### Compiling the Kernel
~~~
cd ~/projects/linux_stable
make oldconfig
~~~

Select default option for all configuration prompts. Just press enter.

Start the build:

~~~
make -j$(nproc)
~~~

### Handling Certificate Errors

If you see errors something like this; 

make[3]: *** No rule to make target 'debian/canonical-certs.pem', needed by 'certs/x509_certificate_list'.  Stop.
make[3]: *** Waiting for unfinished jobs....

This is because it search for certificate. As per the chatgpt instruction I add create empty files  

~~~
scripts/config --set-str SYSTEM_TRUSTED_KEYS ""
scripts/config --set-str SYSTEM_REVOCATION_KEYS ""
~~~

Verify:

~~~
grep -E "CONFIG_SYSTEM_TRUSTED_KEYS|CONFIG_SYSTEM_REVOCATION_KEYS" .config
~~~

Expected output:

CONFIG_SYSTEM_TRUSTED_KEYS=""
CONFIG_SYSTEM_REVOCATION_KEYS=""

### Rebuild:
~~~
make olddefconfig
~~~

Then build the kernel

~~~
make -j$(nproc)
~~~

Once it ready you will see sommthing like this.

...   
Kernel: arch/x86/boot/bzImage is ready  (#1)

### Installing the New Kernel

~~~
sudo make modules_install install
~~~

Saving current kernel logs

~~~
sudo dmesg -t > dmesg_current
sudo dmesg -t -k > dmesg_kernel
sudo dmesg -t -l emerg > dmesg_current_emerg
sudo dmesg -t -l alert > dmesg_current_alert
sudo dmesg -t -l crit > dmesg_current_crit
sudo dmesg -t -l err > dmesg_current_err
sudo dmesg -t -l warn > dmesg_current_warn
sudo dmesg -t -l info > dmesg_current_info
~~~

We have to disable the secure boot.

check the secure boot status

~~~
mokutil --sb-state
~~~

If it is disable then we can continue. If not we have to disable it.

~~~
Disable validation:

sudo mokutil --disable-validation
root password
mok password: 12345678
mok password: 12345678
~~~

### Editing GRUB

~~~
sudo nano /etc/default/grub
~~~

Then edit like this.

~~~
GRUB_TIMEOUT=10
#GRUB_TIMEOUT_STYLE=hidden
GRUB_CMDLINE_LINUX="earlyprintk=vga"
~~~

Update GRUB:

~~~
sudo update-grub
~~~

### Booting the kernel

~~~
sudo reboot
~~~

Then you can select the kernel from the GRUB menu. Newly installed kernel you can find in the Advanced option for Ubuntu. 

After reboot run this

~~~
uname -r
~~~

You can check kernel version.