# Abstract

This document provides an overview of Version 1 of the OST C&C Specification.  It is intended to provide a detailed description of messages and the fields within those messages.

# Introduction

The motivation behind this specification is to improve interoperability between C&C frameworks without requiring cumbersome translation layers.  The goal is for frameworks to implement a standard set of message structures so that implants generated from one framework can be controlled natively from another.  This includes tasking, structured output, and peer-to-peer routing.

This document is not intended to describe what C&C is.  It is assumed the reader understands what it is and what it is used for.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" are to be interpreted as described in [[RFC2119](https://datatracker.ietf.org/doc/html/rfc2119)].

## Environmental Assumptions

This specification makes the following assumptions:

- Messages are sent over an unencrypted network.
- Implant payloads are embedded with the public RSA key used by the team server it is intended to communicate with.

## Glossary of Terms

Below is a list of terms used throughout this document.

- **Implant Metadata**:  Information that an implant reports about itself to a team server.

- **Task Request**:  A task given to an implant to perform.

- **Task Response**:  The status and output (if any) of a given task.

- **Session Key**:  A unique encryption key used by an implant to encrypt its messages.

# Task Messages

## Task Header

Each task request and response message MUST have the following 16-byte header.

```text
| Byte |   0  |   1  |   2  |   3  |   4  |   5  |   6  |   7  |
| -------------------------------------------------------------|
|  0   | Type | Code |    Flags    |           Label           |
| -------------------------------------------------------------|
|  1   |         Identifier        |           Length          |
| -------------------------------------------------------------|
```

