# aps-modo-cluster

info de aps modo cluster instalation

# EN TODOS LOS NODOS SE INSTALA

JDK 17
TOMCAT

# Nodo 1

RRQUISITO
JDK 17
Postgress
elastic search
activit admin
activi nodo1

-- creamos los users
create role alfresco login password 'alfresco';
create database activiti encoding 'utf8';
grant all on database activiti to alfresco;
create database activitiadmin encoding 'utf8';
grant all on database activitiadmin to alfresco;

habilitamos el firewall

# ] systemctl status firewalld
# ] firewall-cmd --zone=public --add-port=7070/tcp --permanent

# (En el nodo donde se instalará Elasticsearch, ejecutar además:)

# ] firewall-cmd --zone=public --add-port=9200/tcp --permanent
# ] firewall-cmd --reload
# ] firewall-cmd --list-all    (Para verificar puertos abiertos)

# ] adduser alfresco

 Ajuste de límites de archivos abiertos: Editar el archivo /etc/security/limits.conf en cada nodo para aumentar los límites de descriptores de archivos para el usuario alfresco. Añadir al final del archivo las siguientes líneas:

    alfresco    soft    nofile    40000
alfresco    hard    nofile    65536

- Servidor nfs
Los pasos generales para configurar el almacenamiento compartido son:
 • En el servidor NFS: Puede ser uno de los nodos APS (e.g. Nodo1) o un servidor dedicado de archivos. Instalar los paquetes de NFS si no están presentes (por ejemplo, yum install -y nfs-utils). Definir un directorio a compartir, por ejemplo /opt/alfresco/alfresco-process-services/compartido (puede ser otra ruta según convenga). Establecer la exportación NFS editando /etc/exports:

/opt/alfresco/alfresco-process-services/compartido  <IP_Nodo1>(rw,sync,no_root_squash) <IP_Nodo2>(rw,sync,no_root_squash)

El ejemplo anterior asume que crearemos el punto compartido bajo /opt/alfresco/alfresco-process-services/compartido en el servidor NFS, otorgando acceso de lectura/escritura a las IP de Nodo1 y Nodo2. Tras agregar la línea, exportar el recurso ejecutando:

```bash
exportfs -r
systemctl start nfs-server
systemctl enable nfs-server
```

 • En los nodos APS (clientes NFS): Instalar los utilitarios NFS si es necesario (yum install -y nfs-utils). Crear el punto de montaje donde se enlazará el recurso compartido. En nuestro caso, APS usará la carpeta act_data/data para contenido, así que podemos montar el NFS directamente en esa ubicación. Primero, crear la ruta en cada nodo APS:

```bash
mkdir -p /opt/alfresco/alfresco-process-services/act_data/data
```

Luego, montar el recurso NFS exportado en esa ruta. Por ejemplo, si usamos Nodo1 como servidor NFS y hemos compartido /opt/alfresco/alfresco-process-services/compartido, montarlo en Nodo1 y Nodo2 así:

#] mount -t nfs <IP_Servidor_NFS>:/opt/alfresco/alfresco-process-services/compartido \
     /opt/alfresco/alfresco-process-services/act_data/data

Verificar que el contenido montado es accesible en ambos nodos (por ejemplo creando un archivo de prueba en la ruta y comprobando que aparece en el otro nodo). Para que el montaje sea permanente, agregar una entrada en /etc/fstab en ambos nodos APS, por ejemplo:

<IP_Servidor_NFS>:/opt/alfresco/alfresco-process-services/compartido  /opt/alfresco/alfresco-process-services/act_data/data  nfs  defaults  0 0