# Capacitación Ansible

## Procedimientos

 - Login a redhat developer [developers.redhat.com](https://developers.redhat.com/)
 - Descargar la ISO desde [access.redhat.com](https://access.cdn.redhat.com/content/origin/files/sha256/39/398561d7b66f1a4bf23664f4aa8f2cfbb3641aa2f01a320068e86bd1fc0e9076/rhel-9.4-x86_64-dvd.iso?user=a6756d7dea19200a6b3d3ceccb01836b&_auth_=1722924666_110a9310f42357463d661f9d170a52e5)
 - Atachar una subscripción

        susubscription-manager register --username user

 - Asignar la suscripción        

 - Instalación de ansible-core

        dnf install ansible-core

 - Generar relación de confianza localmente 

        ssh-keygen
        ssh-copy-id
