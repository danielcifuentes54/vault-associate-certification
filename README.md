<img src="icons/vault.png" />

#  Vault Associate Certification
Information regarding the Vault Associate Certification

## Exam Objectives

* Compare authentication methods
* Create Vault policies
* Assess Vault tokens
* Manage Vault leases
* Compare and configure Vault secrets engines
* Utilize Vault CLI
* Utilize Vault UI
* Be aware of the Vault API
* Explain Vault architecture
* Explain encryption as a service

## What is Vault

* Manage secrets and protect sensitive data providing a single source of secrets for both humans and machines, also provides complete lifecycle management for secrets (usernames and passwords, API keys, certificates, encryption keys)

## Vault interfaces

* API
* UI 
* CLI

## Token Generation

1. [user, machine] Authentication on vault
2. [vault] Generate token
3. [user, machine] Retrieve data from a path

## Benefits

* Store Long-lived static secrets
* Dynamically generate secrets upon request
* Fully-featured API
* Identity-based access across different clouds and systems
* Provide Encryption as a service
* Act as a Root or intermediate certificate authority

> Centralize the storage of secrets across the organization into a consolidated platform

## Versions:

* Open Source: 
  * No replication capabilities
  * Does not include access to Enterprise integrations (MFA, HSM,Automated Backups)
* Enterprise:
  * Self-managed
* Vault on HCP (Hashicorp cloud platform)

## Vault Architecture

### Vault Components

* Storage Backends:
  * Configures location for storage of vault data.
  * Storage is defined in the main Vault configuration file with desired parameters
  * All data is encrypted in transit (TLS) and at-rest using AES256
  * Storage backend examples: S3, Consul, DynamoDB
  * there is only one storage backend per Vault cluster
* Secrets Engines:
  * Vault components tha are responsible for managing secrets for your organization.
  * Secrets engines can store, generate, or encrypt data.
  * Many secrets engines connect to other services to generate dynamic credentials on-demand
  * Many secrets engines can be enabled and used as needed (even multiple secrets engines of the same type)
  * Secret engines are enabled and isolated at a "path"
* Authentication Methods
  * Vault components that perform authentication and manage identities.
  * Multiple authentication methods can ve enabled depending on your use case
  * Once authenticated, Vault will issue a client token used to make all subsequent Vault requests, **the fundamental goal of all auth methods is to obtain a token**
  * Each token has an associated **policy** (or policies) and a **TTL**.
  Default authentication method for a new Vault deployment = tokens
* Audit Devices
  * Keeps detailed JSON log of all requests and responses to vault.
  * Sensitive information is hashed before logging.
  * Can (and should) have **more than one** audit device enabled (Vault **Prioritizes safety over availability)

### High Level Architecture

* HTTP/s API :arrow_right: Barrier :arrow_right: Vault Core (Token Store, Path Routing, secrets Engines, Systmen Backend, Auth Method, Audit device...) :arrow_right: Barrier :arrow_right: Storage Backend

### Vault Paths

* Everything in Vault is **path-based**
* The path prefix tell Vault which component a request should be routed.
* Secret engines, auth methods, and audit devices are  "mounted" at a specified path
* Paths available are dependent on the features enabled in Vault, such as Auth methods and secrets engines
* System backend is a default backend in Vault which is mmounted at the /sys endpoint
* Vault components can be enabled at ANY path you'd like using the -path flag (Each component have a default path)
* Vault has a few system reserved path which you cannot use or remove:
  * auth/
  * cubbyhole/
  * identity/
  * secret/
  * sys/

### Vault Data Protection

* Master Key:
  * used to decrypt the encryption key.
  * Created during Vault initialization or during a rekey operation
  * Never written to storage using traditional unseal mechanism
  * Written to core/master (storage backend) when using Auto Unseal
* Encryption Key:
  * Used to encrypt/decrypt data written to storage backend
  * Encrypted by the Master Key
  * Stored alongside the data in a keyring on the storage backend.
  * Can be easily rotated (manual operation)

### Seal and Unseal

* Vault stats in a **sealed state**, meaning it knows where to access the data, and how, but can't decrypt it (you can also manually seal it via UI, API, or CLI).
* Almost **no operations** are possible when Vault os on a sealed state (only status check and unsealing are possible)
* Unsealing Vault means that **a node can reconstruct the master key** in order to decrypt the encryption key, and ultimately and read the data
* After unsealing, the encryption key is stored in memory
* Sealing Vault means Vault "throws away" the encryption key and requires another unseal to perform any further operations.
* When would I seal Vault?
  * Key shards are inadvertently exposed.
  * Detection of a compromise or network intrusion.
  * Spyware/malware on the Vault nodes.
