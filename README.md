# Elasticsearch-Ansible

Parte Teórica (Preguntas de Opción Múltiple)
1. ¿Qué es DevOps?
a) Una herramienta de automatización de tareas.
b) Una cultura y práctica que enfatiza la colaboración entre desarrollo y operaciones. X
c) Una plataforma de orquestación de contenedores.

2. ¿Cuál es el propósito principal de Ansible en el contexto de DevOps?
a) Gestionar contenedores Docker.
b) Automatizar la configuración y el despliegue de sistemas y aplicaciones. X
c) Administrar bases de datos.

3. ¿Cuál es el propósito de Elasticsearch en una infraestructura DevOps?
a) Una herramienta de seguimiento de problemas y errores.
b) Un motor de búsqueda y análisis de datos en tiempo real. X
c) Un servidor de aplicaciones.

4. ¿Por qué es importante utilizar Ansible para instalar Elasticsearch en lugar de configurarlo
manualmente en cada nodo?
a) Acelera el proceso de instalación y configuración. X
b) Garantiza que todos los nodos tengan configuraciones idénticas. X
c) Facilita la depuración de errores.

Documentación instalacion

- name: Configure and Install Elasticsearch on Debian nodes
  hosts: debian_elasticsearch
  become: yes
  tasks:

Paso 1: Encabezado del playbook

name: Nombre del playbook, "Configure and Install Elasticsearch on Debian nodes".
hosts: Los hosts en los que se ejecutará este playbook, en este caso, "debian_elasticsearch".
become: yes: El playbook se ejecutará con privilegios de superusuario.

    - name: Package update
      apt:
        upgrade: dist

Paso 2: Actualización de paquetes

Utiliza el módulo apt para actualizar todos los paquetes del sistema.
    - name: Update package list
      apt:
        update_cache: yes

Paso 3: Actualización de la lista de paquetes

Utiliza el módulo apt para actualizar la lista de paquetes disponibles en los repositorios locales.
    - name: Install required packages
      apt:
        name:
          - default-jre
          - apt-transport-https
          - software-properties-common
        state: present

Paso 4: Instalación de paquetes requeridos

Utiliza el módulo apt para instalar paquetes necesarios como el entorno de ejecución Java (default-jre), 
herramientas de transporte apt (apt-transport-https) y herramientas comunes de administración de 
software (software-properties-common).
  
    - name: Add the Elasticsearch public key
      apt_key:
        url: https://artifacts.elastic.co/GPG-KEY-elasticsearch
        state: present

Paso 5: Agregación de la clave pública de Elasticsearch

Utiliza el módulo apt_key para agregar la clave pública de Elasticsearch al sistema.

    - name: Add the Elasticsearch repository
      apt_repository:
        repo: deb https://artifacts.elastic.co/packages/7.x/apt stable main
        state: present
        
Paso 6: Agregación del repositorio de Elasticsearch

Utiliza el módulo apt_repository para agregar el repositorio de Elasticsearch a las fuentes de paquetes.

    - name: Update package list again
      apt:
        update_cache: yes

Paso 7: Actualización de la lista de paquetes nuevamente

Utiliza el módulo apt para actualizar nuevamente la lista de paquetes disponibles.

    - name: Install Elasticsearch
      apt:
        name: elasticsearch
        state: present

Paso 8: Instalación de Elasticsearch

Utiliza el módulo apt para instalar el paquete "elasticsearch".

    - name: Get the IP of the local machine
      ansible.builtin.setup
      register: local_ip_info

Paso 9: Obtención de la dirección IP de la máquina local

Utiliza el módulo ansible.builtin.setup para obtener información sobre la máquina local y registra el 
resultado en la variable local_ip_info.

    - name: Extract the local IP
      set_fact:
        local_ip: "{{ local_ip_info.ansible_facts['ansible_default_ipv4']['address'] }}"

Paso 10: Extracción de la dirección IP local

Utiliza el módulo set_fact para extraer la dirección IP local del resultado anterior y almacenarla en la variable local_ip.

    - name: Create elasticsearch.yml
      ansible.builtin.copy:
        content: |
          cluster.name: my-cluster
          node.name: {{ inventory_hostname }}
          network.host: {{ local_ip }}
          http.port: 9200
          # Otras configuraciones de Elasticsearch aquí
        dest: /etc/elasticsearch/elasticsearch.yml
      become: yes

Paso 11: Creación del archivo elasticsearch.yml

Utiliza el módulo ansible.builtin.copy para crear el archivo de configuración "elasticsearch.yml" con las configuraciones especificadas.

    - name: Configure the elasticsearch.yml file
      lineinfile:
        path: /etc/elasticsearch/elasticsearch.yml
        line: "{{ item }}"
        insertafter: '^cluster.name:'
      with_items:
        - "node.name: {{ inventory_hostname }}"
        - "path.data: /var/lib/elasticsearch"
        - "path.logs: /var/log/elasticsearch"
        - "network.host: {{ ansible_host }}"
        - "discovery.seed_hosts: {{ groups['debian_elasticsearch'] | join(',') }}"
        - "cluster.initial_master_nodes: {{ groups['debian_elasticsearch'][0] }}"
      notify: Restart Elasticsearch

Paso 12: Configuración del archivo elasticsearch.yml

Utiliza el módulo lineinfile para agregar configuraciones adicionales al archivo "elasticsearch.yml" después de la 
línea que contiene "cluster.name". Notifica al manejador "Restart Elasticsearch" en caso de cambios.

    - name: Enable and start Elasticsearch
      systemd:
        name: elasticsearch
        enabled: yes
        state: started
      when: inventory_hostname in groups['debian_elasticsearch']

Paso 13: Habilitación y inicio de Elasticsearch

Utiliza el módulo systemd para habilitar y poner en marcha el servicio de Elasticsearch en los nodos del grupo "debian_elasticsearch".

  handlers:
    - name: Restart Elasticsearch
      service:
        name: elasticsearch
        state: restarted

Paso 14: Manejador para reiniciar Elasticsearch

Utiliza el módulo service para reiniciar el servicio de Elasticsearch cuando se invoca el manejador.
Este playbook automatiza la instalación y configuración de Elasticsearch en nodos Debian que pertenecen al grupo 
"debian_elasticsearch". Asegúrate de configurar adecuadamente tu inventario de Ansible y de que los grupos y hosts 
estén definidos correctamente antes de ejecutar este playbook.
