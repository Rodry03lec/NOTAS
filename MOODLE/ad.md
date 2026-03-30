# 1.  Habilitar en moodle
### Administración del sitio > Plugins > Autenticación > Gestionar autenticación
### Busca ahi: Servidor LDAP y habilitalo

# 2. Configurar LDAP
### Administración del sitio > Plugins > Autenticación > Servidor LDAP
```typescript
export const configDirectory = {
    url: "ldap://10.1.0.39",
    baseDN: "DC=ine,DC=gov,DC=bo",
    username: "authcnpv@ine.gov.bo",
    password: "********"
}
```

# 3. Configurar
## LDAP server settings
### Host URL : ldap://10.1.0.39
### Version : 3
### Use TLS : No
## Bind settings
### Distinguished name (bind_dn) : authcnpv@ine.gov.bo
### Password (bind_pw) : ********
## User lookup settings
### User type : Active Directory
### Contexts : DC=ine,DC=gov,DC=bo
### Search subcontexts : Yes
### User attribute : sAMAccountName

## Data mapping (muy importante)
### First name : givenName
### Last name : sn
### Email address : mail