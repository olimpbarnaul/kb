# Linux...

## Disable ACPI Wake-Up after suspend settings 
### List of wakeup devices
cat /proc/acpi/wakeup
### Disable all devices (except power button): write this row before exit 0 in /etc/rc.local
for i in $(cat /proc/acpi/wakeup|grep enabled|awk '{print $1}'|xargs); do [ $i != PWRB ] && echo $i|tee /proc/acpi/wakeup;done
