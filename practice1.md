## Writing a bash script that creates 20 users from a CSV file with default passwords & implementing the former with an Ansible playbook.

The task is in two phases; 
1. Writing a bash script that creates 20 Users(10 users to sys-admin group,5 users to data-admin group, 5 users to database group)
2. Implementing No.1 with an Ansible Playbook.

## For the first phase, start by Creating a .csv file locally for sys-admin group, data-admin group and database group containing the list of users for each group.

    nano sys-admin.csv
    nano data-admin.csv
    nano database.csv
