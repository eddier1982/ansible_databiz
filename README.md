# Capacitación Ansible

## Sesiones realizadas

| Fecha | horas | Tema |
|:----- |:-----|:-----|
|05/ago/24 20:00 - 22:00 | 2 | Intrducción a Ansible, Instalación y relación de confianza |
|08/ago/24 20:00 - 21:00 | 1 | Command Line (ansible ad-hoc), invenatrio básico |
|09/ago/24 20:00 - 21:00 | 1 | Diferencias entre algunos módulos, creación primar playbook |
|13/ago/24 20:00 - 21:00 | 1 | Solución de preguntas |
|20/ago/24 20:00 - 21:00 | 1 | Conceptos básicos de playbook y roles |
|22/ago/24 20:00 - 21:00 | 1 | Ejercicio de práctica, pre-requsitos de instalar Oracle DB |
|29/ago/24 20:00 - 21:00 | 1 | Check de sistaxis de código (ansible lint --check) y uso de ansible-vault |

## Conceptos

| Termino | Descripción |
|:----- |:-----|
| ansible | Automatización de código abierto |
| cml ad-hoc | línea de comandos básicos |
| ansible-core | librería que contiene los paquetes necesarios (como python) para la ejecución de código |
| ansible AWX | versión open source de la versión Anisble Tower |
| Ansible Automation Platform | Software open source de Red Hat que contiene componentes adicionales a la versión AWX |
| playbook | conjunto de tareas utilizando modulos ansible |
| collections | conjunto de playbook desarrollados por la comunidad que ayudan a la ejecución de tareas en un playbook |
| roles | unión de varios playbook's y otros archivos de de apoyo a la ejecución de los mismos |
| YAML/YML | lenguaje de formato standard y de representación de datos estructurados |
| inventarios | fuente de origen de hosts destino |

## Primeros pasos

### Servidor de Ansible

