# Crea tu primera red pública con Ethereum

>Tutorial para crear una red local con Ethereum en el marco del taller Construye tu primera Red de Blockchain organizada por Hyperledger Mérida y Blockchain Mérida.

*Autor: Greco Rubio*

Esta es una guía para configurar una red local de ethereum con *2 nodos selladores* (sealers) y *1 nodo local* con un *bootnode* para que los nodos se encuentren fácilmente.

Usaremos el protocolo de concenso **Proof-of-Authority (Clique)** en lugar de *Proof-of-Work (Ethash)* debido a que **PoA** requiere menos recursos computacionales y así tendremos mayor velocidad de aprobación de las transacciones.

### Resumen:

1. Requerimientos de software
2. Estructura del proyecto
3. Cuentas
4. Archivo génesis
5. Inicializar nodos
6. Crear bootnode
7. Iniciar bootnode
8. Iniciar nodos selladores (sealer)
9. Iniciar el nodo local
10. Probar tu red local de ethereum

## 1. Requerimientos de software

Descargar **Geth** que es la implementación oficial del protocolo de Ethereum escrita en Go.

Utilizaremos la **versión all tools 1.8.23**, puedes verificar la versión que tienes instalada ejecutando `geth version`

**Linux 64-bit:** https://gethstore.blob.core.windows.net/builds/geth-alltools-linux-amd64-1.8.23-c9427004.tar.gz

**OSX:** https://gethstore.blob.core.windows.net/builds/geth-alltools-darwin-amd64-1.8.23-c9427004.tar.gz

**Windows 64-bit:** https://gethstore.blob.core.windows.net/builds/geth-alltools-windows-amd64-1.8.23-c9427004.zip

Sitio oficial para descargar geth:

https://geth.ethereum.org/downloads/

Para extraer un archivo `tar.gz`:

```bash
tar xvfz ./archivo.tar.gz
```

Para acceder a los binarios de manera global:

```bash
cp geth/* /usr/local/bin
```

## 2. Estructura del proyecto

```
.
├── accounts.txt
├── boot.key
├── bootnode.txt
├── genesis.json
├── localnode
│   ├── geth
│   ├── geth.ipc
│   ├── keystore
│   └── password.txt
├── sealernode1
│   ├── geth
│   ├── geth.ipc
│   ├── keystore
│   └── password.txt
└── sealernode2
    ├── geth
    ├── geth.ipc
    ├── keystore
    └── password.txt
```

#### Crear carpetas iniciales

**Se requiere abrir una ventana de terminal para seguir adelante.**

Crear carpeta del proyecto:

```bash
mkdir devnet
```

Ir a la carpeta del proyecto:

```bash
cd devnet
```

Crear carpetas de los nodos:

```
mkdir localnode sealernode1 sealernode2
```

## 3. Cuentas

Se crearán 3 cuentas, 2 para los nodos selladores (sealers) y 1 para el nodo local.

### Cuenta para sealernode1

#### Crear password para sealernode1

```bash
nano sealernode1/password.txt
```
`ctrl+o` `enter` `ctrl+x`

```bash
chmod 700 sealernode1/password.txt
```

Copiar la llave pública y guardarla en en archivo `accounts.txt` de la siguiente forma:

```bash
echo "sealernode1: 0xfed44...4e37e" >> accounts.txt
```

#### Crear cuenta para sealernode1

```bash
geth --datadir sealernode1/ account new --password sealernode1/password.txt
# Address: {fed44...4e37e}
```

### Cuenta para sealernode2

#### Crear password para sealernode2

```bash
nano sealernode2/password.txt
```
`ctrl+o` `enter` `ctrl+x`

```bash
chmod 700 sealernode2/password.txt
```

#### Crear cuenta para sealernode2

```bash
geth --datadir sealernode2/ account new --password sealernode2/password.txt
# Address: {f3af8...a3d00}
```

Copiar la llave pública y guardarla en en archivo `accounts.txt` de la siguiente forma:

```bash
echo "sealernode2: 0xf3af8...a3d00" >> accounts.txt
```

### Cuenta  para localnode

