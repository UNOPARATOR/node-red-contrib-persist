# node-red-contrib-persist

`node-red-contrib-persist`aims to ***persist data*** over Node-RED ***restarts*** and ***deploys***.

This node set consists of 
- a persistent store ***configuration node*** which buffers the messages and writes them to the filesystem at regular intervals,
- an ***input node*** to record messages and
- an ***output node*** which replays the last message saved when Node-RED restarts or is deployed.


![node-appearance](assets/node-appearance.png "Node appearance")  
**Fig. 1:** Node appearance

**Basic operation**  
A message stored by the input node is saved under the name of that node.  Only the last message received is saved, irrespective of its topic.   The message is replayed by the output node with the same name when it is triggered.  Output nodes are triggered on startup, after deploys, on receipt of any message, or when the button on the node is pressed in the Node-RED console.

<img src="assets/basic-structure.png" title="Basic structure" width="700" />

**Fig. 2:** Basic structure

Messages are stored in an internal buffer until flushed to the filesystem. The buffer is flushed not more frequently than the interval specified in the persistence store configuration node. Messages received within the flush interval are discarded, except for the last message received. If no message is received within the flush interval, the next message received will trigger a flush. Setting a large flush interval will reduce filesystem writes but will increase the risk of data loss. The buffer is flushed when Node-RED is shutdown.

This node set was written in particular to persist data used in the Dashboard graphs.  The graph nodes output their entire current data set for each new input received.  This output can be persisted and fed back to the graph node on startup or deploy.

<a name="installation"></a>
## Installation

<a name="installation_in_node-red"></a>
### In Node-RED (preferred)
* Via Manage Palette -> Search for "node-red-contrib-persist"

<a name="installation_in_a_shell"></a>
### In a shell
* go to the Node-RED installation folder, e.g.: `~/.node-red`
* run `npm install node-red-contrib-persist`

<a name="usage"></a>
## Usage

<a name="node_configuration"></a>
### Node Configuration
#### Persist configuration node

<img src="assets/node-settings.png" title="Node configuration of the persist configration node" width="600" />

**Fig. 3:** Node properties of the persist configuration node

Fig. 3 shows the configuration of the persist configuration node. The filename may be given with a path (e.g. ~/.node-red/persistence.json). It is important, that the file system is writable. Otherwise an error is thrown. This can be examined via the debug message panel.  
In the case that this file does not exist, is will be generated.

The contents of the file is a JSON object containing the persistence data. An example for the file contents is:
`{"myPersistence":{"_msgid":"3f99dd02.975182","topic":"","payload":"Rhett Nowed"}}`

#### Node configuraton of the `persist in` and `persist out` node

The configuration of the `persist in` and `persist out` nodes look like in the following figure. In *Store* the file location of the persistence storage file (i.e. the corresponding persist configuration node) has to be selected. 

<img src="assets/persistence-persist_in.png" title="Node configuration of the 'persist in' and 'persist out' node" width="280" />

**Fig. 4:** Node properties of the `persist in` and `persist out` node

Different options regarding persist nodes and persistence storage are possible. They are described in the following.

##### Single `persist in` node (with single persistence file)
This is the very basic and simplest case. 
The configuration entry *Name* denotes the JSON object name. The node property *Name* from Fig. 4 ("myPersistence") is identical to the part in the JSON object within the persistence file: 
`{"myPersistence":{"_msgid":"3f99dd02.975182","topic":"","payload":"Rhett Nowed"}}`

<img src="assets/persistence-storage1.png" title="Single 'persist in' node" width="400" />

**Fig. 5:** Single `persist in` node

##### Multiple `persist in` nodes with identical *Name* configuration with single persistence file
Multiple `persist in` nodes may used with the same persistence name are shown in Fig. 6. The behaviour is identical as if all four inject nodes would be linked to one single `persist in` node.

<img src="assets/persistence-storage2.png" title="Two 'persist in' nodes working on the same persistence configuration" width="400" />

**Fig. 6:** Two `persist in` nodes working on the same persistence *Name* configuration


