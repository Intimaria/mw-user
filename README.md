# Role mw-user

Este role maneja los usuarios de mikroways y sus claves públicas de forma
consistente. Para ello, al incluir el role, por defecto se creará una cuenta
mikroways por defecto con las claves públicas de todos los integrantes
disponibles en el repositorio [git de claves ssh](https://gitlab.com/mikroways/public/ssh_keys/).

## Requerimientos

Este role depende de otros pero deberían instalarse automáticamente con
`ansible-galaxy`.

## Variables

Podemos personalizar el role mediante las siguientes variables que se describen
a continuación:

* **`mw_user_create_one_account_per_user`:** su valor por defecto es `false`. Si es
  true creará una cuenta para cada usuario. Sino solamente creará la cuenta
`mw_user_name`.
* **`mw_user_name`:** por defecto es el nombre `mikroways`. Esta cuenta siempre
  será creada por este role. Todas las claves públicas de los miembros de
mikroways seleccionados serán autorizados a usar esta cuenta.
* **`mw_user_url`:** por defecto es `https://mikroways.gitlab.io/public/ssh_keys/_users`. Esta url devuelve un listado de usuarios válidos de mikroways.
* **`mw_user_key_url`:** url template para descargar la clave pública de un
  usuario. El valor por defecto es `https://mikroways.gitlab.io/public/ssh_keys/%user%.pub`. Puede observarse que el valor de `%user%` es dinámico y será reemplazado por el valor de un usuario válido de mikroways.
* **`mw_user_enabled_users`:** lista de usuarios habilitados. Si es vacío, todos
los usuarios válidos de mikroways (los listados en `mw_user_url`) serán
considerados. Caso contrario se realizará la intersección de usuarios válidosm
con los usernames dados en esta lista.
* **`mw_user_template`:** diccionario modelo de cómo será creado cada usuarioen
  el sistema. El modelo de diccionario depende del role [grog.management-user](https://github.com/GROG/ansible-role-management-user)
* **`mw_users_to_remove`:** lista de usuarios que deben ser eliminados. Si un usuario en esta lista fue creado previamente, será eliminado automáticamente del sistema. Además, si las claves públicas de estos usuarios están autorizadas en la cuenta compartida `mw_user_name`, también se eliminarán de la lista `authorized_keys`.

### Configuración para usuarios y administrador de cliente
* **`add_client_accounts`:** su valor por defecto es `false`. Si es true, se creará una o más cuentas para el cliente. Si es false, se omite la sección para el cliente.
* **`client_user_create_one_account_per_user`:** su valor por defecto es `false`. Si es true, se creará una cuenta para cada usuario encontrado en el directorio especificado por client_ssh_key_directory. Si es false, no se crearán cuentas individuales para los usuarios, solo el admin user.
* **`client_admin_name`:** por defecto es el nombre `admin`. Esta cuenta siempre será creada si se habilita la opción `add_client_accounts`. Todas las claves públicas asociadas se agregarán a esta cuenta.
* **`client_ssh_key_directory`:** por defecto este directorio es client_ssh_keys, en la sección `files`. Este directorio debe contener las claves públicas (archivos con extensión .pub) para los usuarios del cliente. Cada subdirectorio dentro de este directorio representa un usuario, y cada archivo .pub dentro de esos subdirectorios es una clave pública asociada a ese usuario.
* **`client_user_template`:** diccionario modelo de cómo será creado el usuario administrador del cliente en el sistema. Este modelo de diccionario depende del role
  [grog.management-user](https://github.com/GROG/ansible-role-management-user) y puede ser personalizado para definir shell, permisos sudo, y otras configuraciones.


## Ejemplo

Crear un archivo de requerimientos de galaxy `requirements.yml` con el siguiente
contenido:

```yaml
# from GitLab or other git-based scm
- name: mikroways.mw_user
  src: git@gitlab.com:mikroways/ansible/mw-user.git
  scm: git
  version: "1.0.0" 
```

Luego, en un playbook es posible invocar el role usando:

```yaml
- name: Some useful playbook
  hosts: all
  gather_facts: true
  tasks:
    - import_role:
        name: mikroways.mw_user
      become: true
      tags:
        - user
```

## TODO

* [x] Eliminar usuarios que ya no trabajan con nosotros. Pensaba que en el repo
  de nuestras claves, podemos agregar quienes se fueron y han trabajado con
  nosotros. De esta forma el playbook debe eliminar estos usuarios si fueron
  creados previamente
* [ ] Mejorar los tests. Faltan probar algunos casos