#### Crear password para localnode

```bash
nano localnode/password.txt
```
`ctrl+o` `enter` `ctrl+x`

```bash
chmod 700 localnode/password.txt
```

#### Crear cuenta para localnode

```bash
geth --datadir localnode/ account new --password localnode/password.txt
# Address: {183c2...def12}
```

Copiar la llave pública y guardarla en en archivo `accounts.txt` de la siguiente forma:

```bash
echo "localnode: 0x183c2...def12" >> accounts.txt
```

### Verificar cuentas

```bash
cat accounts.txt
# sealernode1: 0xfed44...4e37e
# sealernode2: 0xf3af8...a3d00
# localnode: 0x183c2...def12
```

## 4. Archivo Génesis

Para crear el archivo `genesis.json` utilizaremos la herramienta **puppeth** porque facilita mucho este proceso:

```bash
puppeth
```

Con esto iniciamos la herramienta interactiva y seguimos los siguientes pasos:

```bash
Please specify a network name to administer (no spaces, hyphens or capital letters please)
> blockchainmerida
```

```bash
What would you like to do? (default = stats)
 1. Show network stats
 2. Configure new genesis
 3. Track new remote server
 4. Deploy network components
> 2

What would you like to do? (default = create)
 1. Create new genesis from scratch
 2. Import already existing genesis
> 1

Which consensus engine to use? (default = clique)
 1. Ethash - proof-of-work
 2. Clique - proof-of-authority
> 2

How many seconds should blocks take? (default = 15)
> 15

Which accounts are allowed to seal? (mandatory at least one)
> 0xfed44...4e37e
> 0xf3af8...a3d00
> 0x

Which accounts should be pre-funded? (advisable at least one)
> 0xfed44...4e37e
> 0xf3af8...a3d00
> 0x183c2...def12
> 0x

Should the precompile-addresses (0x1 .. 0xff) be pre-funded with 1 wei? (advisable yes)
> no

Specify your chain/network ID if you want an explicit one (default = random)
> 12345
```

```bash
What would you like to do? (default = stats)
 1. Show network stats
 2. Manage existing genesis
 3. Track new remote server
 4. Deploy network components
> 2

 1. Modify existing fork rules
 2. Export genesis configurations
 3. Remove genesis configuration
> 2

Which folder to save the genesis specs into? (default = current)
  Will create blockchainmerida.json, blockchainmerida-aleth.json, blockchainmerida-harmony.json, blockchainmerida-parity.json
>
```

`ctrl+c`

Eliminar archivo innecesario para los fines de este tutorial:

```bash
rm blockchainmerida-harmony.json
```

Renombrar archivo genesis recien creado a `genesis.json`:

```bash
mv blockchainmerida.json genesis.json
```

## 5. Inicializar nodos

sealernode1:

```bash
geth --datadir sealernode1 init genesis.json
```

sealernode2:

```bash
geth --datadir sealernode2 init genesis.json
```

localnode:

```bash
geth --datadir localnode init genesis.json
```

## 6. Crear bootnode

Crear llave para el bootnode:
```bash
bootnode -genkey boot.key
```

Obtener la dirección del bootnode:
```bash
bootnode -nodekey boot.key -writeaddress
# 977aa...c8419
```

Copiar la dirección del bootnode y guardarla en en archivo `bootnode.txt` de la siguiente forma:

```bash
echo 977aa...c8419 >> bootnode.txt
```

## 7. Iniciar bootnode

```bash
bootnode -nodekey boot.key -verbosity 9 -addr :30303
```

**Nota: El servicio se quedará corriendo. Se requiere abrir otra ventana de terminal para seguir adelante.**

En la nueva ventana, ir a la carpeta del proyecto:

```bash
cd devnet
```

## 8. Iniciar nodos selladores (sealer)

### Iniciar sealernode1

**Ojo: Sustituir** los valores de la cuenta del `sealernode1` y la dirección del `bootnode` con los que apuntamos recientemente en los archivos `accounts.txt` y `bootnode.txt` respectivamente:

