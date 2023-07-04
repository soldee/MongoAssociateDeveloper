# REPLICATION

A replicaset in MongoDB is a group of mongod processes that mantain the same dataset.
- Replicasets provide:
    - **high availability** (a system that can operate without downtime).
    - **Increased read capacity** (only in some cases) as clients can send read operations to different servers

- A replicaset contains:
    - Primary node 
        - Receives all write operations
    - Secondary nodes
        - Replicate operations from the primary to mantain an identical dataset


<br><br>


## Options when creating a node

#### **Arbiter node** 
- In some situtations (such as you have a primary and a secondary but cost constraints prohibit adding another secondary) you may choose you include an arbiter that participates in elections
- Does not hold any data

#### **Delayed**
- Delayed members of a replicaSet reflect an earlier state of the dataset.
- Help recover from human errors
- Must be priority 0 and hidden
    
#### **Hidden**
- Mantains a copy of the dataset but is invisible to client applications. 
- Must be priority 0.
    
#### **Priority**
- Number between 0 and 1000 that indicates the relative eligibility of a member to become primary.
- A priority 0 member cannot become primary and cannot trigger elections.
    - Might be desired if the particular member is deployed in a data center that is distant from the main deployment and therefore has higher latency.

#### **Votes**
- El nombre de vots que té un servidor. Pot ser 0 o 1.
- Members with priority higher than 0 cannot have 0 votes.


<br><br>


## Com es repliquen les dades als secondaris ?
- **Statement-based replication** 
- Es guarden les operacions al oplog del primari. Els secondaris repliquen l’oplog del primari i apliquen les operacions.
- The write commands undergo a small transformation before they are stored in the oplog. The goal of this transformation is that the statement stored in the oplog can be applied multiple times while still resulting in the same data state. This is called **idempotence**.
    - Ex: 
        - Tenim un statement que incrementa un valor de la manera: {$inc: {“page_views”: 1} }
        - El que es guardarà al oplog és un statement que faci un set (mitjançant el _id) de page_views al valor que sigui enlloc de guardar un statement que incrementi page_views en 1.
        - Així aconseguim que es puguin aplicar els statements múltiples vegades sense modificar l’estat de les dades.
        - També s’ha de saber que si per exemple fem un updateMany, es guardarà al oplog cadascuna de les operacions fetes sobre cadascun dels documents afectats. Així, un updateMany pot generar moltes operacions a l’oplog.
- El oplog és una capped collection, és a dir, una colecció amb un tamany màxim; quan s’arriba al tamany màxim es comencen a produir overwrites.


<br><br>


## Initiating a ReplicaSet
- Ex: setup per iniciar 3 mongod com a un replicaSet
    - Primer creem un fitxer de configuració de mongod
    ```shell
    storage:
    dbPath: /var/mongodb/db/node1
    net:
    bindIp: 192.168.103.100, localhost
    port: 27011
    security:
    authorization: enabled
    keyFile: /var/mongodb/pki/m103-keyfile
    systemLog:
    destination: file
    path: /var/mongodb/db/node1/mongod.log
    logAppend: true
    processManagement:
    fork: true
    ```

    - Amb això podem crear les 3 instàncies de mongod en una mateixa màquina, només s’ha de canviar el path del systemLog, el storage i el port
    - El keyFile es pot generar fent:
    ```shell
    openssl rand –base64 741 > /var/mongodb/pki/m103-keyfile
    chmod 600 /var/mongodb/pki/m103-keyfile 
    ```

    - Iniciem els 3 mongod: mongod –f node1.conf (per a cadascun dels nodes amb el seu respectiu fitxer de configuració)
    - Ara hem d’iniciar el replicaSet:
    ```js
    rs.initiate()
    rs.add("hostname:27012")

    // També podriem iniciar-lo fent:
    rs.initiate({
        _id: "myReplSet",
        version: 1,
        members: [
            { _id: 0, host : "mongodb0.example.net:27017" },
            { _id: 1, host : "mongodb1.example.net:27017" },
            { _id: 2, host : "mongodb2.example.net:27017" }
        ]
    })
    ```

    - Podem forçar unes eleccions de primari:
        - rs.stepDown()
    - Podem veure el status del replicaSet:
        - rs.status() o bé rs.master()


<br><br>


## Failovers and Elections
- Failovers 
    - Triggered when a Primary node is down.
    - Failovers trigger Elections

