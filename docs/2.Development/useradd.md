# user add script

~~~
#!/bin/bash

echo "-------------------------"
echo "   group and user add"
echo "-------------------------"
echo -n "username : " ; read user
echo -n "expire date (YYYY-dd-mm) : " ; read expire_date
echo -n "user group : " ; read group

if [ -z "$user" ] || [ -z "$expire_date" ] || [ -z "$group" ] ; then
        echo "Error input data null"
fi

if [ Z"`cat /etc/group | grep $group`" == Z  ]; then
        groupadd $group
fi

useradd -g $group $user
echo "${user}1!" | passwd --stdin ${user}

if [ $group == "SECL" ]; then
        echo "${user} ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/91-admins
        chage -E ${expire_date} ${user}
else
        echo "${user} ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/92-developers
        chage -E ${expire_date} ${user}
fi
~~~