```bash
geth --datadir sealernode1 --syncmode 'full' --port 30310 --networkid 12345 --gasprice '0' -unlock '0xfed44...4e37e' --password sealernode1/password.txt --bootnodes 'enode://977aa...c8419@127.0.0.1:30303' --mine
```

**Nota: El servicio se quedará corriendo. Se requiere abrir otra ventana de terminal para seguir adelante.**

En la nueva ventana, ir a la carpeta del proyecto:

```bash
cd devnet
```

### Iniciar sealernode2

**Ojo: Sustituir** los valores de la cuenta del `sealernode2` y la dirección del `bootnode` con los que apuntamos recientemente en los archivos `accounts.txt` y `bootnode.txt` respectivamente:

```bash
geth --datadir sealernode2 --syncmode 'full' --port 30311 --networkid 12345 --gasprice '0' -unlock '0xf3af8...a3d00' --password sealernode2/password.txt --bootnodes 'enode://977aa...c8419@127.0.0.1:30303' --mine
```

**Nota:** a pesar de la nomenclatura, el argumento `--mine` sirve para habilitar la función de sealer en el nodo debido a que estamos trabajando con PoA.

**Nota: El servicio se quedará corriendo. Se requiere abrir otra ventana de terminal para seguir adelante.**

En la nueva ventana, ir a la carpeta del proyecto:

```bash
cd devnet
```

## 9. Iniciar el nodo local

**Ojo: Sustituir** los valores de la cuenta del `localnode` y la dirección del `bootnode` con los que apuntamos recientemente en los archivos `accounts.txt` y `bootnode.txt` respectivamente:

```bash
geth --datadir localnode --syncmode 'full' --port 30312 --networkid 12345 --gasprice '0' -unlock '0x183c2...def12' --password localnode/password.txt --bootnodes 'enode://977aa...c8419@127.0.0.1:30303' --rpc --rpcaddr '127.0.0.1' --rpcport 8545 --rpccorsdomain '*'
```

Como se puede ver, en este caso configuramos el nodo local con acceso vía RPC para acceder vía clientes como Web3.js, Metamask, Remix IDE, etc.

**Nota: El servicio se quedará corriendo. Se requiere abrir otra ventana de terminal para seguir adelante.**

En la nueva ventana, ir a la carpeta del proyecto:

```bash
cd devnet
```

## 10. Probar tu red local de Ethereum

Para entrar al CLI (Command Line Interface) de nuestro nodo local:

```bash
geth attach ipc:./localnode/geth.ipc
```

Enlistar cuentas:

```javascript
eth.accounts
```

Crear una nueva cuenta:

```javascript
personal.newAccount()
```

Para ver el balance de una cuenta:

```javascript
web3.eth.getBalance(eth.accounts[1])
```

Para enviar ether de una cuenta a otra:

```javascript
eth.sendTransaction({from: eth.accounts[0], to: eth.accounts[1], value: web3.toWei(1, "ether")})
```

## Conclusiones

Con esto aprendimos a grandes rasgos cómo funciona el protocolo de Ethereum al crear una red local usando el protocolo **Proof-of-Authority (Clique)** y la herramienta **puppeth** para crear el archivo génesis de manera clara y sencilla.

Aquí tienes algunos retos a lograr si quieres poner en práctica lo aprendido en este tutorial e ir más allá para tener un mayor entendimiento de Ethereum y los smart contracts:

### Reto 1:
Crear una red de Ethereum ya sea con PoW o PoA entre nodos configurados en distintas computadoras e interactuar entre ellos.

### Reto 2:
Crear un smart contract en la red (ej. un ERC20) e interactuar con él mediante los nodos que se encuentren conectados.

## Referencias

https://ethereum.org

https://geth.ethereum.org/

https://remix.ethereum.org

https://web3js.readthedocs.io/en/1.0/

https://metamask.io/

https://github.com/grekinsky/my-first-dapp

https://theethereum.wiki/w/index.php/ERC20_Token_Standard

https://hackernoon.com/setup-your-own-private-proof-of-authority-ethereum-network-with-geth-9a0a3750cda8

https://github.com/ConsenSys/local_ethereum_network