##### Multiple `persist in` nodes with different *Name* configurations with single persistence file
This option is used if multiple persistence objects have to be stored (within one single file).

This is shown in Fig. 7: The first `persist in` node has the *Name* **myPersistence**, the second `persist in` node has the *Name* **myOtherPersistence**. 

<img src="assets/persistence-storage3.png" title="Two 'persist in' nodes working on different persistence configurations" width="400" />

**Fig. 7:** Two `persist in` nodes working on different persistence *Name* configurations

In this case, the contents of the file is a JSON object containing both persistence data. An example for the file contents is:
`{"myPersistence":{"_msgid":"4573396f.41eb48","topic":"","payload":"Nat Rowed"},    "myOtherPersistence":{"_msgid":"865e961d.ede2a8","topic":"","payload":"Rhett Nowed"}}`

As can be seen, the example has two sub-parts ("myPersistence", "myOtherPersistence") with their corresponding stored objects.


##### Multiple persistence files
Additionally, several persist stores (= storage files) may be used in Node-RED, each having an own configuration node and according `persist in` resp. `persist out` nodes. 

<a name="input"></a>
### Input
#### Node `persist in`
The contents of all `msg` objects given to the input of the `persist in` node are transfered to the persistence storage. The last message overwrites all previous message contents.  
Remark: The `msg` structure of several messages may vary.


#### Node `persist out`
Every input `msg` triggers a restore of the data of this persistence *Name* configuration.
Also, the button on the left manually triggers this procedure.


<a name="output"></a>
### Output
Only the node `persist out` contains an output. It contains the persisted data, i.e. the corresponding message which was transmitted to the corresponding `persist in` node input.

This output data has to be used to set other data: This is the basis of the persist functionality! Otherwise data is only stored and not replayed.


<a name="further_information"></a>
### Further information
Check Node-REDs info panel to see more information on how to use the easing node.

<a name="example"></a>
## Examples

### Persistence storage process example
The example flow shows the "persistence storing" part of the functionality: The last inject of the names is stored at a rate of one minute to the persistence file.

<img src="assets/flow.png" title="Example flow" width="400" />

The contents of the file may e.g. look like:
{"myPersistence":{"_msgid":"c493a2a3.73a8e","topic":"","payload":"Rat Nohde"}}

**Fig. 8:** Persistence storage process example

```json
[{"id":"f0172f83.431df","type":"persist in","z":"b51e984d.4279b","name":"myPersistence","storageNode":"42dd5cc5.694f1c","x":749,"y":240,"wires":[]},{"id":"3aa581f3.d377be","type":"inject","z":"b51e984d.4279b","name":"name1","topic":"","payload":"Rhett Nowed","payloadType":"str","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":490,"y":240,"wires":[["f0172f83.431df","2bba8123.3c3a16"]]},{"id":"7610b2fd.8d8eb4","type":"inject","z":"b51e984d.4279b","name":"name2","topic":"","payload":"Ned Rhode","payloadType":"str","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":490,"y":280,"wires":[["f0172f83.431df","2bba8123.3c3a16"]]},{"id":"2bba8123.3c3a16","type":"debug","z":"b51e984d.4279b","name":"","active":true,"tosidebar":true,"console":false,"tostatus":true,"complete":"payload","targetType":"msg","x":740,"y":300,"wires":[]},{"id":"7eec9ac1.45cddc","type":"inject","z":"b51e984d.4279b","name":"name3","topic":"","payload":"Rat Nohde","payloadType":"str","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":490,"y":320,"wires":[["f0172f83.431df","2bba8123.3c3a16"]]},{"id":"ddfefd8.eec7f8","type":"inject","z":"b51e984d.4279b","name":"name4","topic":"","payload":"Nat Rowed","payloadType":"str","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":490,"y":360,"wires":[["f0172f83.431df","2bba8123.3c3a16"]]},{"id":"42dd5cc5.694f1c","type":"persist-store","z":"","filename":"/home/Node-RED/persistance.json","interval":"60"}]
```  
**Fig. 9:** Persistence storage process example flow


### Persistence replay process example

xxx ToDo: Example

