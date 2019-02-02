# Tutorial de Reglas
## Introduccion
Siguiendo las siguientes instrucciones, se espera que el alumno pueda actualizar las reglas de acceso del Blockchain con lo que aprenderá a autorizar o denegar el acceso a transacciones y recursos a perfiles específicos

## 1. El caso de uso
De momento, la red en su estado actual, permite control total por parte del administrador, sin embargo si entraramos a la red como otro participante (por ejemplo un comerciante), veríamos que no podemos acceder a la red ni ver sus contenidos.

Como primera parte de este tutorial, editaremos el archivo de reglas para permitir que los participantes accedan la red y que puedan ver los contenidos de la misma (los participantes y los activos).

## 2. Primera Definición de Reglas
Entremos al editor de reglas y borremos el contenido existente. En su lugar, pondremos el siguiente conjunto de reglas y en su lugar escribiremos lo siguiente:

```
/*
 * Archivo con las reglas de acceso para el tutorial 
 * de blockchain
 */

/**
 * Reglas para el Administrador
 */
rule NetworkAdminUser {
    description: "Grant business network administrators full access to user resources"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: ALL
    resource: "**"
    action: ALLOW
}

rule NetworkAdminSystem {
    description: "Grant business network administrators full access to system resources"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: ALL
    resource: "org.hyperledger.composer.system.**"
    action: ALLOW
}

/**
 * Reglas para los comerciantes
 */

 rule ComercianteVerComerciantes {
    description: "Regla que permite a un comerciante ver a otros comerciantes"
    participant: "org.tutorial.mercancias.Comerciante"
    operation: READ
    resource: "org.tutorial.mercancias.Comerciante"
    action: ALLOW
}

rule ComercianteVeMercancias {
  description: "Regla que permite que un comerciante ver mercancias"
  participant: "org.tutorial.mercancias.Comerciante"
  operation: READ
  resource: "org.tutorial.mercancias.Mercancia"
  action:ALLOW
}

rule ComercianteSystem {
    description: "Reglas para dar acceso al sistema al comerciante"
    participant: "org.tutorial.mercancias.Comerciante"
    operation: READ
    resource: "org.hyperledger.composer.system.**"
    action: ALLOW
}
```
A continuación explicaremos las reglas una a una:

### 2.1 Regla NetworkAdminUser
Esta regla le otorga al usuario administrador, `org.hyperledger.composer.system.NetworkAdmin`, acceso a la totalidad de recursos definidos dentro de nuestra red, sean estos activos, transacciones, eventos o participantes. Esto lo vemos pues se le otorga el acceso __ALL__ al recurso `**` que representa la totalidad de recursos definidos por el usuario

### 2.2 Regla NetworkAdminSystem
Esta Regla le otorga al usuario administrador, `org.hyperledger.composer.system.NetworkAdmin`, acceso a la totalidad de recursos de sistema. Esto lo vemos pues se le otorga el acceso __ALL__ al recurso `org.hyperledger.composer.system.**` que representa todos los recursos de sistema (distinto de los recursos definidos por el usuario)

### 2.3 Regla ComercianteVerComerciantes
Esta Regla le otorga a un usuario de tipo comerciante, `org.tutorial.mercancias.Comerciante`, acceso de lectura para poder ver a otros comerciantes. Esto lo vemos pues se le otorga el acceso __READ__ al recurso `org.tutorial.mercancias.Comerciante`.

### 2.4 Regla ComercianteVeMercancias
Esta Regla le otorga a un usuario de tipo comerciante, `org.tutorial.mercancias.Comerciante`, acceso de lectura para poder ver las mercancias registrada. Esto lo vemos pues se le otorga el acceso __READ__ al recurso `org.tutorial.mercancias.Mercancia`.

### 2.5 Regla ComercianteSystem
Finalmente, esta regla le otorga a un usuario de tipo comerciante, `org.tutorial.mercancias.Comerciante` acceso de lectura a los recursos de sistema. Esto lo vemos pues se le otorga el acceso __READ__ a el recurso `org.hyperledger.composer.system.**`.

## 3. Ajustando las Reglas para una Red de Negocio
Las reglas que tenemos definidas de momento, resultan muy simples y no permiten un verdadero control de operaciones comerciales. Para poder reflejar un escenario mas realista, debemos ajustar las reglas. Un primer reto, consistirá en ajustar dichas reglas para que un comerciante solo pueda ver las mercancias que son de su propiedad y un segundo reto consistirá en hacer que solo un comerciante pueda realizar una operación de venta de una mercancía si y solo si esa mercancía es de su propiedad.