Se requiere una máquina con sistema RHEL 8.6 o superior. Para obtenerla se puede suscripcbir a Red Hat developer [developers.redhat.com](https://developers.redhat.com/) para obtener 10 suscripciones de SO RHEL con una vigencia de 1 año y las cuales se pueden renovar al siguiente año. No tienen soporte. Anisble no está soportado para instalarse sobre Windows.

Una ves se tiene el login anterior, se puede descargar la imagen ISO [access.redhat.com](https://access.cdn.redhat.com/content/origin/files/sha256/39/398561d7b66f1a4bf23664f4aa8f2cfbb3641aa2f01a320068e86bd1fc0e9076/rhel-9.4-x86_64-dvd.iso?user=a6756d7dea19200a6b3d3ceccb01836b&_auth_=1722924666_110a9310f42357463d661f9d170a52e5)

#### Procedimientos iniciales en el servidor para Ansible

 - Instalción básica de SO, con sepra ciones de filesystem basadas en LVM (con excepción del boot y se suguiere un /home de 20GB independiente). Tener al meno 4Gb de RAM y 2 CPU.

*Particionamiento sugerido*

```
boot          500 Mb
/             20  Gb
/home         20  Gb
/tmp          5   Gb
/var          10  Gb
/var/log      5   Gb
/tmp          5   Gb
```

 - **Atachar una subscripción**, se utiliza el usuario de redhat developer y la contraseña asignada

```
susubscription-manager register --username user
```

 - **Asignar la suscripción**. Este proceso se debe realizar desde el [portal de red hat en la sección de suscripciones](https://access.redhat.com/management/subscriptions). Allí se debe selecionar el systema que anteriormente se registró (validar que el hostname de la máquina sea el correcto, de lo contrario corregirlo y repetir el proceso de suscripción). Una ves se seleccione se se le atacha la suscripción de de RHEL disponible

 <img src="images/rh-access_subscription.png" style="width:600px;height:200px;display: block; margin: 0 auto">    

***Nota de buena práctica*** 

  **Crear un usuario en la máquina de anisble con privilegios de sudo a root sin password. Este usuario tambien se recomienda estar creado en los sistemas remotos linux a los que se va a llegar.** Las líneas a continuación resumen cómo crear el usuario

```
root@hostname ~ % useradd username
root@hostname ~ % su - username
root@hostname ~ % exit
root@hostname ~ % touch //etc/sudoers.d/username
root@hostname ~ % visudo /etc/sudoers.d/eocampo
                  %username	ALL=(ALL)	NOPASSWD:ALL
```


 - **Instalación de ansible-core**. En este proceso se realiza la instalación de ansible, como se ve a continuación:

```
dnf install ansible-core
```

 - **Generar relación de confianza localmente**. Estando en la máquina de ansible, se asegura la relación de confianza consigo mismo. Primero generar la llave privada y pública del usuario y luego se estable la relación de confianza.

```
ssh-keygen  ## generar la llave
ssh-copy-id username@hostname_local ## relación de confianza local
```

***Nota de buena práctica*** 

  **Utilizar el fqdn completo de la máquina. Si no se cuenta con DNS se debe registrar la IP con el hostname del fqdn completo en /etc/hosts**


## Invenatarios

La siguiente es la estructura de un inventario para ansible. NOTA: no lleva extensión.

```
estoesunaprueba ansible_host=192.168.241.130
test2.eocampo.lab
test3.eocampo.lab

[apache]
estoesunaprueba
test3.eocampo.lab

[so]
estoesunaprueba
test3.eocampo.lab
test2.eocampo.lab

[db]
test3.eocampo.lab
test2.eocampo.lab

[erp:children]
apache
db
```
***Nota de buena práctica*** 

  **Se pueden agregar variables a cada hosts, como en el ejemplo exista la variable que determina la IP del host. También se pueden hacer grupos y subgrupos** [doc ansible inventory](https://docs.ansible.com/ansible/latest/getting_started/get_started_inventory.html)

Documentación [Building Ansible inventories](https://docs.ansible.com/ansible/latest/inventory_guide/index.html) 

## Ansible ad-hoc (command line)

### Estructura general cml

```
ansible -i {ruta del inventario} -m {comando Ansible} -a '{parámetros de so o de ansible}' {máquina o grupo} [--become] (become si requiere ejeuctar como root)
```

### Ejemplos con módulos:

```
# shell
ansible -i inventory -m shell -a 'cat /etc/fstab' prdrhel
ansible -i inventory -m shell -a 'ls -la /tmp/javasharedresources' prdrhel
ansible -i inventory -m shell -a 'chmod o-w /data' prdal 
ansible -i inventory -m shell -a 'touch /data/prueba' prdal 
ansible -i inventory -m shell -a 'rm -rf /data/prueba' prdal 

# lineinfile
ansible -i inventory -m lineinfile -a 'path={ruta} line="{expresión o línea}" create=yes'
ansible -i inventory -m lineinfile -a 'path={ruta} line="{expresión o línea}" state=absent'

# yum
ansible -i inventory -m yum -a 'name="telnet" state=present' drrhel --become
ansible -i inventory -m yum -a 'name="telnet" state=absent' drrhel --become
ansible -i inventory -m yum -a 'name=tlenet state=list' WUE1ATPSPFIMAP1 --become
ansible -i inventory -m yum -a 'name=rsync state=absent' WUE1ATPSPFIMAP1 --become

# file

ansible -i inventory -m file -a 'path=/etc/cron.allow state=touch mode=600' secrhel --become

# copy
ansible -i inventory -m copy -a 'src=/home/test1/all_logs.sh dest=/tmp/all_logs.sh owner=root group=root mode=640' other --become

# blockinfile
ansible -i inventory -m blockinfile -a 'path={ruta} block="| prueba "'

# cron
ansible -i inventory -m cron -a 'name="Integrity FS" minute="0" hour="5" user=root job="/usr/bin/aide.wrapper --config /etc/aide/aide.conf --check"' localhost

#### user
ansible -i inventory.yml -m user -a 'name=test1 groups=adminso' prdrhel

#### ansible.builtin.group
ansible -i inventory.yml -m ansible.builtin.group -a 'name=adminso state=present' drrhel
```

## Playbooks

La siguiente estructura de comando es como se ejecuta un playbook
```
ansible-playbook path_playbook.yml -i path_inventory
```

### Mi primer playbook

       ---
       - name: Mi primer playbook
         hosts: all
         gather_facts: false
         become: yes

         tasks:

         - name: Instalar Httpd
           ansible.builtin.dnf:
             name: httpd
             state: latest

         - name: Validar si el firewall está arriba
           ansible.builtin.service:
             name: firewalld
             state: started
           register: firewalld_status

         - name: Restart firewalld if not running
           ansible.builtin.service:
             name: firewalld
             state: restarted
           when: not firewalld_status.status.ActiveState == "active"

### Características de un playbook

El poder de Ansible es que puedes usar playbooks para ejecutar múltiples y complejas tareas a un conjunto de hosts objetivo de una manera fácilmente y repetible.

Una *task* es la aplicación de un módulo para realizar un trabajo específico. Un *play* es una secuencia de *tasks* que se aplican, en orden, a uno o más hosts seleccionados de su inventario. Un *playbook* es un archivo de texto que contiene una lista de una o más *playbooks* para ejecutar en un orden específico.

Las secuencias le permiten transformar un conjunto largo y complejo de tareas administrativas manuales en una rutina fácilmente repetible con resultados predecibles y satisfactorios. En un playbook, puede guardar la secuencia de tareas de un play en un formato legible por el ser humano y de ejecución inmediata. Las propias tareas, por la forma en que están escritas, documentan los pasos necesarios para desplegar su aplicación o infraestructura.

**Ejemplo**


```
---
- name: Configure important user consistently
  hosts: servera.lab.example.com
  tasks:
    - name: Newbie exists with UID 4000
      ansible.builtin.user:
        name: newbie
        uid: 4000
        state: present
```


Un playbook es un archivo de texto escrito en formato YAML y normalmente se guarda con la extensión .**yml**. El playbook utiliza sangría con caracteres de espacio para indicar la estructura de sus datos. YAML no impone requisitos estrictos sobre cuántos espacios se utilizan para la sangría, pero se aplican dos reglas básicas:
  
  - Los elementos de datos situados en el mismo nivel jerárquico (por ejemplo, los elementos de una misma lista) deben tener la misma sangría.
  - Los elementos que son hijos de otro elemento deben tener más sangría que sus padres. También puede añadir líneas en blanco para facilitar la lectura.

***Importante: Sólo puede utilizar caracteres de espacio para la sangría; no utilice caracteres de tabulación.***

Un playbook inicia con una línea formada por tres guiones (---) para indicar el inicio del documento. Puede terminar con tres puntos (...) para indicar el final del documento, aunque en la práctica se suele omitir.

Entre esos marcadores, el playbook se define como una lista de plays. Un elemento de una lista YAML comienza con un guión seguido de un espacio. Por ejemplo, una lista YAML podría tener el siguiente aspecto:

```
- apple
- orange
- grape
```

En el ejemplo de playbook anterior, la línea después de --- comienza con un guión e inicia el primero (y único) play de la lista de plays.
El play en sí es una colección de pares llave-valor. Las llaves del mimo play deben tener la misma sangría. El siguiente ejemplo muestra un fragmento YAML con tres llaves. Las dos primeras llaves tienen valores simples. La tercera tiene como valor una lista de tres elementos.

```
name: just an example
hosts: webservers
tasks:
  - first
  - second
  - third
```

En el play inicial de ejemplo tiene tres llaves: nombre, hosts y tareas. Todas estas llaves tienen la misma sangría.
La primera línea del play de ejemplo comienza con un guión y un espacio (indicando que el play es el primer elemento de una lista), y luego la primera llave, nombre. La llave nombre asocia una cadena arbitraria al play como una etiqueta que identifica el propósito del play. La llave name es opcional, pero se recomienda porque ayuda a documentar tu playbook. Esto es especialmente útil cuando un playbook contiene múltiples plays.

```
- name: Configure important user consistently
```

La segunda llave del play es una llave hosts, que especifica los hosts contra los que se ejecutan las tareas del play. La llave hosts toma un patrón de hosts como valor, como los nombres de hosts gestionados o grupos en el inventario.

```
hosts: servera.lab.example.com
```

Finalmente, la última llave en el play es tasks, cuyo valor especifica una lista de tareas a ejecutar para este play. Este ejemplo tiene una única tarea, que ejecuta el módulo ansible.builtin.user con argumentos específicos (para asegurar que el usuario newbie existe y tiene el UID 4000).

```
tasks:
  - name: newbie exists with UID 4000
    ansible.builtin.user:
      name: newbie
      uid: 4000
      state: present
```

La llave de tareas es la parte del play que realmente enumera, en orden, las tareas que deben ejecutarse en los hosts gestionados. Cada tarea de la lista es en sí misma una colección de pares llave-valor.
En este ejemplo, la única tarea del play tiene dos llaves:
 
 - name es una etiqueta opcional que documenta el propósito de la tarea. Es una buena idea nombrar todas sus tareas para ayudar a documentar el propósito de cada paso del proceso de automatización.
 - ansible.builtin.user es el módulo a ejecutar para esta tarea. Sus argumentos se pasan como una colección de pares llave-valor, que son hijos del módulo ( name, uid, y state).

El siguiente es otro ejemplo de una llave de tareas con múltiples tareas, cada una usando el módulo ansible.builtin.service para asegurar que un servicio debe iniciarse en el arranque:

```
tasks:
  - name: Web server is enabled
    ansible.builtin.service:
      name: httpd
      enabled: true
  - name: NTP server is enabled
    ansible.builtin.service:
      name: chronyd
      enabled: true
  - name: Postfix is enabled
    ansible.builtin.service:
      name: postfix
      enabled: true
```
Documentación de [pre_tasks & post_tasks](https://www.redhat.com/sysadmin/ansible-pretasks-posttasks)

***Importante: El orden en el que se enumeran los plays y las tasks en un playbook es importante, porque Ansible los ejecuta en el mismo orden.***

### Módulos para las tareas

Los módulos son las herramientas que los plays utilizan para realizar tareas. Se han escrito cientos de módulos que hacen cosas diferentes. Normalmente puedes encontrar un módulo probado y de propósito especial que haga lo que necesitas, a menudo como parte del entorno de ejecución de automatización por defecto.

El paquete ansible-core incluye una única colección de contenidos de Ansible llamada **ansible.builtin.** Estos módulos están siempre disponibles. Visite [https://docs.ansible.com/ansible/latest/collections/ansible/builtin/](https://docs.ansible.com/ ansible/latest/collections/ansible/builtin/) para ver una lista de los módulos que contiene la colección ansible.builtin.


Documentación [Using Ansible playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/index.html)

### Comando de verificación

Al comento de ejecutar un playbook puede aumentar el ouput de la ejecución agregando:

| Opción | Descripción |
|:-------|:------------|
| -v | Muestra los resultados de la tarea |
| -vv | Muestra los resultados de la tarea y la configuración de la tarea |
| -vvv | Muestra información extra de conexión |

También se puede utilizar la línea de comandos agregando extra variables para realizar check, por ejemplo

```
ansible-playbook name_playbook.yml -i path/inventory --check 
ansible-navigator run -m stdout  name_playbook.yml --syntax-check
```

Para facilitar la validación del código tambien se cuenta con la herramienta **ansible-lint**. Si se tiene los repositorios correctos de RHEL 9 y la suscripción activa, bastará con solo ejecutar

```
sudo dnf install ansible-lint
```

También puede hacer la instalación utilizando phyton como está en este [https://ansible.readthedocs.io/projects/lint/installing/](https://ansible.readthedocs.io/projects/lint/installing/). Un ejemplo de ejecución:

```
[user@localhost ansible]$ ansible-lint change_pass_win.yml 
WARNING  Overriding detected file kind 'yaml' with 'playbook' for given positional argument: change_pass_win.yml
WARNING  Listing 1 violation(s) that are fatal
syntax-check: couldn't resolve module/action 'ansible.windows.win_shell'. This often indicates a misspelling, missing collection, or incorrect module path.
change_pass_win.yml:16:7 [WARNING]: provided hosts list is empty, only localhost is available. Note that
the implicit localhost does not match 'all'
ERROR! couldn't resolve module/action 'ansible.windows.win_shell'. This often indicates a misspelling, missing collection, or incorrect module path.

The error appears to be in '/home/user/ansible/change_pass_win.yml': line 16, column 7, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:


    - name: Listar usuarios de dominio en el server
      ^ here



Finished with 1 failure(s), 0 warning(s) on 1 files.

```

Otra herramienta de ayuda es **ansible-vault**, lo que permite encriptar de manera segura data sensible como por ejemplo passwords. Al momento de crear solicitará un password para el archivo, así:

```
user@localhost ~ % ansible-vault create credentials.yml
New Vault password: 
Confirm New Vault password:

```
Allí deberá estar el contenido relación llave valor, sin comillas, ejemplo: **mypassword: M1p4ssw0rd**

NOTA: debe ser un archivo en fomrato YAML

Al tratar de validar el contenido del archivo por SO o por otra aplicación, estre trendrá una salida así:

```
user@localhost ~ % cat credentials.yml 
$ANSIBLE_VAULT;1.1;AES256
61316330663230633439386233383932393665616334383132643033353831313030323263333331
3861656565653063623039653166623066636332656530390a366336373862386461643933393836
30306335633132376666303434333632623038373066343731373039653466326231653666343764
6431653137383231620a666535306266346330393131383031393037356134303330653831313362
6361353762313732656333396637356365613036383039393165616166616436623
```
Para ver la información del archivo, es solo invocar el vault con la opción de view/edit y suministrando el password podrá verlo o editarlo:

```
user@localhost ~ % ansible-vault view credentials.yml
Vault password: 
mypassword: M1p4ssw0rd
```

Para ejecutar un playbook utilizando un archivo encriptado, utilice la siguiente sintaxis:

```
ansible-playbook  ssh-config.yaml --ask-vault-pass
```

NOTA: Deberá disponer de una declaración de variables en su playbook.

Documentación [Ansible Vault](https://docs.ansible.com/ansible/2.9/user_guide/vault.html)

## Roles

Ansible ofrece la capacidad de crear un custom de coleccion de playbooks o tareas, para ello está la herramienta **ansible-galaxy** la que nos permitirá crear un template de la siguiente manera:

```
user@localhost roles % ansible-galaxy init pruebas1
- Role pruebas1 was created successfully 
user@localhost pruebas1 % cd pruebas 1 \ tree
.
├── README.md
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml

```

Con el siguiente proósito:

| Subdirectorio | Función |
| :------------ | :------ |
| defaults      | El archivo main.yml de este directorio contiene los valores por defecto de las variables de rol que pueden sobrescribirse cuando se utiliza el rol. Estas variables tienen baja precedencia y están pensadas para ser cambiadas y personalizadas en los playbook |
| files         | Este directorio contiene archivos estáticos a los que hacen referencia las tareas de rol |
| handlers      | El archivo main.yml en este directorio contiene las definiciones de los manejadores del rol |
| meta          | El archivo main.yml en este directorio contiene información sobre el rol, incluyendo autor, licencia, plataformas y dependencias opcionales del rol |
| tasks         | El archivo main.yml de este directorio contiene las definiciones de tareas del rol |
| templates     | Este directorio contiene plantillas Jinja2 que son referenciadas por las tareas de rol |
| tests         | Este directorio puede contener un inventario y un playbook test.yml que se puede utilizar para probar el rol |
| vars          | El archivo main.yml en este directorio define los valores de las variables del rol. A menudo estas variables se usan para propósitos internos dentro del rol. Estas variables tienen una alta precedencia y no están pensadas para ser cambiadas cuando se usan en un playbook |

Documentación [Roles](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html)