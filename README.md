# SynologyDirectoryServerSmartCardLogin
Synology Directory Server with Smartcard (PIV) Login for Windows Clients.

I would like to document a Howto for Smartcard-login for Windows 10 clients that join of a Synology Directory Server (Windows Active Directory). For the Smartcard I'm using a UbiKey. Anyone who wants to help?

## Howto for Samba
I've read: https://wiki.samba.org/index.php/Samba_AD_Smart_Card_Login#Allowing_Smart_Card_Login_to_a_Samba4_Domain

## Synology implementation
I typical Samba config on Synology AD is: __./etc/samba/smb.conf__

### smb.conf ###

```shell
[global]
	printcap name = cups
	winbind enum groups = yes
	include = /var/tmp/nginx/smb.netbios.aliases.conf
	workgroup = DOMAIN
	min protocol = SMB2
	security = ads
	local master = no
	realm = DOMAIN.LAN
	netbios name = NAS1
	passdb backend = smbpasswd
	printing = cups
	max protocol = SMB3
	winbind enum users = yes
	load printers = yes
	admin users = @DOMAIN\Domain Admins,@DOMAIN\Enterprise Admins
```

### synoadserver.conf ###

```shell
[global]
    include = /etc/samba/smb.conf
    include = /var/packages/DirectoryServerForWindowsDomain/conf/etc/smb.tls.conf
    server services = rpc,nbt,wrepl,ldap,cldap,kdc,drepl,ntp_signd,kcc,dnsupdate
    realm = DOMAIN.LAN
    netbios name = NAS1
    private dir = /var/packages/DirectoryServerForWindowsDomain/target/private
    server role = active directory domain controller
    workgroup = DOMAIN
    security = auto
    server schannel = Yes
[netlogon]
    read only = No
    edit synoacl = Yes
    path = /var/packages/DirectoryServerForWindowsDomain/target/private/sysvol/domain.lan/scripts
[sysvol]
    read only = No
    edit synoacl = Yes
    path = /var/packages/DirectoryServerForWindowsDomain/target/private/sysvol
```

## OpenSSL ##

OpenSSL Config File: __./etc/ssl/openssl.cnf__

### openssl.cnf ###

```shell
HOME        = .
RANDFILE    = $ENV::HOME/.rnd

[ ca ]
default_ca                  = CA_default

[ CA_default ]
dir                         = /etc/ssl
certs                       = $dir/certs
crl_dir                     = $dir/crl
database                    = $dir/index.txt
new_certs_dir               = $dir/newcerts
certificate                 = $dir/cacert.pem
serial                      = $dir/serial
crlnumber                   = $dir/crlnumber
crl                         = $dir/crl.pem
private_key                 = $dir/private/cakey.pem
RANDFILE                    = $dir/private/.rand
x509_extensions             = usr_cert
name_opt                    = ca_default
cert_opt                    = ca_default
default_days                = 365
default_crl_days            = 30
default_md                  = default
preserve                    = no
policy                      = policy_match

[ policy_match ]
countryName                 = match
stateOrProvinceName         = match
organizationName            = match
organizationalUnitName      = optional
commonName                  = supplied
emailAddress                = optional

[ policy_anything ]
countryName                 = optional
stateOrProvinceName         = optional
localityName                = optional
organizationName            = optional
organizationalUnitName      = optional
commonName                  = supplied
emailAddress                = optional

[ req ]
default_bits                = 2048
default_keyfile             = privkey.pem
default_md                  = sha256
distinguished_name          = req_distinguished_name
attributes                  = req_attributes
x509_extensions             = v3_ca
string_mask                 = utf8only

[ req_distinguished_name ]
countryName                 = Country Name (2 letter code)
countryName_default         = TW
countryName_min             = 2
countryName_max             = 2
stateOrProvinceName         = State or Province Name (full name)
stateOrProvinceName_default = Taiwan
localityName                = Locality Name (eg, city)
localityName_default        = Taipei
0.organizationName          = Organization Name (eg, company)
0.organizationName_default  = Synology Inc.
organizationalUnitName      = Organizational Unit Name (eg, section)
commonName                  = Common Name (e.g. server FQDN or YOUR name)
commonName_max              = 64
emailAddress                = Email Address
emailAddress_max            = 64
emailAddress_default        = product@synology.com

[ req_attributes ]
challengePassword           = A challenge password
challengePassword_min       = 4
challengePassword_max       = 20
unstructuredName            = An optional company name

[ usr_cert ]
basicConstraints            = CA:FALSE
nsComment                   = "Synology Generated Certificate"
nsCertType                  = server, sslCA
subjectKeyIdentifier        = hash
authorityKeyIdentifier      = keyid, issuer

[ v3_req ]
basicConstraints            = CA:FALSE
keyUsage                    = nonRepudiation, digitalSignature, keyEncipherment

[ v3_ca ]
subjectKeyIdentifier        = hash
authorityKeyIdentifier      = keyid:always, issuer
basicConstraints            = CA:true

[ crl_ext ]
authorityKeyIdentifier      = keyid:always
```

## From here ##
From here, the questions are:
1. What is already installed on a Synology with AD running, and what components are missing? I would say: we have a CA, OpenSSL, Samba, DNS, Webserver, etc.
2. Is the SAMBA server already using a certificate like in step [1.6.3 Copy Necessary Files to the Samba Domain Controller](https://wiki.samba.org/index.php/Samba_AD_Smart_Card_Login#Copy_Necessary_Files_to_the_Samba_Domain_Controller)
3. What needs to be configured? Like: [1.6.8 Edit the Samba KDC Configuration File to Enable PKINIT Authentication](https://wiki.samba.org/index.php/Samba_AD_Smart_Card_Login#Edit_the_Samba_KDC_Configuration_File_to_Enable_PKINIT_Authentication)
4. How to setup a CRL Distribution Point, like in step [1.5 Set up the CRL Distribution Point](https://wiki.samba.org/index.php/Samba_AD_Smart_Card_Login#Set_up_the_CRL_Distribution_Point)
5. How to create certificates for each user, like step [1.4.6 Generate Certificates for Each User](https://wiki.samba.org/index.php/Samba_AD_Smart_Card_Login#Generate_Certificates_for_Each_User)
