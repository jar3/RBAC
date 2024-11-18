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

```
$ kubectl create serviceaccount demo-user
serviceaccount/demo-user created
``` 

A continuación, ejecute el siguiente comando para crear un token de autorización para su cuenta de servicio:

```
$ kubectl create serviceaccount demo-user
serviceaccount/demo-user created
```

A continuación, ejecute el siguiente comando para crear un token de autorización para su cuenta de servicio:

`$ TOKEN=$(kubectl create token demo-user)`


El valor del token ahora se guardará en la variable de entorno $TOKEN en su terminal.

### Configurar kubectl con su cuenta de servicio
Ahora agregue un nuevo contexto de kubectl que le permita autenticarse como su cuenta de servicio. Primero, agregue su cuenta de servicio como credencial en su archivo Kubeconfig:

```
$ kubectl config set-credentials demo-user --token=$TOKEN
User "demo-user" set.
```

A continuación, agrega tu nuevo contexto. Lo llamaremos `demo-user-context`. Haz referencia a tu nueva credencial `demo-user` y a tu clúster de Kubernetes actual. (Estamos usando el clúster `default`, pero debes cambiar este valor si estás conectado a un clúster con un nombre diferente).

```
$ kubectl config set-context demo-user-context --cluster=minikube --user=demo-user
Context "demo-user-context" created.
```

Antes de cambiar a su nuevo contexto, primero verifique el nombre de su contexto actual para que pueda volver fácilmente a su cuenta administrativa en la siguiente sección:

```
$ kubectl config current-context
default
```

Ahora, cambia a tu nuevo contexto que se autentica como tu cuenta de servicio:

```
$ kubectl config use-context demo-user-context
Switched to context "demo-user-context".
```

Intenta enumerar los Pods en el espacio de nombres:

```
$ kubectl get pods
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:default:demo-user" cannot list resource "pods" in API group "" in the namespace "default"
```

Se devuelve un error Forbidden porque a su cuenta de servicio no se le han asignado roles RBAC que incluyan el permiso para obtener pods.

Antes de continuar, vuelva a su contexto kubectl original para restaurar sus privilegios de administrador. Esto le permitirá crear sus objetos Role y RoleBinding en la siguiente sección.
```
$ kubectl config use-context default
Switched to context "default"
```

### Creación de un rol
Se requiere un objeto Role (o ClusterRole) para cada uno de los roles que desea usar con Kubernetes. En este tutorial, usamos un rol porque trabajamos con pods, que son recursos con espacios de nombres.

Los manifiestos de roles requieren un campo de reglas que enumera los grupos de API, los tipos de recursos y los verbos que pueden usar los titulares del rol.

A continuación, se incluye un ejemplo que permite las acciones de obtención, lista, creación y actualización en pods:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: demo-role
  namespace: default
rules:
  - apiGroups:
      - ""
    resources:
      - pods
    verbs:
      - get
      - list
      - create
      - update
```

El campo apiGroups se establece en una matriz vacía porque los pods existen en el grupo de nivel superior. Los campos de recursos y verbos enumeran las interacciones permitidas dentro de los grupos de API que ha definido. Debido a que los roles tienen espacios de nombres, los titulares del rol solo pueden acceder a los recursos permitidos dentro del espacio de nombres al que pertenece el rol.

Guarde el manifiesto del rol en role.yaml y luego use kubectl para agregarlo a su clúster:

```
$ kubectl apply -f role.yaml
role.rbac.authorization.k8s.io/demo-role created
```

Cree un RoleBinding
El rol se ha creado, pero aún no está asignado a su cuenta de servicio. Se requiere un RoleBinding para realizar esta conexión.

Los objetos RoleBinding requieren los campos roleRef y subject:

roleRef: identifica el objeto de rol que se está asignando.
subject: una lista de uno o más usuarios o cuentas de servicio a los que se les asignará el rol.
Este es un RoleBinding que asignará el rol recién creado a su cuenta de servicio:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: demo-role-binding
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: demo-role
subjects:
  - namespace: default
    kind: ServiceAccount
    name: demo-user
```

El manifiesto indica que el rol demo-role (dentro del espacio de nombres default) debe asignarse a la cuenta de servicio denominada demo-user. (Si estuviera asignando el rol a un usuario humano, establecería kind: User en lugar de kind: ServiceAccount).

Guarde el manifiesto RoleBinding como rolebinding.yaml y, luego, agréguelo a su clúster para otorgarle el rol a su usuario:

```
$ kubectl apply -f rolebinding.yaml
rolebinding.rbac.authorization.k8s.io/demo-role-binding created
```

Verifique que su cuenta de servicio esté funcionando
Ahora puede probar que su nueva regla RBAC esté funcionando correctamente. Su cuenta de servicio debería poder interactuar con los pods en el espacio de nombres default.

Vuelva al contexto kubectl que se autentica como el usuario de la cuenta de servicio:

```
$ kubectl config use-context demo-user-context
Switched to context "demo-user-context".
```

Verifique que el comando get pods ahora se ejecute correctamente:

```
$ kubectl get pods
No resources found in default namespace.
```

También puede intentar crear un pod como su cuenta de servicio:

```
$ kubectl run nginx --image=nginx:latest
pod/nginx created
```

El pod se creó correctamente:

```
$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          31s
```

Sin embargo, el usuario de la cuenta de servicio no puede eliminar pods porque el rol que ha asignado no incluye el verbo de acción de eliminación requerido:

```
$ kubectl delete pod nginx
Error from server (Forbidden): pods "nginx" is forbidden: User "system:serviceaccount:default:demo-user" cannot delete resource "pods" in API group "" in the namespace "default"
```































