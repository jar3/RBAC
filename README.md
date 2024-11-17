# RBAC
RBAC (Control de acceso basado en roles) es un método que se utiliza para controlar quién puede hacer cosas específicas en Kubernetes. A los usuarios se les asignan roles y cada rol les permite realizar determinadas tareas. Es posible que a un usuario se le permita crear nuevos pods en Kubernetes, pero no eliminarlos. Esto ayuda a mantener su sistema Kubernetes seguro y organizado al garantizar que todos solo puedan hacer lo que necesitan.

RBAC es solo una forma de administrar el acceso en sistemas como Kubernetes. También existe ABAC (Control de acceso basado en atributos). ABAC le brinda un control más detallado en situaciones sensibles. A diferencia de RBAC, que solo analiza el rol de un usuario para decidir si puede acceder a algo, ABAC también considera otros factores sobre el usuario y a qué intenta acceder. Esto podría incluir su departamento, el tipo de entorno en el que está trabajando y qué tan sensible es esa área. Sin embargo, en Kubernetes, solo se admite RBAC de forma predeterminada.

¿Cómo funciona RBAC en Kubernetes?
El servidor de API de Kubernetes incluye compatibilidad integrada con RBAC (control de acceso basado en roles). Esto funciona tanto para usuarios normales como para cuentas de servicio. Para usar RBAC en su clúster, primero active la función RBAC. Luego, cree objetos utilizando los recursos de la API rbac.authorization.k8s.io. Hay cuatro tipos principales de estos objetos:
![image](https://github.com/user-attachments/assets/e6f596b1-dc67-4254-9585-e32e9ff30217)
