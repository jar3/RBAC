# RBAC
RBAC (Control de acceso basado en roles) es un método que se utiliza para controlar quién puede hacer cosas específicas en Kubernetes. A los usuarios se les asignan roles y cada rol les permite realizar determinadas tareas. Es posible que a un usuario se le permita crear nuevos pods en Kubernetes, pero no eliminarlos. Esto ayuda a mantener su sistema Kubernetes seguro y organizado al garantizar que todos solo puedan hacer lo que necesitan.

RBAC es solo una forma de administrar el acceso en sistemas como Kubernetes. También existe ABAC (Control de acceso basado en atributos). ABAC le brinda un control más detallado en situaciones sensibles. A diferencia de RBAC, que solo analiza el rol de un usuario para decidir si puede acceder a algo, ABAC también considera otros factores sobre el usuario y a qué intenta acceder. Esto podría incluir su departamento, el tipo de entorno en el que está trabajando y qué tan sensible es esa área. Sin embargo, en Kubernetes, solo se admite RBAC de forma predeterminada.

### ¿Cómo funciona RBAC en Kubernetes?

El servidor de API de Kubernetes incluye compatibilidad integrada con RBAC (control de acceso basado en roles). Esto funciona tanto para usuarios normales como para cuentas de servicio. Para usar RBAC en su clúster, primero active la función RBAC. Luego, cree objetos utilizando los recursos de la API rbac.authorization.k8s.io. Hay cuatro tipos principales de estos objetos:
![image](https://github.com/user-attachments/assets/e6f596b1-dc67-4254-9585-e32e9ff30217)


Role es un grupo de permisos. Estos permisos permiten a los usuarios realizar determinadas acciones (como obtener, crear o eliminar) en tipos específicos de recursos de Kubernetes (como pods, implementaciones y espacios de nombres). Los roles están vinculados a un espacio de nombres en particular, lo que significa que los permisos que otorgan solo son válidos en ese espacio de nombres.

ClusterRole es similar a Roles, pero se usa para recursos a nivel de clúster, no solo dentro de un único espacio de nombres. Use un ClusterRole para elementos como Nodos, que no son parte de ningún espacio de nombres. Los ClusterRoles también pueden brindar acceso a recursos con espacios de nombres en todos los espacios de nombres, como cada Pod en su clúster.

RoleBinding es lo que conecta sus Roles con Usuarios o Cuentas de Servicio. Un RoleBinding hace referencia a un Rol y luego otorga esos permisos a uno o más usuarios (llamados Sujetos).

ClusterRoleBinding hace lo mismo que un RoleBinding pero para ClusterRoles. Vincula ClusterRoles con usuarios o grupos.

### Usuarios vs. Cuentas de servicio
Antes de configurar RBAC, es importante comprender el modelo de usuario de Kubernetes. Hay dos formas de crear usuarios según el tipo de acceso que se requiere:

Usuarios En Kubernetes, un usuario representa a un humano que se autentica en el clúster mediante un servicio externo. Puede utilizar claves privadas (el método predeterminado), una lista de nombres de usuario y contraseñas, o un servicio OAuth como Cuentas de Google. Los usuarios no son administrados por Kubernetes; no hay ningún objeto API para ellos, por lo que no puede crearlos mediante kubectl. Debe realizar cambios en el proveedor de servicios externo.

Las cuentas de servicio son valores de token que se pueden usar para otorgar acceso a espacios de nombres en su clúster. Están diseñadas para que las usen las aplicaciones y los componentes del sistema. A diferencia de los usuarios, las cuentas de servicio están respaldadas por objetos de Kubernetes y se pueden administrar mediante la API.

Crearemos una cuenta de servicio para representar a nuestro usuario en el siguiente tutorial. Sin embargo, vale la pena recordar que debes seguir los pasos de la documentación de Kubernetes para vincular tu proveedor de identidad existente si vas a agregar varios usuarios humanos a tu clúster.

### Configuración de RBAC en Kubernetes
A continuación, se muestra un ejemplo de uso de RBAC en Kubernetes. Primero, crearemos una cuenta de servicio y luego le asignaremos un rol. Este rol permitirá que la cuenta enumere, cree y cambie pods, pero solo en un espacio de nombres en particular. La cuenta no tendrá permiso para hacer otras cosas, trabajar con diferentes tipos de recursos ni acceder a otros espacios de nombres.

El control de acceso basado en roles es una característica de Kubernetes que puedes elegir usar, pero generalmente ya está activada en la mayoría de las versiones más comunes. Para ver si está disponible, puedes ejecutar un comando llamado 
`kubectl api-versions`.Si ves `rbac.authorization.k8s.io` en la lista que aparece, entonces puedes comenzar a usar RBAC en tu configuración.

`kubectl api-versions | grep rbac`

Para habilitar manualmente la compatibilidad con RBAC, debe iniciar el servidor de API de Kubernetes con el indicador --authorization-mode=RBAC establecido:

`kube-apiserver --authorization-mode=RBAC`

### Crear una cuenta de servicio
Debe crear una cuenta de servicio para usarla en los próximos pasos. Vinculará el rol que cree a esta cuenta de servicio:

```$ kubectl create serviceaccount demo-user`
serviceaccount/demo-user created```

A continuación, ejecute el siguiente comando para crear un token de autorización para su cuenta de servicio:

`$ kubectl create serviceaccount demo-user
serviceaccount/demo-user created`

A continuación, ejecute el siguiente comando para crear un token de autorización para su cuenta de servicio:

`$ TOKEN=$(kubectl create token demo-user)`


El valor del token ahora se guardará en la variable de entorno $TOKEN en su terminal.

### Configurar kubectl con su cuenta de servicio
Ahora agregue un nuevo contexto de kubectl que le permita autenticarse como su cuenta de servicio. Primero, agregue su cuenta de servicio como credencial en su archivo Kubeconfig:

`$ kubectl config set-credentials demo-user --token=$TOKEN
User "demo-user" set.`











