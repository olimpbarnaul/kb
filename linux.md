# Linux...

### Disable ACPI Wake-Up after suspend settings 
#### List of wakeup devices
cat /proc/acpi/wakeup

#### Disable all devices (except power button): write this row before exit 0 in /etc/rc.local
for i in $(cat /proc/acpi/wakeup|grep enabled|awk '{print $1}'|xargs); do [ $i != PWRB ] && echo $i|tee /proc/acpi/wakeup;done

### Mount remote filesystem with specific ssh key
sshfs me@x.x.x.x:/remote/path /local/path/ -o IdentityFile=/path/to/key

### Package info

#### Arch
pacman -Qi package_name

#### Ubuntu, Debian
apt show package_name

apt-cache show package_name

aptitude show package_name

dpkg -s package_name

#### RHEL & CentOS 
yum info package_name

### New features
locate

showkey -a

#### сайт где объясняется назначение параметров, с которыми можно запускать утилиты командной строки
https://explainshell.com/

#### программа для анализа исходного кода проекта, в частности подсчитывает количество строчек кода в проекте
cloc