- **Type**: 1-byte integer.  The ‘type’ of task this is, e.g. ‘change directory’, ‘list processes’, etc.  See [[Task Types and Codes](https://github.com/rasta-mouse/ost-c2-spec?tab=readme-ov-file#task-types-and-codes)].

- **Code**: 1-byte integer.  A ‘sub code’ for the given Type, e.g ‘connect’, ‘read’, ‘write’, and ‘close’ for SOCKS.  See [[Task Types and Codes](https://github.com/rasta-mouse/ost-c2-spec?tab=readme-ov-file#task-types-and-codes)].

- **Flags**: 2-byte integer.  A set of bitwise flags to describe the state of the message.  See [[Task Flags](https://github.com/rasta-mouse/ost-c2-spec?tab=readme-ov-file#task-flags)].

- **Label**: 4-byte integer.  A unique label to correlate multiple messages related to the same task.

- **Identifier**: 4-byte integer.  A sequential identifier used to construct fragmented messages in the correct order.

- **Length**: 4-byte integer.  The total length of the task data.

## Task Types and Codes

```text
|------------------|--------------------------|
| Type             | Code                     |
|------------------|--------------------------|
| 0 - NOP          | 0                        |
|------------------|--------------------------|
| 1 - Exit         | 0                        |
|------------------|--------------------------|
| 2 - Set          | 0 - Sleep/Jitter         |
|                  | 1 - SpawnTo              |
|                  | 2 - BlockDLLs            |
|                  | 3 - PPID                 |
|------------------|--------------------------|
| 3 - File         | 0 - Copy                 |
|                  | 1 - Move                 |
|                  | 2 - Delete               |
|                  | 3 - Upload               |
|                  | 4 - Download             |
|------------------|--------------------------|
| 4 - Directory    | 0 - Print                |
|                  | 1 - Change               |
|                  | 2 - Create               |
|                  | 3 - Copy                 |
|                  | 4 - Move                 |
|                  | 5 - List                 |
|                  | 6 - Delete               |
|------------------|--------------------------|
| 5 - WhoAmI       | 0                        |
|------------------|--------------------------|
| 6 - Process      | 0 - List                 |
|                  | 1 - Kill                 |
|                  | 2 - Inject Spawn         |
|                  | 3 - Inject Explicit      |
|------------------|--------------------------|
| 7 - Registry     | 0 - Query                |
|                  | 1 - Add                  |
|                  | 2 - Delete               |
|------------------|--------------------------|
| 8 - RPortFwd     | 0 - Bind                 |
|                  | 1 - Read                 |
|                  | 2 - Write                |
|                  | 3 - Close                |
|------------------|--------------------------|
| 9 - Environment  | 0 - Get                  |
|                  | 1 - Set                  |
|------------------|--------------------------|
| 10 - SOCKS       | 0 - Connect              |
|                  | 1 - Read                 |
|                  | 2 - Write                |
|                  | 3 - Close                |
|------------------|--------------------------|
| 11 - Tokens      | 0 - List                 |
|                  | 1 - Make                 |
|                  | 2 - Steal                |
|                  | 3 - Use                  |
|                  | 4 - Revert               |
|                  | 5 - Delete               |
|                  | 6 - Purge                |
|------------------|--------------------------|
| 12 - Run         | 0                        |
|------------------|--------------------------|
| 13 - ItemStore   | 0 - List                 |
|                  | 1 - Add                  |
|                  | 2 - Delete               |
|                  | 3 - Purge                |
|------------------|--------------------------|
| 14 - LocalExec   | 0 - .NET                 |
|                  | 1 - BOF                  |
|                  | 2 - Managed PowerShell   |
|                  | 3 - Unmanaged PowerShell |
|------------------|--------------------------|
| 15 - PrintScreen | 0                        |
|------------------|--------------------------|
| 16 - RemoteExec  | 0 - WinRM                |
|                  | 1 - WMI                  |
|                  | 2 - PsExec               |
|                  | 3 - SSH                  |
|------------------|--------------------------|
| 17 - Link        | 1 - Link SMB             |
|                  | 2 - Link TCP             |
|------------------|--------------------------|
| 18 - Unlink      | 0                        |
|------------------|--------------------------|
| 19 - P2P         | 0 - Acknowledge          |
|                  | 1 - PassThru             |
|------------------|--------------------------|
```

## Task Flags

Some flags are mutually exclusive and MUST NOT be set together (e.g. the encoding flags).  If no flags are set, a task SHOULD be assumed to have completed successfully and NOT fragmented.

```text
| Value | Description                              |
| ----- | ---------------------------------------- |
| 0     | No flags                                 |
| 1     | Task Error                               |
| 2     | Task Running (as job)                    |
| 4     | Message is fragmented, more to follow    |
| 8     | Message is fragmented, no more to follow |
| 16    | Data is JSON encoded                     |
| 32    | Data is XML encoded                      |
| 64    | Data is Protobuf encoded                 |
```

## Task Data

The task data is appended to the header and will consist of a serialised structure, depending on the specific task type and code.  Each task request and response message type are defined in [[Message Definitions](https://github.com/rasta-mouse/ost-c2-spec?tab=readme-ov-file#message-definitions)].

It is NOT MANDATORY for a task request or response to have any data if it is not required.

## Encrypted Task Message

The task header and task data are combined and AES-encrypted with the implant’s session key.

```text
| Byte | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| ------------------------------------ |
|  0   |            Iv                 |
|  8   |                               |
| ------------------------------------ |
|  16  |                               |
|  24  |          Checksum             |
|  32  |                               |
|  40  |                               |
| ------------------------------------ |
|  48  |           Data                |
|  ..  |                               |
| ------------------------------------ |
```

- **Iv**:  A 16-byte initialisation vector.
- **Checksum**:  A 32-byte HMAC256 checksum.
- **Data**:  The encrypted data.

# Message Exchanges

## Implant Registration

An implant MUST register itself with a team server before it can receive or send any task data.

### Generation of IMPLANT-METADATA

The implant generates an [[IMPLANT-METADATA](https://github.com/rasta-mouse/ost-c2-spec?tab=readme-ov-file#implant-metadata)] message, encrypts it with the team server’s public RSA key, and sends it to the team server.

### Receipt of IMPLANT-METADATA

The team server uses its private RSA key to decrypt the implant’s [[IMPLANT-METADATA](https://github.com/rasta-mouse/ost-c2-spec?tab=readme-ov-file#implant-metadata)] and MUST register it as a new session/callback.

## Implant Check In

An implant MUST “check in” with the team server to receive any pending task data for itself or child descendants.

### Check in Request

The method of check in is specific to the C2 channel and not covered under this specification.  A registered implant MAY only send its ID to check in.  However, if the implant has since changed its session key, it MUST also re-send its metadata.

### Check in Response

If there are no pending tasks, a team server MAY respond with no data, or dummy data in the form of one or more [[NOP](https://github.com/rasta-mouse/ost-c2-spec?tab=readme-ov-file#nop)] tasks.  Otherwise, it MUST respond with a collection of task requests that are AES-encrypted with the implant’s session key.

## Peer-to-Peer

### Receipt of LINK-X-REQ

A child implant MUST write its metadata to the P2P channel (e.g. named pipe or TCP socket) once a connection is established with a new parent.

### Generation of LINK-REP

The parent MUST read this metadata and send it back to the team server in a [[LINK-REP](https://github.com/rasta-mouse/ost-c2-spec?tab=readme-ov-file#link-rep)] message.

### Receipt of LINK-REP

The team server MUST decrypt the child’s metadata and register it as a new session/callback or update the existing parent-child relationships in the case of an unlink & link.

### Generation of LINK-ACK

The team server MUST send a [[LINK-ACK](https://github.com/rasta-mouse/ost-c2-spec?tab=readme-ov-file#link-ack)] message back to the new parent to confirm the ID of the child.  The parent SHOULD use the message Label to correlate this process.

### Child Tasks

Tasks for child implants are wrapped in one or more [[LINK-PASS-THRU](https://github.com/rasta-mouse/ost-c2-spec?tab=readme-ov-file#link-pass-thru)] messages.  These will be encrypted with the parent implant’s session key.  Upon receipt, an implant MUST decrypt the message and forward the wrapped data to the child implant ID indicated by the child-id field.

The wrapped data may be the task itself or another LINK-PASS-THRU if the child is another level down the chain.

# Message Definitions

## Optional Integer Fields

Some languages do not distinguish between an omitted integer value and a transmitted value of zero.  For consistency, implementations SHOULD treat omitted values as having been transmitted with a value of zero.

## Timestamp Fields

Timestamps are unsigned 64-bit integers which represents the UNIX epoch (the number of seconds that have elapsed since 1 January 1970).

## Extending the Specification

Implementations MAY include message types, control codes, and flags that are not defined in this specification, as per their unique design and functionality.  It is RECOMMENDED to use values that are on the higher end of the unreserved pool to reduce the chance of them being assigned in a future revision.  Implementations MUST NOT use a type, code, or flag that is defined for anything other than its intended purpose.

### Unrecognised Messages

Implementations SHOULD gracefully handle receiving a message with fields or flags that it does not recognise and return an appropriate error message.

## IMPLANT-METADATA

```text
IMPLANT-META {
  id             [1]   Uint32
  sleep          [2]   UInt32                   OPTIONAL
  jitter         [3]   UInt32                   OPTIONAL
  session-key    [4]   SEQUENCE of Byte (32)
  username       [5]   String
  hostname       [6]   String
  internal-ips   [7]   SEQUENCE of Byte (4)
  process-name   [8]   String
  process-id     [9]   UInt32
  architecture   [10]  [Architecture]
  platform       [11]  [Platform]
  os-description [12]  String                   OPTIONAL
  integrity      [13]  [Integrity]
}
```

### Platform

```text
Platform {
  Linux   = 0,
  MacOS   = 1,
  Windows = 2
}
```

## TASK-ERROR

```text
TASK-ERROR {
  error-code  [1]  Uint32
  message     [2]  String  OPTIONAL
}
```

## NOP Definitions

### NOP

```text
NOP {
  padding  [1]  SEQUENCE of Byte  OPTIONAL
}
```

## Set Definitions

### SET-SLEEP-REQ

```text
SET-SLEEP-REQ {
  sleep   [1]  UInt32
  jitter  [2]  UInt32  OPTIONAL
}
```

### SET-SPAWNTO-REQ

If the `spawnto` field is NOT set, the implant SHOULD revert to its default configuration.

```text
SET-SPAWNTO-REQ {
  spawnto  [1]  String  OPTIONAL
}
```

### SET-BLOCKDLLS-REQ

If the `blockdlls` field is NOT set, the implant SHOULD revert to its default configuration.

```text
SET-BLOCKDLLS-REQ {
  blockdlls  [1]  Boolean  OPTIONAL
}
```

### SET-PPID-REQ

If the `ppid` field is NOT set, the implant SHOULD revert to its default configuration.

```text
SET-PPID-REQ {
  ppid  [1]  UInt32  OPTIONAL
}
```

## FileSystem Definitions

### FILE-COPY-REQ

```text
FILE-COPY-REQ {
  source       [1]  String
  destination  [2]  String
}
```

### FILE-MOVE-REQ

```text
FILE-MOVE-REQ {
  source       [1]  String
  destination  [2]  String
}
```

### FILE-DELETE-REQ

```text
FILE-DELETE-REQ {
  path  [1]  String
}
```

### FILE-UPLOAD-REQ

```text
FILE-UPLOAD-REQ {
  destination  [1]  String
  content      [2]  SEQUENCE of Byte
}
```

### FILE-DOWNLOAD-REQ

```text
FILE-DOWNLOAD-REQ {
  path  [1]  String
}
```

### FILE-DOWNLOAD-REP

```text
FILE-DOWNLOAD-REP {
  file-length    [1]  UInt32
  current-chuck  [2]  Byte
  total-chunks   [3]  Byte
  chunk-content  [4]  SEQUENCE of Byte
}
```

### CHANGE-DIR-REQ

```text
CHANGE-DIR-REQ {
  path  [1]  String
}
```

### CREATE-DIR-REQ

```text
CREATE-DIR-REQ {
  path  [1]  String
}
```

### CREATE-DIR-REP

```text
CREATE-DIR-REP {
  entry  [1]  [FileSystemEntry]
}
```

### COPY-DIR-REQ

```text
COPY-DIR-REQ {
  source       [1]  String
  destination  [2]  String
}
```

### MOVE-DIR-REQ

```text
MOVE-DIR-REQ {
  source       [1] String
  destination  [2]  String
}
```

### LIST-DIR-REQ

```text
LIST-DIR-REQ {
  path            [1]  String
  access-control  [2]  [FileSecurity]  OPTIONAL
}
```

### LIST-DIR-REP

```text
LIST-DIR-REP {
  entries  [1]  SEQUENCE of [FileSystemEntry]
}
```

### DELETE-DIR-REQ

```text
DELETE-DIR-REQ {
  path     [1]  String
  recurse  [2]  Boolean  OPTIONAL
  force    [3]  Boolean  OPTIONAL
}
```

### FileSystemEntry

```text
FileSystemEntry {
  path            [1]  String
  length          [2]  UInt32
  attributes      [3]  [FileAttributes]  OPTIONAL
  owner           [4]  String            OPTIONAL
  created         [5]  Timestamp         OPTIONAL
  last-accessed   [6]  Timestamp         OPTIONAL
  last-written    [7]  Timestamp         OPTIONAL
  access-control  [8]  [FileSecurity]    OPTIONAL
}
```

### FileAttributes

Bitwise flags.

```text
FileAttributes {
  Normal     = 1,
  Archive    = 2,
  Compressed = 4,
  ReadOnly   = 8,
  Hidden     = 16,
  Directory  = 32,
  System     = 64
}
```

### FileSecurity

```text
FileSecurity {
  identity     [1]  String
  access-mask  [2]  Int32
  inheritance  [3]  [Inheritance]  OPTIONAL
  propagation  [4]  [Propagation]  OPTIONAL
}
```

### Inheritance

Bitwise flags.

```text
Inheritance {
  None             = 0,
  ContainerInherit = 1,
  ObjectInherit    = 2,
}
```

### Propagation

Bitwise flags.

```text
Propagation {
  None               = 0,
  NoPropagateInherit = 1,
  InheritOnly        = 2,
}
```

## WhoAmI Definitions

### WHOAMI-REP

```text
WHOAMI-REP {
  primary        [1]  String
  impersonation  [2]  String  OPTIONAL
}
```

## Process Definitions

### PROC-LIST-REP

```text
PROC-LIST-REP {
  processes  [1]  SEQUENCE of [ProcessEntry]
}
```

### PROC-KILL-REQ

```text
PROC-KILL-REQ {
  process-id  [1]  UInt32
  force       [2]  Boolean  OPTIONAL
}
```

### PROC-INJ-SPAWN-REQ

```text
PROC-INJ-SPAWN-REQ {
  shellcode  [1]  SEQUENCE of Byte
  technique  [2]  [InjectionTechnique]  OPTIONAL
}
```

### PROC-INJ-EXPLIC-REQ

```text
INJECT-EXPLIC-REQ {
  process-id  [1]  UInt32
  shellcode   [2]  SEQUENCE of Byte
  technique   [3]  [InjectionTechnique]  OPTIONAL
}
```

### ProcessEntry

```text
ProcessEntry {
  process-name       [1]  String
  process-id         [2]  UInt32
  parent-process-id  [3]  UInt32          OPTIONAL
  session-id         [4]  Byte            OPTIONAL
  owner              [5]  String          OPTIONAL
  architecture       [6]  [Architecture]  OPTIONAL
  integrity          [7]  [Integrity]     OPTIONAL
}
```

### Architecture

```text
Architecture {
  X86   = 0,  // 32-bit Intel
  X64   = 1,  // 64-bit Intel
  Arm   = 2,  // 32-bit ARM
  Arm64 = 3,  // 64-bit ARM
  Wasm  = 4   // WebAssembly
}
```

### Integrity

```text
Integrity {
  Untrusted = 0,
  Low       = 1,
  Medium    = 2,  // user
  High      = 3,  // sudoers
  System    = 4   // root
}
```

### InjectionTechnique

```text
InjectionTechnique {
  CreateThread        = 0,
  QueueUserApc        = 1,
  SetThreadContext    = 2,
  RtlCreateUserThread = 3
}
```

## Registry Definitions

### REQ-QUERY-REQ

```text
REQ-QUERY-REQ {
  hive            [1]  [RegistryHive]
  key             [2]  String          OPTIONAL
  value           [3]  String          OPTIONAL
  access-control  [4]  Boolean         OPTIONAL
}
```

### REQ-QUERY-REP

```text
REQ-QUERY-REP {
  values  [1]  SEQUENCE of [RegistryValue]
  keys    [2]  SEQUENCE of [RegistryKey]
}
```

### REQ-ADD-REQ

```text
REG-ADD-REQ {
  hive   [1]  [RegistryHive]
  key    [2]  String
  value  [3]  [RegistryValueKind]
  type   [4]  String
  value  [5]  SEQUENCE of Byte
}
```

### REG-DELETE-REQ

```text
REG-DELETE-REQ {
  hive   [1]  [RegistryHive]
  key    [2]  String
  value  [3]  String          OPTIONAL
  force  [4]  Boolean         OPTIONAL
}
```

### RegistryHive

```text
RegistryHive {
  ClassesRoot   = 0,
  CurrentUser   = 1,
  LocalMachine  = 2,
  Users         = 3,
  CurrentConfig = 4
}
```

### RegistryKey

```text
RegistryKey {
  hive            [1]  [RegistryHive]
  name            [2]  String
  access-control  [3]  [RegistrySecurity]  OPTIONAL
}
```

### RegistryValue

```text
RegistryValue {
  name            [1]  String
  type            [2]  [RegistryValueKind]
  data            [3]  SEQUENCE of Byte
  access-control  [4]  [RegistrySecurity]  OPTIONAL
}
```

### RegistryValueKind

```text
RegistryValueKind {
  None         = 0,  // REG_NONE
  String       = 1,  // REG_SZ
  ExpandString = 2,  // REG_EXPAND_SZ
  Binary       = 3,  // REG_BINARY
  DWord        = 4,  // REG_DWORD
  MultiString  = 5,  // REG_MULTI_SZ
  Qword        = 6   // REG_QWORD
}
```

### RegistrySecurity

```text
RegistrySecurity {
  identity     [1]  String
  access-mask  [2]  Int32
  inheritance  [3]  [Inheritance]  OPTIONAL
  propagation  [4]  [Propagation]  OPTIONAL
}
```

## Reverse Port Forward Definitions

### RPORTFWD-BIND

```text
RPORTFWD-BIND {
  port       [1]  UInt32
  localhost  [2]  Boolean
}
```

### RPORTFWD-READ

```text
RPORTFWD-READ {
  data  [1]  SEQUENCE of Byte
}
```

### RPORTFWD-WRITE

```text
RPORTFWD-WRITE {
  data  [1]  SEQUENCE of Byte
}
```

## Environment Definitions

### ENV-GET-REQ

```text
ENV-GET-REQ {
  key  [1]  String
}
```

### ENV-GET-REP

```text
ENV-GET-REP {
  value  [1]  String
}
```

### ENV-SET-REQ

```text
ENV-SET-REQ {
  key    [1]  String
  value  [2]  String
}
```

## SOCKS Definitions

### SOCKS-CONNECT-REQ

```text
SOCKS-CONNECT-REQ {
  target  [1]  SEQUENCE of Byte  (4)
  port    [2]  Uint32
}
```

### SOCKS-WRITE-REQ

```text
SOCKS-WRITE-REQ {
  data  [1]  SEQUENCE of Byte
}
```

### SOCKS-READ-REQ

```text
SOCKS-READ-REQ {
  data  [1]  SEQUENCE of Byte
}
```

## Token Definitions

### TOKEN-LIST-REP

```text
TOKEN-LIST-REP {
  tokens  [1]  SEQUENCE of [TokenEntry]
}
```

### TOKEN-CREATE-REQ

```text
TOKEN-CREATE-REQ {
  username  [1]  String
  domain    [2]  String  OPTIONAL
  password  [3]  String  OPTIONAL
}
```

### TOKEN-STEAL-REQ

```text
TOKEN-STEAL-REQ {
  process-id   [1]  UInt32
  access-mask  [2]  UInt32  OPTIONAL
}
```

### TOKEN-USE-REQ

```text
TOKEN-USE-REQ {
  index  [1]  Byte
}
```

### TOKEN-DELETE-REQ

```text
TOKEN-STEAL-REQ {
  index  [1]  Byte
}
```

### TokenEntry

```text
Token {
  index       [1]  Byte
  username    [2]  String
  handle      [3]  String  OPTIONAL
  process-id  [4]  UInt32  OPTIONAL
}
```

## Implant Store Definitions

### LIST-STORE-REP

```text
LIST-STORE-REP {
  items  [1]  SEQUENCE of [StoreItem]
}
```

### ADD-STORE-ITEM Definition

```text
ADD-STORE-ITEM-REQ {
  item  [1]  SEQUENCE of Byte
  name  [2]  String
  type  [3]  [StoreItemType]
}
```

### StoreItem

```text
StoreItem {
  index  [1]  Byte
  name   [2]  String
}
```

### StoreItemType

```text
StoreItemType {
  Assembly = 0,
  BOF      = 1,
  Script   = 2,
  Generic  = 3
}
```

## Local Execution Definitions

### RUN-REQ

```text
RUN-REQ {
  program    [1]  String
  arguments  [2]  String  OPTIONAL
  token      [3]  Byte    OPTIONAL
}
```

### RUN-REP

```text
RUN-REP {
  output  [1]  String
}
```

### EXEC-ASM-REQ

Either store-index or assembly MUST be provided.

```text
EXEC-ASM-REQ {
  store-index  [1]  Byte                OPTIONAL
  assembly     [2]  SEQUENCE of Byte    OPTIONAL
  arguments    [3]  SEQUENCE of String  OPTIONAL
  bypass-amsi  [4]  Boolean             OPTIONAL
  bypass-etw   [5]  Boolean             OPTIONAL
}
```

### EXEC-ASM-REP

```text
EXEC-ASM-REP {
  output  [1]  String
}
```

### EXEC-BOF-REQ

Either store-index or bof MUST be provided.

```text
EXEC-BOF-REQ {
  store-index  [1]  Byte              OPTIONAL
  bof          [2]  SEQUENCE of Byte  OPTIONAL
  arguments    [3]  SEQUENCE of Byte  OPTIONAL
  bypass-amsi  [4]  Boolean           OPTIONAL
  bypass-etw   [5]  Boolean           OPTIONAL
}
```

### EXEC-BOF-REP

```text
EXEC-BOF-REP {
  output  [1]  String
}
```

### EXEC-POSH-REQ

Either store-index or script MUST be provided.

```text
EXEC-POSH-REQ {
  cmdlet       [1]  String
  store-index  [2]  Byte              OPTIONAL
  script       [3]  SEQUENCE of Byte  OPTIONAL
  bypass-amsi  [3]  Boolean           OPTIONAL
  bypass-etw   [4]  Boolean           OPTIONAL
}
```

### EXEC-POST-REP

```text
EXEC-POST-REP {
  output  [1]  String
}
```

## Screenshot Definitions

### SCRNSHOT-REP

```text
SCRNSHOT-REP {
  data  [1]  SEQUENCE of Byte
}
```

## Remote Execution Definitions

### WINRM-REQ

```text
WINRM-REQ {
  target     [1]  String
  program    [2]  String
  arguments  [3]  String  OPTIONAL
}
```

### WMI-REQ

```text
WMI-REQ {
  target     [1]  String
  program    [2]  String
  arguments  [3]  String  OPTIONAL
}
```

### PSEXEC-REQ

```text
PSEXEC-REQ {
  target               [1]  String
  service-name         [2]  String
  service-description  [3]  String  OPTIONAL
  bin-path             [4]  String
}
```

## Peer-to-Peer Definitions

### LINK-SMB-REQ

```text
LINK-SMB-REQ {
  target    [1]  String
  pipename  [2]  String
}
```

### LINK-TCP-REQ

```text
LINK-TCP-REQ {
  target  [1]  String
  port    [2]  UInt32
}
```

### LINK-REP

```text
LINK-SMB-REP {
  child-metadata  [1]  SEQUENCE of Byte
}
```

### LINK-ACK

```text
LINK-ACK {
  child-id  [1]  UInt32
}
```

### LINK-PASS-THRU

```text
LINK-PASS-THRU {
  child-id  [1]  UInt32
  message   [2]  SEQUENCE of Byte
}
```