* Unseal Options:
  * Unsealing with key shards (Sharmir):
    * Default option for unsealing.
    * No single person should have access to all key shards
    * Ideally, each key shard should be stored by a different employee
    * When initializing Vault, you can request the individual shards to be encrypted with different PGP Keys
    * When unsealing Vault,yYou will need an equal number of employees to provide they key which is equal to the threshold
    * Key shards should not be stored online and shoul be highly protected - ideally stored encypted
  * Unsealing with Auto Unseal:
    * Uses a cloud service to protect our master key
    * Cloud Auto Unseal automatically unseals Vault upon service or node restart **without additional intervention**
    * Available in both open source & Enterprise editions
    * You must to configure this unseal in the configuration file, for example:
      ``` bash 
      seal "awskms" {
        region = "AWS_REGION"
        kms_key_id = "AWS_KMS_KEY _ID"
      }
      ```
  * Unsealing with Transit Auto Unseal
    * Have a Vault cluster for unsealing (it generates a dependency), uses the Transit Secret Engine of a different Vault cluster
    * Must be configured in a namespace
    * Support key rotation
    * Available in open source & Enterprise editions
    * The core Vault cluster must be higly-available
    * You must to configure this unseal in the configuration file, for example:
      ``` bash 
      seal "transit" {
        address     = "VAULT_ADDRESS"
        token       = "VAULT_TOKEN"
        key_name    = "KEY_NAME"
        mounth_path = "MOUNTH_PATH"

        // Key configuration
        ...

        // TLS Configuration
        ...
      }
      ```
### Vault Initialization

* Prepares the backend storage to receive data.
* Only need to initialize a Vault cluster one time via single node.
* Is when Vault creates the master key and key shares.
* You can set options to define thresholds, key shares, recovery keys, and encryption.
* Is also where the initial root token is generated and returned to the user.
* Vault can be initialized via CLI, API, or UI.
* CLI command: `vault operator init <OPTIONS>`

### Configuration File

* Vault servers are confifured using a file (written in HCL or JSON)
* Configuration file is specified when starting Vault using the `-config` flag
  * `vault server -config <FILE_PATH>`
* The only Vault component that is configured in the file is the storage backend
* Available Stanzas:
  * seal
  * listener
  * storage
  * telemetry
* Example of root parameters: 
  * cluster_name
  * log_level
  * ui
  * api_addr
  * cluster_addr

### Storage Backends

* Enterprise Vault Cluster should use Hashicorp Consul or  Integrated Storage
* There is **only one** storage backend per Vault cluster (not per node)
* In Enterprise, the replications happens in a cluster or node level (not in a storage backend level)

### Audit Device

* Keep detailed log of all authenticated requests and responses to Vault
* Audit log is formatted using JSON
* Sensitive information is hashed with a salt using HMAC-SHA256
* Log files should be protected as a user with permission can still check the value of those secrets via the /sts/audit-hash API and compare to the log file
* Log Types:
  * File
  * Syslog
  * Socket
* Can and should have more than one audit device enabled
* If Vault cannot write to a persistent log, it will stop responding to client requests - which means Vault is down!.

### Vault Interfaces

* Three interfaces to interact with Vault: UI, CLI, and HTTP API.
* Not all Vault features are available via UI and CLI but **all** the features can be accesed using the HTTP API.
* Calls from the CLI and UI invoke the HTTP API.
* UI must be enabled via configuration file.
* Authentication required to access any of the interfaces

## Vault Commands

### Init Vault
`vault operator init -key-shares=3 -key-threshold=2`

### Ckeck Configuration File
`vault operator diagnose -config=/etc/vault.d/vault.hcl`

### Unseal Vault
`vault operator unseal`

### Unseal Vault & Migrate
`vault operator unseal -migrate `

### login in Vault

`vault login`

### Enable Secret Engine
`vault secrets enable -path=secrets kv-v2`
### List Secrets
`vault secrets list`
### Enable transit

`vault secrets enable transit`

### Create an encryption key

`vault write -f <KEY_PATH/KEY_NAME>`