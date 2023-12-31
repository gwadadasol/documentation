<!-- ------------------------------------- -->
<!-- instruction to create the certificate -->
<!-- ------------------------------------- -->

KEYFILE=$TMP_PATH/dotnet-devcert.key
CRTFILE=$TMP_PATH/dotnet-devcert.crt
PFXFILE=$TMP_PATH/dotnet-devcert.pfx


### Create the certificate
```sh
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout dotnet-devcert.key -out dotnet-devcert.crt -config localhost.conf --passout pass:xxxxxxx
```

### convert the certificate 
```sh
openssl pkcs12 -export -out dotnet-devcert.pfx -inkey dotnet-devcert.key -in dotnet-devcert.crt --passout pass:xxxxxxx
```

### Update the DB
#### delete existing certificate
```sh
certutil -d sql:$HOME/.pki/nssdb -D -n dotnet-devcert
```

#### add certificate
```sh
certutil -d sql:$HOME/.pki/nssdb -A -t "CP,," -n dotnet-devcert -i dotnet-devcert.crt
```

### import certificate with the password
```sh
dotnet dev-certs https --clean --import dotnet-devcert.pfx -p "xxxxxxx"
```

### remove existing certificate
```sh
sudo rm /etc/ssl/certs/dotnet-devcert.pem
```

### copy the certificate in the /usr/local/share/ca-certificates directory

```sh
sudo cp dotnet-devcert.crt "/usr/local/share/ca-certificates"
```

### Check that the file has the right 644

### Update store with the new certificate
```sh
sudo update-ca-certificates
```
### Configuration .net application 

####  appsettings.json for DEV env ONLY

```json
{
  "Kestrel": {
    
    "Certificates": {
      "Default": {
        "Path": "/usr/local/share/ca-certificates",
        "PathKey": "PATH/dotnet-devcert.key",
        "Password": "xxxxxxx"
      }
    }
  }
}
```
#### launchSettings.json for DEV env ONLY

```json
{
  "$schema": "https://json.schemastore.org/launchsettings.json",
  "profiles": {
  
    "https": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "launchUrl": "swagger",
      "applicationUrl": "https://localhost:7182;http://localhost:5155",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```