- ReplicaSets can trigger an Election in response to these events:
    - Adding a new member to a ReplicaSet
    - Initiating a ReplicaSet
    - Using methods like rs.stepDown() or rs.reconfig()
    - The secondary members loose connectivity to the Primary for more than the configured timeout (10 seconds by default)

- When a Primary becomes unavailable…
    - The ReplicaSet cannot process write operations until the election completes
    - Elections should not exceed 12 seconds (including time to mark the primary as unavailable and complete an election)
    - MongoDB drivers detect the loss of the Primary and automatically retry write operations a single time

### Elections
- To figure out which secondary will run for election we need to know the priorities and which nodes hold the latest copies of the data (optime)
- If all the nodes have the same priority, the ones that have the latest copy of the data will run for election and the other nodes will break the tie and elect a Primary. Si es produeix un empat, es repeteixen les eleccions; així, és recomanable tenir un número imparell de nodes.
- Priority determines the eligibility of a ReplicaSet to become primary. Members with higher priority are more likely to become Primary.
    - Priority 0 nodes cannot become Primary
- Curiositat: If the primary can’t reach a majority of the other nodes in the set, then it will automatically step down to become a secondary. Si això passa en un ReplicaSet, tindriem dos nodes caiguts i un node Primari que es converteix automàticaent en Secondari; en aquest cas no tindriem writes fins que s’aixequés un altre node.

### Rollback
    - A rollback reverts write operations on a former primary when the member rejoins after a failover. 
    - A rollback is only necessary if the primary had accepted write operations that the secondaries had not successfully replicated before the primary stepped down.
    - A rollback does not occur if the write operations replicate to another member of the replicaSet before the primary steps down and if that member remains available and accessible to a majority of the replicaset.
    - To prevent rollbacks of data that have been acknowledged to the client, run all volting members with journaling enabled and use {w: “majority”}
    - Regardless of the write concern, other clients using “local” or “available” read concern can see the result of a write operation before it is acknowledged to the issuing client.
    - Rollback data is written to BSON files. To read its contents we can use bsondump. If the operation to roll back is a collection drop or a document deletion, the rollback is not written to the rollback data directory.


<br><br>


## Useful commands
- *rs.status()*
    - Information about replicaSet
        - health: 1 (OK)
        - state: 1 (PRIMARY), 2 (SECONDARY)
        - lastHeartbeat
        - optime: Date object. per saber si els nodes tenen les mateixes dades que el Primari.
- *rs.add()*
    - Ex: rs.add(“m103.mongodb.university:27017”)
- *rs.addArb()*
    - Add arbiter node
- *rs.remove()*
- *rs.conf()*
    - Retorna el fitxer de configuració actual del replicaSet
- *rs.reconfig()*
    - Ex: modifiquem un node d’un replicaSet de 4 nodes perquè no pugui votar i sigui hidden 
    - cfg = rs.conf()
    - cfg.members[3].votes = 0
    - cfg.members[3].hidden = true
    - cfg.members[3].priority = 0
    - rs.reconfig(cfg)


<br><br>


### Read Preference	
- Modes:
    - *primary* (default): read from primary
    - *primaryPreferred*: read from primary, if it is unavailable, read from secondaries
    - *secondary*: read from secondaries
    - *secondaryPreferred*: read from secondaries, if no secondaris are available, read from primary on sharded clusters
    - *nearest*: read from a node based on a specified latency threshold
- If we don’t use primary read preference we can receive stale data (dades antigues) depending on the latency of replication.


<br><br>


### Write Concern
- Level of acknowledgement requested from MongoDB for write operations
- { w: <value>, j: <boolean>, wtimeout: <number> }
    - j: acknowledgement that the write has been written to the journal
        - Write concern majority implies j: true. 
        - If set to false, the operations are only stored in memory before reporting success (no és una garantia de que s’hagi fet la operació) 
- Generally, setting a w: majority (default) provides a middle ground between fast writes and data durability guarantees
    - Amb un w: <max numero de nodes> trobariem que els writes poden arribar a trigar més, sobretot quan incrementen el nombre de nodes. A més, si falla un node ens arrisquem a que es bloqueji la operació fins que s’arribi al wtimeout.
    - Amb un w:1, es podria donar la situació en que el primari guardi el document i retorni el ACK al client però caigui abans de notificar el write als secondaris. Això faria que el client es pensés que s’ha guardat el document (només s’ha guardat al primari i està caigut). A més, crearia un Rollback quan el primari torni a estar available. 
- writeConcernMajorityJournalDefault 
    - Determines de behaviour of {w: “majority”} write concern if the j:<boolean> option is not explicitly set.
