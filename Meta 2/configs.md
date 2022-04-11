# Assignment 2 - Configs

# LDAP

## Coimbra Machine
```bash
sudo apt install slapd ldap-utils
dkpkg-reconfigure slapd
    > Domain gisi.pt
    > Org Domain gisi.pt

    ldapadd -x -D "cn=admin,dc=gisi,dc=pt" -W -H ldapi:// -f OUs.ldif
    ldapadd -x -D "cn=admin,dc=gisi,dc=pt" -W -H ldapi:// -f groups.ldif
    ldapadd -x -D "cn=admin,dc=gisi,dc=pt" -W -H ldapi:// -f users.ldif

```

# Installing email services

``` bash
apt install postfix dovecot-imapd dovecot-pop3d dovecot-lmtpd
```

# Check Report 2 for more info