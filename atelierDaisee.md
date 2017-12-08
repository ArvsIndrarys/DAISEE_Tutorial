Cette page va décrire la mise en place d'une blockchain locale utilisant le client Parity ainsi qu'une application de DAISEE sur un Raspberry Pi 3. 
 
## Configuration de Parity  
Pour des raisons de test ainsi que de ressources, le consensus utilisé au sein de la blockchain locale sera un [Proof-of-Authority](https://github.com/paritytech/parity/wiki/Proof-of-Authority-Chains).  
> \[Consensus\] : le consensus correspond à comment les noeuds de la blockchain vont autoriser les transactions ainsi que se partager le ledger.  
> \[Proof-of-Authority\]  : Contrairement au minage du Bitcoin qui consomme énormément de ressources en faisant résoudre par les noeuds des problèmes mathématiques ayant une difficulté arbitraire, PoA (Proof Of Authority) sélectionne parmi les noeuds participants des validateurs. Ces derniers se mettront d'accord pour accepter les transactions, empêcher les fraudes et sécuriser la blockchain. Pour une chaine privée dont on est sûr de la confiance que l'on peut apporter à ces noeuds, ce mécanisme est le plus simple à utiliser et à vérifier.  
 
La configuration de Parity est très simple car elle ne nécessite que deux fichers de configuration :   
- L'un pour configurer le comportement local du noeud, nommé **config.toml**  
- L'autre pour configurer la chaine à laquelle le noeud se connecte, nommé **demo-spec.json**  
  
> **Le fichier json configurant la chaine, tous les noeuds devront avoir à chaque instant exactement le même pour fonctionner et intéragir correctement.**  
   
### Initialisation des fichiers de configuration  

Tous les fichiers seront créés dans le répertoire `/home/pi/DAISEE`. Pour ce faire, connectez vous en ssh sur le raspberry qui vous est associé via Putty.  
La connexion effectuée, effectuez la commande `cd DAISEE`. Ceci fait, tapez `nano demo-spec.json` puis collez le code suivant :  
```
{      
    "name": "DemoPoA",
    "engine": {
        "authorityRound": {
            "params": {
                "gasLimitBoundDivisor": "0x400",
                "stepDuration": "5",
                "validators" : {
                    "list": []
                }
            }
        }
    },
    "params": {
        "maximumExtraDataSize": "0x20",
        "minGasLimit": "0x1388",
        "networkID" : "0x2323"
    },
    "genesis": {
        "seal": {
            "authorityRound": {
                "step": "0x0",
                "signature": "0x0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
            }
        },
        "difficulty": "0x20000",
        "gasLimit": "0x5B8D80"
    },
    "accounts": {
        "0x0000000000000000000000000000000000000001": { "balance": "1", "builtin": { "name": "ecrecover", "pricing": { "linear": { "base": 3000, "word": 0 } } } },
        "0x0000000000000000000000000000000000000002": { "balance": "1", "builtin": { "name": "sha256", "pricing": { "linear": { "base": 60, "word": 12 } } } },
        "0x0000000000000000000000000000000000000003": { "balance": "1", "builtin": { "name": "ripemd160", "pricing": { "linear": { "base": 600, "word": 120 } } } },
        "0x0000000000000000000000000000000000000004": { "balance": "1", "builtin": { "name": "identity", "pricing": { "linear": { "base": 15, "word": 3 } } } }
    }
}
```
*Pour coller du texte dans un terminal, tapez shift+ctrl+v au lieu de ctrl+v*  
  
Ensuite, enregistrez le fichier avec ctrl+o puis 'Enter', puis quittez nano avec ctrl+x.  
  
Editons alors **config.toml** de la même manière en exécutant `nano config.toml` puis en collant le code suivant :  
```
[parity]
chain = "demo-spec.json"
base_path = "/home/pi/DAISEE/parity"
[network]
port = 30300
[rpc]
port = 8540
apis = ["web3", "eth", "net", "personal", "parity", "parity_set", "traces", "rpc", "parity_accounts"]
[ui]
port = 8180
interface = "0.0.0.0"
[websockets]
port = 8450
[ipc]
disable = true
```
Une fois ceci fait, Parity est (presque) entièrement configuré !  

### Lancement de Parity  

Pour lancer Parity, exécutons la commande `parity -c config.toml --unsafe-expose`. Nous devons obtenir le résultat suivant :  
 
![noderunning](https://framapic.org/kGrNa8b41vVn/io7pi2l3DDMY)
 
Laissez le terminal tel quel puis ouvrez votre navigateur à l'adresse http://**<ip_raspberry>**:8180. Pour trouver l'adresse de votre raspberry, il est indiqué à la fin de la ligne *Public node URL* dans le terminal (192.168.1.xx).
![](https://framapic.org/Fmlke1ALWYh3/aN9S5i0XaLRW)  

A partir d'ici il y a trois cas de figures :
- soit une fenêtre présentant Parity s'est affichée, auquel cas vous serez guidé jusqu'à la création de votre premier compte.
- soit vous êtes sur l'UI sans avoir créé votre premier compte, ce que vous allez faire en allant dans l'onglet 'Accounts' et en cliquant sur "ACCOUNT", "New account".
- soit une fenêtre vous demandant d'entrer un token s'est affiché. Dans ce dernier cas, vous allez établir une nouvelle connexion SSH sur le raspberry puis `cd DAISEE` et `parity signer new-token -c config.toml`. Cette dernière commande affichant sur la dernière ligne le token que vous allez copier (shift+ctrl+v dans un terminal) puis coller dans le champ demandant le token.

> *Pour la création du compte, n'oubliez pas de mettre un mot de passe simple ainsi que de garder la phrase de récupération, elle est demandée par la suite.*
 
### Connecter les noeuds 

Le but de cet étape est de connecter les noeuds ensemble. Pour ce faire, nous allons éditer le fichier de configuration **config.toml**.  
Mais auparavant, il s'agit de récupérer l'enode de votre noeud, ce que l'on obtient soit en ré cupérant la ligne *Public node URL* à partir de 'enode' jusqu'à '30303' compris dans le terminal, à l'endroit qui vous a permis de récupérer l'adresse IP du raspberry, soit dans l'UI en allant sur l'onglet représenté par les barres de réseau dans le champ enode (il y aura peut être besoin de rafraîchir la page pour qu'elle s'affiche correctement avec la touche f5).
![](https://framapic.org/bgZb0PSYhs7m/VkpAgUf4psdO)  
*l'enode est en bas à droite*  

Vous devez donc entrer les enodes des autres participants dans votre fichier **config.toml**. Pour ce faire, ajouter dans la partie **[network]** :
```
pour ajouter 1 noeud :
bootnodes = ["enode://ff14ae0a273e08ffbbe20b4b398460eb471e23f1b4301ce46b92a86ad420f67b9b470d097f1939fa7b9b2aae7d24e72cf7c63fe67217bdf3fd6cb60bbb7ecc59@192.168.0.47:30300"]  

pour ajouter plusieurs noeuds :  
bootnodes = ["enode://ff14ae0a273e08ffbbe20b4b398460eb471e23f1b4301ce46b92a86ad420f67b9b470d097f1939fa7b9b2aae7d24e72cf7c63fe67217bdf3fd6cb60bbb7ecc59@192.168.0.47:30300","enode://ff14ae0a273e08ffbbe20b4b398460eb471e23f1b4301ce46b92a86ad420f67b9b470d097f1939fa7b9b2aae7d24e72cf7c63fe67217bdf3fd6cb60bbb7ecc59@192.168.0.47:30300"]
```
*remplacer les enodes affichés par ceux des autres participants et n'ajoutez pas les lignes "pour ajouter..."*

Une fois ceci fait, stoppez Parity en tapant ctrl+c dans le terminal où vous l'avez lancé, puis relancez-le `parity -c config.toml --unsafe-expose`.  
Si les ajouts ont bien été effectués, vous devez voir dans le terminal le nombre de noeuds connectés augmenter, ou dans l'UI les barres de réseau passer au jaune puis au vert. De plus, dans l'onglet réseau, vous aurez le message `Connected Peers (x/25)`
![nodeconnected](https://framapic.org/Gp6UgPgiP2sF/P8NyaK6bbTRH)
 
### Ajout des validateurs  
Pour que cette implémentation en Proof Of Authority fonctionne, nous allons définir quels sont les noeuds qui vont valider les transactions.  
Ces validateurs correspondent à des comptes de la blockchain.  
  
Une fois que les validateurs ont été choisis, tous les participants vont remplacer dans **demo-spec.json** les lignes suivantes dans la partie `validators` :
```
"list": [
    "0x005d23c129e6866B89E1C73FC3b05014255CEFA2",
    "0x0011067b3a4fE6fd301296AD5bC730F7a1CeCE4f"
]
```
*Remplacer les adresses écrites ici par ceux des validateurs désignés. Ces derniers peuvent accéder à l'adresse de leur compte dans l'UI, onglet **Accounts**, noté juste en dessous du nom de leur compte*
*A la fin de chaque addresse, rajoutez une virgule, sauf pour la dernière, sinon Parity reportera un problème dans le fichier*

De plus pour les validateurs, quelques étapes en plus sont à effectuer. Il faut d'une part rajouter dans le **config.toml** que le compte a été désigné comme validateurs avec les lignes suivantes :
```
[account]
password = ["node.pwds"]
[mining]
engine_signer = "0x002e28950558fbede1a9675cb113f0bd20912019"
reseal_on_txs = "none"
```
*remplacer l'adresse écrite par celle de votre compte*  

Et créer le fichier node.pwds ( `nano node.pwds` ) où vous n'écrirez que le mot de passe du compte.
 
### Alimenter les comptes en Ether fictifs  

Pour effectuer les actions suivantes, les comptes doivent être alimentés en Ether, le "carburant" de la blockchain.
> \[Ether\] : l'Ether, comme les autres cryptomonnaies, a pour but de permettre le fonctionnement de la blockchain. Sans Ether, une blockchain Ethereum ne peut pas fonctionner, car pour pouvoir effectuer une action au sein de la blockchain, il faut payer le droit de faire cette action en Ether. Ceci permet de responsabiliser les participants et empêche les attaques de type "déni de service".  

Pour ce faire, dans la partie **accounts** du fichier **demo-spec.json**, chaque participant doit rajouter son compte et celui des autres de la manière suivante : 
```
"0x005d23c129e6866B89E1C73FC3b05014255CEFA2": { "balance": "100000000000000000000" }
```
*à la fin de chaque ligne sauf la dernière, une virgule devra être rajoutée, sans quoi parity indiquera une erreur dans le fichier*  

Ces modifications ajoutées, nous pouvons relancer Parity `parity -c config.toml --unsafe-expose` et dans l'UI sous l'onglet **Accounts**, nous observerons que le compte est alimenté en Ether.
 
Si c'est le cas, ajoutez dans votre carnet d'adresses les comptes des autres participants : allez dans l'onglet **Addressbook**, cliquez sur 'Address' et coller l'adresse ainsi que le nom du compte que vous voulez ajouter.  
![](https://framapic.org/UYgClcmnF97S/bungWbnezu8i) 
*L'adresse d'un compte est affichée sous son nom dans l'onglet **Accounts** .*
 
## Smart-contracts

Pour l'application de DAISEE que nous allons mettre en oeuvre, nous aurons besoin d'un fichier présent dans le dossier `/home/pi/DAISEE`, nommé **daisee.sol**.   
Ce dernier est composée de deux parties, la première permettant de créer un token, le DaiseeCoin, qui permettra de financer les transactions, et la deuxième partie correspond à l'échange d'énergie entre les pairs et le stockage de données dans la blockchain.  


### Déploiement des contrats  

Un seul des participants va déployer ces contrats car de par le fonctionnement de la blockchain, ils seront décentralisés et tous pourront y accéder.

Allez dans l'onglet **Contracts**, cliquez sur "DEVELOP" et collez le code de **token.sol**.
![](https://framapic.org/bd3PlL7voo1h/Y0rstT9XQlNr)  
  
Selectionnez dans 'Select a Solidity version' la version 0.4.2 et cliquez sur "COMPILE". Choisissez ensuite dans 'Select a contract' le contrat MyAdvancedToken et cliquez sur "DEPLOY".
![](https://framapic.org/j2Q5T3lleA8c/6UVWBTQgAE8v)  
  
Entrez les infos du contrat tels que présentés aux images suivantes puis confirmez avec le mot de passe du compte qui déploie le contrat.
![](https://framapic.org/xL3WJP6DyOC3/fmSPKmy0NYAX)  
  
![](https://framapic.org/3EFEwAHH5LLm/GNCVekcH76oc)  
  
![](https://framapic.org/sBVW01VIdool/tvvgpXF9qD1T)  
  
Ensuite selectionnez le contrat Daisee dans 'Select a contract' et cliquez de nouveau sur "DEPLOY"
![](https://framapic.org/wNAfTD8HJQgU/ux4Q4HRwCMsL)  

Appelez le simplement Daisee et confirmez la transaction. Les deux smart-contracts apparaissent maintenant dans l'onglet **Contracts**.
![](https://framapic.org/3kiQhcuf7ECw/zkhGxv6s8Xqt)  

### Partage des contrats  

Pour que tous les noeuds puissent utiliser ces contrats, ils doivent l'ajouter. Pour ce faire, nous allons récupérer deux infos sur chacun des contrats. La première correspond à l'adresse (0x-------------).
![](https://framapic.org/g85tjH3yB6Wq/bUWJaJJD66Uy)  

La deuxième correspond à l'ABI, que l'on peut trouver en cliquant sur le contrat puis sur "DETAILS" en haut à droite, et il sera affiché tout en bas.
![](https://framapic.org/ggeEzw2RetGL/ettadL1qIifq)
*Pour le copier facilement, cliquer sur son code puis ctrl+a (sélectionner tout le code) puis ctrl+c (copier)*  

Les autres noeuds vont alors pouvoir utiliser ces informations en cliquant dans **Contracts** puis "WATCH", 'Token' pour DaiseeCoin et 'Custom Contract' pour Daisee, et en collant les informations dans le bon champ.
![](https://framapic.org/2ZqtxW66tU0J/gLyV9E25nlKA)  

Une fois ceci fait, chacun des noeuds doit pouvoir voir les deux contrats dans l'onglet **Contracts**.
![](https://framapic.org/HLM8W77L0qGc/gG1ZV3zQrIfQ)
  

Note: the current version of DAISEE smart contract allows to update consumption and production directly. To do this, select the Daisee contract, click the "EXECUTE" button at the top right, and choose the function 'setProduction' or 'consumeEnergy':
![](https://framapic.org/r3IFUYGIKWYQ/pp71NzT78HIl)  
  
![](https://framapic.org/M2JzuIOl9qbP/8Na2nbGOd3Ds)

## Interface

To view data, a Web interface communicates with the local node, through [Web3 JavaScript API](https://github.com/ethereum/wiki/wiki/JavaScript-API), using [web3.js](https://github.com/ethereum/web3.js).  
The microframework [Flask](http://flask.pocoo.org/) allows to display the interface from the Raspberry Pi node.
> _Tutorial used_:  
➡ [A simple smart contract Web UI using web3.js](http://hypernephelist.com/2016/06/21/a-simple-smart-contract-ui-web3.html)  

Install the requirements
```bash
$ sudo aptitude install git
$ sudo pip3 install flask pyyaml requests
```

Clone the repository
```bash
$ git clone https://github.com/DAISEE/DApp-v2.git
```


In `DApp-v2/dapp`, create the configuration file (`config.yml`) from the example and complete it:


```
contracts:
  daisee: '0xbeaE6e2747bD6db798d222E2D2185c484b5f2f9d'
  token: '0x9cf61b2b43f5695D65e633d0CA2dC03908eB6dd1'
user:
  login: 'daisee'
  pwd: '4df74be9792adc7848b15d833748b3affe59fced7e5dd5623831fa3040424761'
  coinbase: '0x005d23c129e6866b89e1c73fc3b05014255cefa2'
  name: 'node1'
  typ: 'C'
  url: 'http://0.0.0.0'
  sensorId:
  sensorLogin: ''
  sensorPassword: ''
  sensorSource: ''
  sensorPort: ''
```

|Type|Field|Description|
| ------------- | ------------- | ------------- |
|**contracts**|||
| |daisee| Daisee.sol address on the blockchain|
| |token| DaiseeCoin smart-contract address on the blockchain|
|**user**|||
| |login | login for UI |
| |pwd | hashed password for the UI* |
| |coinbase | user address |
| |type | type of node ('C' for consumer, 'P' for producer). _Not Used_ |
| |url | url of energy monitoring application** |
| |sensorId | url of energy monitoring application |
| |sensorLogin | login for energy monitoring application |
| |sensorPassword | password for energy monitoring application |
| |sensorSource | energy monitoring application  : `'CW'` for citizenWatt (only app supported for now) |
| |sensorPort | if necessary, port of energy monitoring application, example : `':8080'` |
> still under development, it may change

_\* to obtain the hashed password, use 'raspberry-ip:5000/hash/\<password>' after running the server_  
_\** if the energy monitoring application is (or will be) on the same Raspberry Pi, follow these [instructions](https://github.com/DAISEE/Prototypes/wiki/3.-Energy-monitoring) before running DAISEE app._  
_\*** if sensors are not used, sensor parameters can be empty_  


Run the server
```bash
$ export FLASK_APP=server.py
$ python3 -m flask run --host=0.0.0.0
```

Go to http://_raspberry-ip_:5000 to access to the interface.

![](https://framapic.org/K9JXZbyw9yR4/QuA3uLk6DDNv)
> The current version displays Ethereum transactions and realtime data from the energy monitoring application