### 3.1 Ajustando la Visibilidad de las Mercancias
Para poder limitar el acceso de visibilidad de una mercancía a su dueño, debemos modificar la regla `ComercianteVeMercancias` para que ahora quede como se muestra a continuación:

```
rule ComercianteVeMercancias {
  description: "Regla que permite que un comerciante ver únicamente las mercancias de su propiedad"
  participant(p): "org.tutorial.mercancias.Comerciante"
  operation: READ
  resource(r): "org.tutorial.mercancias.Mercancia"
  condition: (p.getIdentifier() == r.dueño.getIdentifier())
  action:ALLOW
}
```
Como podemos ver, hemos definido una variable (__p__) para representar al comerciante y una variable (__r__) para representar a la mercancía. Mediante la condición `(p.getIdentifier() == r.dueño.getIdentifier())` establecemos que en caso de que el identificador del participante sea igual que el identificador del participante registrado como dueño de la mercancía, entonces aplicaremos la acción __ALLOW__ sobre la operación __READ__ lo que permitirá entonces que el participante pueda leer una mercancia de su propiedad.

### 3.2 Ajustando los Permisos para la Venta
El caso de la siguiente regla es un poco mas compleja pues queremos limitar la venta de una mercancía a el comerciante propietario. Como podemos recordar, durante el tutorial hemos creado una transacción llamada `org.tutorial.mercancias.Opera` que realiza dicha transacción y que a su vez genera un evento llamado `org.tutorial.mercancias.EventoOperacion`. Ahora crearemos las reglas que aplicarán para poder realizar todo esto.

#### 3.2.1 Otorgamos el Permiso para Invocar la Transacción
El primer paso consiste en otorgar al participante el permiso de invocar o crear la transacción Opera
```
rule ComerciantePuedeInvocarOpera {
    description: "Regla que habilita a un comerciante a invocar la transacción Opera"
    participant: "org.tutorial.mercancias.Comerciante"
    operation: CREATE
    resource: "org.tutorial.mercancias.Opera"
    action: ALLOW
}
```

#### 3.2.2 Otorgamos el Permiso para Crear el Evento
El segundo paso, consiste en otorgar al participante el permiso de crear el evento asociado a la transacción
```
rule ComerciantePuedeCrearEventoOperacion {
    description: "Regla que habilita a un comerciante a crear el evento de operacion"
    participant: "org.tutorial.mercancias.Comerciante"
    operation: CREATE
    resource: "org.tutorial.mercancias.EventoOperacion"
    action: ALLOW
}
```
#### 3.2.3 Otorgamos el Permiso Condicional para Modificar una Mercancia
El tercer y mas importante paso, consiste en otorgar al participante el permiso de modificar una mercancía pero esto será bajo dos condiciones. La primera condición consiste en que dicha modificación será realizada únicamente mediante la ejecución de la transacción Opera y la segunda condición consiste en que esto sólo podrá ser hecho cuando la mercancía le pertenezca

```
rule ReglaCondicionalDeOperación {
    description: "Esta regla condiciona la modificación de una mercancia a la transacción opera y únicamente aplica cuando el comerciante es el dueñp"
    participant(p): "org.tutorial.mercancias.Comerciante"
    operation: ALL
    resource(r): "org.tutorial.mercancias.Mercancia"
    transaction(tx): "org.tutorial.mercancias.Opera"
    condition: (p.getIdentifier() == r.dueño.getIdentifier())
    action: ALLOW
}
```
#### 3.2.4 Otorgamos el Permiso para Crear un Registro Historian
Finalmente, requerimos un cuarto permiso que no hemos estudiado a fondo. Este permiso le permite a el participante a crear un registro en el Historian. El historian es la bitácora de todas las transacciones registradas en el blockchain y funciona como el registro de auditoria de todas las operaciones.

```
rule ComerciantePuedeCrearHistorian {
    description: "Un comerciante puede crear un registro en el historian"
    participant: "org.tutorial.mercancias.Comerciante"
    operation: CREATE
    resource: "org.hyperledger.composer.system.HistorianRecord"
    action: ALLOW
}
```

[__Regresar al Inicio__](README.md)