## Writing a bash script that creates  users from a CSV file with default passwords 

The task will be executed as below

## For the first phase, start by Creating a .csv file locally for sys-admin group, data-admin group and database group containing the list of users for each group.

    nano sys-admin.csv
    nano data-admin.csv
    nano database.csv

Create a bash script .sh file (e.g. create-sys.sh)

    nano create-sys.sh

paste the below in the file and save.

    #!/bin/bash

    sys_admin_file="sys-admin.csv"

    PASSWORD="Password@123"

    groupadd sys-admin > /dev/null

    while IFS= read -r USER ; do 
      useradd "$USER"
      echo "$USER:$PASSWORD" | chpasswd
      usermod -aG sys-admin "$USER"
      echo "$USER has been added to group sys-admin"
    done < "$sys_admin_file"
    
Give the file executable rights by doing the below;

    chmod 777 create-sys.sh

Run the script by inputting the below

    ./create-sys.sh

This will create the users listed in the sys-admin.csv file

![image](https://github.com/user-attachments/assets/6c9eeb16-fe7e-4b96-9cff-44146f0212fa)




    
