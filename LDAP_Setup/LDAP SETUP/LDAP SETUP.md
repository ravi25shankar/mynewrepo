# LDAP Setup on podman

## Requirement

1. Install and configure Ldap 389 DS in the containerised platform with persistence storage

2. Create organisation [keenable.in](http://keenable.in/)

3. Create OU [Dev,Support,POC, Document, Observability]

4. Create group Admin,Support

5. Create custom attribute

## Environment Info

**Server Info:-**

Os version

NAME="Ubuntu"

VERSION="20.04.6 LTS (Focal Fossa)"

podman version 3.4.2

**Client Info:-**

NAME="Ubuntu"

VERSION="20.04.6 LTS (Focal Fossa)"

Ldap version:- 3

Apache Directory Studio version:- 2.0.0.v20210717-M17

## List of Tools:-

1. Podman 
2. Ldap
3. Apache Directory Studio

**LDAP**:- The Lightweight Directory Access Protocol is a communication protocol used to access directory servers and is used to store, update and retrieve data from a directory structure.

## Command for the setup or configuration

**Create directory with name 389ds and also create directory data inside**

```bash
mkdir -p 389ds/data
```
```bash
cd 389ds/data
```


**1. Create Bash script for create Pod and container**

 vim ldap.sh

Bash script :-

```bash
#!/bin/bash
#create pod with name ldap389
podman pod create --name ldap389 --publish 3389:3389 --publish 3636:3636
#create container
podman run -dt \
--pod ldap389 \
--name 389ds-ldap \
-v /home/ravi/389ds/data:/data \
-e DS_SUFFIX=dc=keenable,dc=in \
-e DS_DM_PASSWORD=ravi \
docker.io/389ds/dirsrv
```

- **ldap389** Created a pod named
- **#!/bin/bash:** This is called a shebang and specifies that the script should be executed using the Bash shell.
- **podman pod create --name ldap389 --publish 3389:3389 --publish 3636:3636:**
- **podman:** This is the command-line tool used to manage pods and containers.
- **pod create:** This subcommand is used to create a new pod.
- **-name ldap389:** This option specifies the name of the pod as "ldap389".
- **-publish 3389:3389 --publish 3636:3636:** These options publish the
    
    specified container ports to the host. Ports 3389 and 3636 are mapped from the host to the pod. This allows services running inside the pod to be accessed from the host using these ports.
    
- **podman run -dt:** This command runs a container.
    - **d:** Detaches the container from the terminal, allowing it to run in the background.
    - **t:** Allocates a pseudo-TTY, which helps with handling input/output.
- **-pod ldap389:** This option specifies that the container should be created within the "ldap389" pod that was created earlier.
- **-name 389ds-ldap:** This sets the name of the container to "389ds-ldap".
- **-v /home/ravi/389ds/data:/data:** This mounts the directory **/home/ravi/389ds/data** from the host into the container at the **/data** directory. This is used for data persistence, allowing data to be stored on the host filesystem.
- **-e DS_SUFFIX=dc=keenable,dc=in:** This sets the environment variable **DS_SUFFIX** within the container to specify the LDAP directory suffix.
- **-e DS_DM_PASSWORD=<password>:** This sets the environment variable **DS_DM_PASSWORD** within the container to specify the password for the Directory Manager of the LDAP server.
- **docker.io/389ds/dirsrv:** This is the Docker image that will be used to create the container. It's the 389 Directory Server image from Docker Hub.

 In Pod we have set port 3389 and 3636

 In container we have set base Dn as keenable.in and set password of dn


* Give permmision to script file
```bash
sudo chmod 777 ldap.sh
```
> changes the permissions of the file and permissions to 777 means that the owner, the group, and everyone else can read, write, and execute the script.
* Run the script
```
sh -x ldap.sh
```
* Check container list

```bash
podman ps -a --pod
```
* **Check container list**
```bash
podman ps -a
```


* **Go inside container**
```bash
podman exec -it 389ds-ldap bash
```
* **Create backend suffix list**
```bash
dsconf -D "cn=Directory Manager" ldap://localhost:3389 backend create --suffix="dc=keenable,dc=in" --be-name="keenable"
```
 * **Check backend suffix list**
 ```bash
 dsconf -D "cn=Directory Manager" ldap://localhost:3389 backend suffix list
 ```
* **Install Ldap utility on bash machine for runn ldap command**

```bash
sudo apt install ldap-utils
```


- **sudo:** This command is used to run with superuser (root) privileges.
- **apt:** It's used to manage software packages, including installation, updating, and removal.
- **install:** This is an argument passed to the apt command, indicating that you want to install a package.
- **ldap-utils:** This is a collection of command-line utilities for interacting with LDAP (Lightweight Directory Access Protocol) servers. These utilities are useful for managing and querying directory services.



**2.  Setup ApacheDirectory studio for ldap db UI**

 a. Install [ApacheDirectory studio tar](https://directory.apache.org/studio/download/download-linux.html) file and extract in directory  

 b. Untar the downloded tar file 
 ```bash
 tar -xvf ApacheDirectoryStudio-2.0.0.v20210717-M17-linux.gtk.x86_64.tar.gz
 ```

 c. open apache directory 
 ![](<Untitled (6).png>)
 d. Go to new connection and enter some details



```bash
ldap connection name
Hostname -> localhost
ldap port > Press next (3389)
Enter Bind DN name (cn= Directory Manager) 
Enter Bind DN password -> finish
```

![Alt text](<Untitled (4).png>)
![Alt text](<Untitled (5).png>)

**Create Organization_unit ldif file (file extension name , ldif )**

```bash
cat organisation.ldif
```

1. we have create 5 organisation ( Dev, Support, POC, Document, Observability)

```bash
dn: dc=keenable,dc=in
objectClass: top
objectClass: domain
dc: keenable

dn: ou=Dev,dc=keenable,dc=in
objectClass: top
objectClass: organizationalUnit
ou: Dev

dn: ou=Support,dc=keenable,dc=in
objectClass: top
objectClass: organizationalUnit
ou: Support

dn: ou=POC,dc=keenable,dc=in
objectClass: top
objectClass: organizationalUnit
ou: POC

dn: ou=Document,dc=keenable,dc=in
objectClass: top
objectClass: organizationalUnit
ou: Document

dn: ou=Observability,dc=keenable,dc=in
objectClass: top
objectClass: organizationalUnit
ou: Observability
```

 Run this command to add **organisation.ldif**

```bash
ldapadd -a -c -x -H ldap://localhost:3389 -D "cn=Directory Manager" -W  -f organisation.ldif
```

- **ldapadd:** This command is used to add new entries to an LDAP directory server.
- **-a:** It's used for adding new data without overwriting existing data.
- **-c**:If the LDAP server has a schema, this option helps ensure that the data conforms to the defined schema.
- **-x:** This option specifies that simple authentication should be used. Similar to -**x** in **ldapsearch**, it's used for quick testing,
- **-H ldap://localhost:3389:** This option specifies the LDAP URI (Uniform Resource Identifier) of the LDAP server. It's connecting to the LDAP server running on the local machine (**localhost**) at port **3389.**
- **-D "cn=Directory Manager":** This option specifies the Bind DN (Distinguished Name) to authenticate with the LDAP server. In this case, it's using the "Directory Manager" account to authenticate.
- **-W:** This option prompts you to enter the password for the Bind DN interactively after providing the -D option. It's safer than putting the password in the command line itself.
- **-f organisation.ldif**: This option specifies the name of the LDIF (LDAP Data Interchange Format) file that contains the data to be added to the LDAP server. The **f** option is followed by the filename **(organisation.ldif** in this case).


**4.  Create 2 group inside Support**
```bash
vim  group.ldif
```

```bash
dn: cn=Admins,ou=Support,dc=keenable,dc=in
objectClass: top
objectClass: groupOfUniqueNames
cn: Admins
uniqueMember: uid=user1,ou=Support,dc=keenable,dc=in

dn: cn=SupportTeam,ou=Support,dc=keenable,dc=in
objectClass: top
objectClass: groupOfUniqueNames
cn: SupportTeam
uniqueMember: uid=user2,ou=Support,dc=keenable,dc=in
```

Run this command to add **group.ldif**

```bash
ldapadd -a -c -x -H ldap://localhost:3389 -D "cn=Directory Manager" -W -f group.ldif
```

- **-W:** This option prompts you to enter the password for the Bind DN interactively after providing the **D** option. It's safer than putting the password in the command line itself.
- **-f group.ldif:** This option specifies the name of the LDIF (LDAP Data Interchange Format) file that contains the data to be added to the LDAP server. The **f** option is followed by the filename (**group.ldif** in this case).



**5. Run some Command of ldap**

a. First check how many default object class created

```bash
ldapsearch -o ldif-wrap=no -x -H ldap://localhost:3389 -D "cn=Directory Manager" -w "ravi" -b "cn=schema" "(objectClass=subSchema)" -s sub "objectClasses"
```

- **ldapsearch:** This command is used to search and retrieve information from an LDAP directory server.
- **-o ldif-wrap=no:** This option turns off LDIF (LDAP Data Interchange Format) line wrapping. It ensures that the output LDIF is not formatted with line breaks, which can be useful for processing the output programmatically.
- **-D "cn=Directory Manager"**: This option specifies the Bind DN (Distinguished Name) to authenticate with the LDAP server. In this case, it's using the "Directory Manager" account to authenticate.
- **-w "ravi"**: This option specifies the password for the Bind DN. The password is provided directly in the command line using double quotes.
- **-b "cn=schema"**: This option specifies the base DN (Distinguished Name) from which the search should start. It's set to **"cn=schema"** which indicates that the search should start from the schema entry.
- **'(objectClass=subSchema)**': This is the search filter. It specifies that the search should retrieve entries with the **objectClass** attribute equal to **subSchema**. This filter targets the schema definition entries.
- **-s sub**: This option sets the search scope. In this case, it's set to "sub" which means a subtree search, i.e., it searches for the specified filter under the specified base DN and all its subordinates.
- **objectclasses:** This is the attribute(s) we want to retrieve from the entries that match the search filter. In this case, we're requesting the **objectclasses** attribute.

a. Check how many default attributes created

```bash
ldapsearch -o ldif-wrap=no -x -H ldap://localhost:3389 -D "cn=Directory Manager" -w "redhat@" -b "cn=schema" '(objectClass=subSchema)' -s sub attributetypes
```

- **-o ldif-wrap=no:** This option turns off LDIF (LDAP Data Interchange Format) line wrapping. It ensures that the output LDIF is not formatted with line breaks, which can be useful for processing the output programmatically.
- **-D "cn=Directory Manager":** This option specifies the Bind DN (Distinguished Name) to authenticate with the LDAP server. In this case, it's using the "Directory Manager" account to authenticate.
- **-b "cn=schema"**: This option specifies the base DN (Distinguished Name) from which the search should start. It's set to **"cn=schema"** which indicates that the search should start from the schema entry.
- **'(objectClass=subSchema)'**: This is the search filter. It specifies that the search should retrieve entries with the **objectClass** attribute equal to **subSchema.** This filter targets the schema definition entries.
- **-s attributetypes:** This is the attribute(s)we  want to retrieve from the entries that match the search filter. In this case, we 're requesting the **attributetypes** attribute.

**6. Create Custom attribute according to our requirement**

 a. Create customer attribute ldif file

     vim custom_attribute.ldif

```bash
dn: cn=schema
changetype: modify
add: attributeTypes
attributetypes: (emp_code-oid NAME  'EmployeeCode' DESC 'EmployeeCode' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{10} SINGLE-VALUE X-ORIGIN 'user defined' )
attributetypes: (gender-oid NAME 'Gender' DESC 'Gender' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{8} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (certifications-oid NAME 'Certifications' DESC 'Certifications' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{10} X-ORIGIN 'user defined')
attributetypes: (passport-oid NAME 'PassportNo'  DESC 'PassportNo.' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{10}  SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (pan_no-oid NAME 'Panno' DESC 'Panno.' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{10}  SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (qualification-oid NAME 'Qualification' DESC 'Qualification' EQUALITY caseIgnoreMatch  SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{10} X-ORIGIN 'user defined')
attributetypes: (correspondence_address-oid  NAME 'CorrespondenceAddress' DESC 'CorrespondenceAddress' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{100} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (personalemail-id-oid NAME 'personalemail-id' DESC 'personalemail-id' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{20} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (facebookaccount-oid NAME 'facebookaccount' DESC 'facebookaccount' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch  SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{20} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (twitteraccount-oid NAME 'twitteraccount' DESC 'twitteraccount' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{15} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (MaritalStatus-oid NAME 'MaritalStatus' DESC 'MaritalStatus' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{10} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (Childinfo-oid NAME 'Childinfo' DESC 'Childinfo' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{20} X-ORIGIN 'user defined')
attributetypes: (pfno-oid NAME 'pfno' DESC 'pfno' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{20} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (bankname-oid NAME 'BankName' DESC 'BankName' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{20} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (AccountNo-oid NAME 'AccountNo' DESC 'AccountNo' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{20} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (IFSCCode-oid  NAME 'IFSCCode' DESC 'IFSCCode' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{10} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (ESICCardNo-oid NAME 'ESICCardNo' DESC 'ESICCardNo' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{20} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (FamilyMembersInsured-oid  NAME 'FamilyMembersInsured' DESC 'FamilyMembersInsured' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{10} X-ORIGIN 'user defined')
attributetypes: (documentssubmitted-oid  NAME 'documentssubmitted' DESC 'documentssubmitted' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{60} X-ORIGIN 'user defined')
attributetypes: (doj-oid NAME 'DateofJoining' DESC 'DateofJoining' EQUALITY caseIgnoreIA5Match SUBSTR caseIgnoreIA5SubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{8} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (doreg-oid NAME 'DateOfResignation' DESC 'DateOfResignation' EQUALITY caseIgnoreIA5Match SUBSTR caseIgnoreIA5SubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{8} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (dob-oid NAME 'DateofBirth' DESC 'DateofBirth' EQUALITY caseIgnoreIA5Match SUBSTR  caseIgnoreIA5SubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{8} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (passport_valid_upto-oid NAME 'PassportValidupto' DESC 'PassportValidupto' EQUALITY caseIgnoreIA5Match SUBSTR caseIgnoreIA5SubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{8} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (ProfessionalStartYEARS-oid NAME 'ProfessionalStartYEARS' DESC 'ProfessionalStartYEARS' EQUALITY caseIgnoreIA5Match SUBSTR caseIgnoreIA5SubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{8} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (YEARSOfExperience-oid NAME 'YEARSOfExperience' DESC 'YEARSOfExperience' EQUALITY caseIgnoreIA5Match SUBSTR caseIgnoreIA5SubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{8} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (dateofjoiningasintern-oid NAME 'dateofjoiningasintern' DESC 'dateofjoiningasintern'  EQUALITY caseIgnoreIA5Match SUBSTR caseIgnoreIA5SubstringsMatch  SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{8} SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (AadharNo-oid  NAME 'AadhaarNo' DESC 'AadhaarNo' EQUALITY integerMatch SUBSTR caseIgnoreIA5SubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.27{20}  SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (YearsofQualification NAME 'YearsofQualification' DESC 'YearsofQualification' EQUALITY integerMatch  SYNTAX 1.3.6.1.4.1.1466.115.121.1.27{10}  SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (mobileno-oid  NAME 'mobileno' DESC 'mobileno' EQUALITY integerMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.27{20} X-ORIGIN 'user defined')
attributetypes: (UANno-oid NAME 'UANno' DESC 'UANno' EQUALITY integerMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.27{20}  SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: (InsuranceMonthlyAmountDeductionINR-oid NAME 'InsuranceMonthlyAmountDeductionINR' DESC 'InsuranceMonthlyAmountDeductionINR'  EQUALITY integerMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.27{10}  SINGLE-VALUE X-ORIGIN 'user defined')
attributetypes: ( projectname-oid NAME 'ProjectName' DESC 'ProjectName' EQUALITY caseIgnoreMatch SUBSTR caseExactSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{50} SINGLE-VALUE X-ORIGIN 'user defined')
```




b. Add this file to ldap db

```bash
ldadadd -a -c -x -H ldap://localhost:3389 -D "cn=Directory Manager" -W  -f custom_attribute.ldif
```

c. Create **Object class** file for add attribute to object class

```bash
vim  object_class.ldif
```

```bash
dn: cn=schema
changetype: modify
add: objectClasses
objectClasses: ( customEmployee-oid NAME 'customEmployee' SUP top STRUCTURAL MUST ( EmployeeCode $ DateofJoining $ Gender $ DateofBirth $ Panno $ Qualification $ YearsofQualification $ ProfessionalStartYEARS $ YEARSOfExperience $ CorrespondenceAddress $ personalemail-id $ mobileno $ MaritalStatus $ BankName $ AccountNo $ IFSCCode $ documentssubmitted) MAY ( DateOfResignation $ Certifications $ PassportNo $ PassportValidupto $ AadhaarNo $ facebookaccount $ twitteraccount $ Childinfo $ pfno $ UANno $ ESICCardNo $ InsuranceMonthlyAmountDeductionINR $ FamilyMembersInsured $ dateofjoiningasintern $ ProjectName )  X-ORIGIN 'user defined')
```

d. Add object class ldif file

```bash
ldapadd -a -c -x -H ldap://localhost:3389 -D "cn=Directory Manager" -W -f object_class.ldif

```

- **-f object_class.ldif:** This option specifies the name of the LDIF (LDAP Data Interchange Format) file that contains the data to be added to the LDAP server. The -f option denotes the filename

**7. Create user with custom attribute**

```bash
vim user1.ldif
```

```bash
dn: cn=Admins,ou=Support,dc=keenable,dc=in
objectClass: posixGroup
cn: Test
gidNumber: 4000

dn: uid=user1,ou=dev,dc=keenable,dc=in
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: user1
sn: shankar
givenName: user1
cn: user1
uidNumber: 5000
gidNumber: 5000
userPassword: redhat
loginShell: /bin/bash
homeDirectory: /home/user1

```

> In this file we have gave objectclass name and must custom attribute

a. After that add user to db

```bash
ldapadd -a -c -x -H ldap://localhost:3389 -D "cn=Directory Manager" -W  -f user1.ldif
```

- **-f user1.ldif:** This option specifies the name of the LDIF (LDAP Data Interchange Format) file that contains the data to be added to the LDAP server. The -f option is followed by the filename (**user1.ldif** in this case).

b. Check user reflected or not through ldapsearch command

```bash
ldapsearch -x -D "cn=Directory Manager" -W -H ldap://localhost:3389 -b "ou=dev,dc=keenable,dc=in" -s sub "(uid=user1)"
```


- **-b "ou=dev,dc=keenable,dc=in"**: This option specifies the base DN (Distinguished Name) from which the search should start. It's set to **"ou=dev,dc=keenable,dc=in"** which indicates that the search should start from the "ou=dev" organizational unit under the base DN "dc=keenable,dc=in".


**Output look like this**

```bash
ravi@serverldp:~/389ds/data/ldif$ ldapsearch -x -D "cn=Directory Manager" -W -H ldap://localhost:3389 -b "ou=dev,dc=keenable,dc=in" -s sub "(uid=user1)"
Enter LDAP Password: 
# extended LDIF
#
# LDAPv3
# base <ou=dev,dc=keenable,dc=in> with scope subtree
# filter: (uid=user1)
# requesting: ALL
#

# user1, dev, keenable.in
dn: uid=user1,ou=dev,dc=keenable,dc=in
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
objectClass: organizationalPerson
objectClass: top
objectClass: person
uid: user1
sn: shankar
givenName: user1
cn: user1
uidNumber: 5000
gidNumber: 5000
loginShell: /bin/bash
homeDirectory: /home/user1
shadowLastChange: 19600
userPassword:: e1BCS0RGMi1TSEE1MTJ9MTAwMDAkdUZoMTM3RVc0TXprQmwvRkx4dTNmQ21FVFV
 NOHRRcVkkelNzbGI0RURKUzF0YUxFVUF2eHpCZE90TDBCWjJUM25GaXhFR3BzRy9RL3JiOFJkeVpI
 b2EwME1DdW1qeTg0STZ5bExyWXpvSkxuemFRVEIyWVZmV1E9PQ==

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1

```

**8. Setup **ldap Client** on another VM**

b. Install package related to ldap client

```bash
 sudo apt -y install libnss-ldap libpam-ldap ldap-utils
```

- **Libnss-ldap:** It provides the necessary libraries to enable the LDAP (Lightweight Directory Access Protocol) Name Service Switch (NSS) module. This module allows us to use LDAP as a source for user, group, and other system information.
- **libpam-ldap:** It provides the necessary libraries to enable the LDAP Pluggable Authentication Module (PAM) module. This module allows us to use LDAP for user authentication.
- **ldap-utils:** This is a collection of command-line utilities for interacting with LDAP servers. These utilities are useful for managing and querying directory services.

c. After run this command we get one pop up screen

> Enter LDAP URI:  IP address or hostname

> Enter Set a Distinguished name(dn) of the search base

d. Open **/etc/nslcd.conf** and check configuration

```bash
# /etc/nslcd.conf
# nslcd configuration file. See nslcd.conf(5)
# for details.

# The user and group nslcd should run as.
uid nslcd
gid nslcd

# The location at which the LDAP server(s) should be reachable.
uri ldap://192.168.122.109:3389

# The search base that will be used for all queries.
base dc=keenable,dc=in

# The LDAP protocol version to use.
#ldap_version 3

# The DN to bind with for normal lookups.
binddn cn=Directory Manager
bindpw ravi

# The DN used for password modifications by root.
#rootpwmoddn cn=admin,dc=example,dc=com

# SSL options
#ssl off
#tls_reqcert never
tls_cacertfile /etc/ssl/certs/ca-certificates.crt

# The search scope.
#scope sub
```

d. Restart nslcd and nscd service 
```
sudo systemctl restart nslcd
```

```
sudo systemctl restart nscd
```

```bash
reboot
```


**nslcd** stands for Name Service LDAP Client Daemon. It is responsible for querying LDAP directories (such as OpenLDAP) for user and group information and providing it to local processes and services.

e. Run **getent passwd** command to check server user reflect or not

```bash
getent passwd user1
```
output 
```bash
user1:x:5000:5000:user1:/home/user1:/bin/bash
```



**Reference Link:-**

For Understand Ldap:- [https://www.windows-active-directory.com/active-directory-ldap.html](https://www.windows-active-directory.com/active-directory-ldap.html)

For Attribute Syntax:- [https://ldap.com/attribute-syntaxes/](https://ldap.com/attribute-syntaxes/)

For Client Setup:-

[https://computingforgeeks.com/how-to-configure-ubuntu-as-ldap-client/?expand_article=1&expand_article=1](https://computingforgeeks.com/how-to-configure-ubuntu-as-ldap-client/?expand_article=1&expand_article=1)

</aside>
