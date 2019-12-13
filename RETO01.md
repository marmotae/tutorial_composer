# Primer Reto
## Introducción
En este primer reto, se busca que el alumno pueda modificar el modelo creado originalmente para poder agregar un nuevo tipo de participante, "El Mercado" y definir las reglas necesarias para que sólo dicho participante pueda crear mercancías. El modelo hasta este punto se puede consultar (aquí)[https://github.com/marmotae/tutorial_composer/modelo/tutorial.bna]

## 1. Modificando el Modelo
El modelo se modifica para agregar ahora el participante tipo Mercado de la siguiente forma:

```
participant Mercado identified by mercadoId {
 o String mercadoId
 o String nombre
}
```
## 2. Modificando los Permisos
Para generar los permisos adecuados, el participante Mercado debe tener acceso de __CREATE__ sobre el activo de mercancias, tener adicionalmente permiso para agregar activos y tener permiso para modificar el historian. Adicionalmente, debemos asegurarnos que el administrador no pueda agregar mercancias
```
/**
 * Reglas para los mercados
 */


rule MecadoCreaMercancia {
  	description: "Reglas para que el mercado pueda crear una mercancia"
    participant: "org.tutorial.mercancias.Mercado"
    operation: CREATE
    resource: "org.tutorial.mercancias.Mercancia"
    action: ALLOW
}

rule MercadoSystem {
    description: "Reglas para dar acceso al sistema al mercado"
    participant: "org.tutorial.mercancias.Mercado"
    operation: READ
    resource: "org.hyperledger.composer.system.**"
    action: ALLOW
}

rule MercadoHistorian {
    description: "Reglas para dar acceso al sistema al mercado"
    participant: "org.tutorial.mercancias.Mercado"
    operation: CREATE
    resource: "org.hyperledger.composer.system.HistorianRecord"
    action: ALLOW
}

rule MercadoAddAsset {
  	description: "Regla que permite al mercado agregar un activo"
  	participant: "org.tutorial.mercancias.Mercado"
    operation: CREATE
  	resource: "org.hyperledger.composer.system.AddAsset"
  	action: ALLOW
}
```

Adicionalmente entre las reglas para el administrador, debemos agregar la siguiente regla:

```
rule AdminCreaMercanciaDeny {
  	description: "Reglas para que el administrador no pueda crear una mercancia"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: CREATE
    resource: "org.tutorial.mercancias.Mercancia"
    action: DENY
}
```

__NOTA__ La reglas son sensibles al orden de ejecución y si por ejemplo primero otorgamos acceso y luego lo denegamos, aplicará la regla que lo otorga por haber sido la primera. Por lo tanto conviene que la regla del administrador sera la primera.

## 3. Resultado Final

A continuación les pongo mi resultado final

### 3.1 Modelo
```
/*
 * Modelo de red inicial para el tutorial de Hyperledger
 * Composer
 */

namespace org.tutorial.mercancias

participant Comerciante identified by comercianteId {
  o String comercianteId
  o String nombre
  o String apellido
}

participant Mercado identified by mercadoId {
 o String mercadoId
 o String nombre
}

asset Mercancia identified by mercanciaId {
  o String mercanciaId
  o String descripcion
  o Double cantidad
  --> Comerciante dueño
}

transaction Opera {
  --> Mercancia mercancia
  --> Comerciante nuevoDueño
}

event EventoOperacion {
  --> Mercancia mercancia
  --> Comerciante viejoDueño
}
```
### 3.2 Reglas
```
/*
 * Archivo con las reglas de acceso para el tutorial 
 * de blockchain
 */

/**
 * Reglas para el Administrador
 */

rule AdminCreaMercanciaDeny {
  	description: "Reglas para que el administrador no pueda crear una mercancia"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: CREATE
    resource: "org.tutorial.mercancias.Mercancia"
    action: DENY
}

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
 * Reglas para los mercados
 */


rule MecadoCreaMercancia {
  	description: "Reglas para que el mercado pueda crear una mercancia"
    participant: "org.tutorial.mercancias.Mercado"
    operation: CREATE
    resource: "org.tutorial.mercancias.Mercancia"
    action: ALLOW
}

rule MercadoSystem {
    description: "Reglas para dar acceso al sistema al mercado"
    participant: "org.tutorial.mercancias.Mercado"
    operation: READ
    resource: "org.hyperledger.composer.system.**"
    action: ALLOW
}

rule MercadoHistorian {
    description: "Reglas para dar acceso al sistema al mercado"
    participant: "org.tutorial.mercancias.Mercado"
    operation: CREATE
    resource: "org.hyperledger.composer.system.HistorianRecord"
    action: ALLOW
}

rule MercadoAddAsset {
  	description: "Regla que permite al mercado agregar un activo"
  	participant: "org.tutorial.mercancias.Mercado"
    operation: CREATE
  	resource: "org.hyperledger.composer.system.AddAsset"
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
  description: "Regla que permite que un comerciante ver únicamente las mercancias de su propiedad"
  participant(p): "org.tutorial.mercancias.Comerciante"
  operation: READ
  resource(r): "org.tutorial.mercancias.Mercancia"
  condition: (p.getIdentifier() == r.dueño.getIdentifier())
  action:ALLOW
}

rule ComerciantePuedeInvocarOpera {
    description: "Regla que habilita a un comerciante a invocar la transacción Opera"
    participant: "org.tutorial.mercancias.Comerciante"
    operation: CREATE
    resource: "org.tutorial.mercancias.Opera"
    action: ALLOW
}

rule ComerciantePuedeCrearEventoOperacion {
    description: "Regla que habilita a un comerciante a crear el evento de operacion"
    participant: "org.tutorial.mercancias.Comerciante"
    operation: CREATE
    resource: "org.tutorial.mercancias.EventoOperacion"
    action: ALLOW
}

rule ReglaCondicionalDeOperación {
    description: "Esta regla condiciona la modificación de una mercancia a la transacción opera y únicamente aplica cuando el comerciante es el dueñp"
    participant(p): "org.tutorial.mercancias.Comerciante"
    operation: ALL
    resource(r): "org.tutorial.mercancias.Mercancia"
    transaction(tx): "org.tutorial.mercancias.Opera"
    condition: (p.getIdentifier() == r.dueño.getIdentifier())
    action: ALLOW
}

rule ComerciantePuedeCrearHistorian {
    description: "Un comerciante puede crear un registro en el historian"
    participant: "org.tutorial.mercancias.Comerciante"
    operation: CREATE
    resource: "org.hyperledger.composer.system.HistorianRecord"
    action: ALLOW
}

rule ComercianteSystem {
    description: "Reglas para dar acceso al sistema al comerciante"
    participant: "org.tutorial.mercancias.Comerciante"
    operation: READ
    resource: "org.hyperledger.composer.system.**"
    action: ALLOW
}
```
