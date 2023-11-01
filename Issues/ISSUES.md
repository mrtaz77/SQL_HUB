# Oracle, Navicat, and SQLPlus Troubleshooting Guide
- [ORA-12541 TNS:No listener](#ora-12541-tnsno-listener)
- [ORA-01109: database not open](#ora-01109--database-not-open)

## ORA-12541 TNS:No listener
![issue](/Issues/12541_TNS_No_listener.png)

Perhaps one of the most issues I faced.

### Properties
    * Sql plus is working properly.
    * Navicat is showing no listener,while opening an existing or new connection.


### Remedy
1. Open your cmd as administrator.
2. Type the following commands: 
    ```cmd 
    lsnrctl stop
    lsnrctl start
    ```
    Cmd should display something similar:
    ```cmd 
    C:\Windows\System32>lsnrctl stop

    LSNRCTL for 64-bit Windows: Version 19.0.0.0.0 - Production on 31-OCT-2023 11:35:00

    Copyright (c) 1991, 2019, Oracle.  All rights reserved.

    Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=your_host_name)(PORT=1521)))
    The command completed successfully

    C:\Windows\System32>lsnrctl start

    LSNRCTL for 64-bit Windows: Version 19.0.0.0.0 - Production on 31-OCT-2023 14:24:23

    Copyright (c) 1991, 2019, Oracle.  All rights reserved.
    ...
    ...
    The command completed successfully
    ```
3. Next in your windows search bar search and open __services.msc__
![services.msc](/Issues/services_msc.png)

4. Make sure that you have the highlighted services running.If not , then right click the service and click on start.

5. Sometimes the __OracleRemExecServiceV2__ will be off.Trying to run this service may give the following error.
![vss](/Issues/vssexec.png)

    No need to panic. Just click ok.

6. Enter navicat and open the connection. It should be working now.


## ORA-01109 : database not open
![issue](/Issues/01109_Database_Not_Open.png)

University schema was opened using a pluggable database.So, while opening the connection in navicat where you originally had the university schema may show the above error.

### Remedy
1. Open Sql plus . Enter username `sys as sysdba` . No password needed , so just press enter while prompted for it.

2. We need to open the pdb. So type :
    ```cmd
    alter pluggable database orclpdb open;
    ```
    The following msg should be displayed. 
    ```cmd
    Pluggable database altered.
    ```

3. That's it. Now enter navicat and open the connection containing the the schema.

4. If your problem is not solved , then check the [link](https://logic.edchen.org/how-to-resolve-ora-01109-database-not-open/).