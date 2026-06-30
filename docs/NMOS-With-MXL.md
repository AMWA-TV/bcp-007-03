# AMWA BCP-007-03: NMOS With MXL
{:.no_toc}  
---
  
{:toc}

## Introduction

[MXL][] provides an open and non-proprietary exchange layer within Dynamic Media Facilities.

This document outlines how MXL enabled media functions can be managed through AMWA [IS-04][] and [IS-05][].

## Use of Normative Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119][RFC-2119].

## Definitions

The NMOS terms 'Controller', 'Node', 'Source', 'Flow', 'Sender', 'Receiver' are used as defined in the [NMOS Glossary](https://specs.amwa.tv/nmos/main/docs/Glossary.html).

## MXL IS-04 Sources, Flows and Senders

Nodes that encapsulate media functions containing MXL writers MUST expose Source, Flow and Sender resources for each MXL writer in their IS-04 Node API.

Nodes compliant with this specification MUST implement IS-04 v1.3 or higher.

### Sources

An MXL Source resource MUST set the `format` attribute to a value specified in the [NMOS formats parameter register][].

Examples of Source resources are provided in [Examples](../examples/).

### Flows

An MXL Flow resource MUST set the `format` attribute to a value specified in the [NMOS formats parameter register][].

An MXL Flow resource MUST set the `media_type` attribute to a value specified in the [NMOS media types parameter register][].

Typical values used by MXL Flow resources for the `media_type` attribute include but are not limited to:

* `video/v210`
* `video/v210a`
* `audio/float32`
* `video/smpte291`

Examples of Flow resources are provided in [Examples](../examples/).

### Senders

An MXL Sender resource MUST set the `transport` attribute to `urn:x-nmos:transport:mxl`.

An MXL Sender resource MUST expose an empty `interface_bindings` array.

An example Sender resource is provided in [Examples](../examples/).

#### Transport file usage

MXL does not require a transport file.

The `manifest_href` attribute of an MXL Sender MUST always be set to `null`.

The response for requests against the `/transportfile` endpoint of an MXL IS-05 Sender MUST always return a 404.

## MXL IS-04 Receivers

Nodes that encapsulate media functions containing MXL readers MUST expose a Receiver resource for each MXL reader in their IS-04 Node API.

An MXL Receiver resource MUST set the `transport` attribute to `urn:x-nmos:transport:mxl`.

An MXL Receiver resource MUST expose an empty `interface_bindings` array.

An MXL Receiver resource MUST set the `format` attribute to a value specified in the [NMOS formats parameter register][].

An MXL Receiver resource MUST set the `media_types` attribute to at least one value. The values used MUST be specified in the [NMOS media types parameter register][].

Typical values used by MXL Receiver resources for the `media_types` attribute include but are not limited to:

* `video/v210`
* `video/v210a`
* `audio/float32`
* `video/smpte291`

The Receiver MUST express its limitations or preferences regarding the MXL Flows that it supports consuming by declaring Receiver Capabilities in accordance with the [BCP-004-01][] specification. The Receiver SHOULD express its capabilities as precisely as possible, to enable a Controller to determine, with high confidence, the Receiver's compatibility with available MXL Flows. It is not always practical for the parameter constraints to enumerate every type of MXL Flow a Receiver can or cannot consume; however, they SHOULD describe as many commonly used operating points as practical, along with any preferences.

The Receiver MUST use the `constraint_sets` parameter within the `caps` object to describe supported combinations of parameters, using the parameter constraints defined in the [Capabilities Register](https://specs.amwa.tv/nmos-parameter-registers/branches/main/capabilities/) of the NMOS Parameter Registers. The full details are described in [BCP-004-01][] NMOS Receiver Capabilities.

An example Receiver resource is provided in [Examples](../examples/).

## MXL IS-05 Senders and Receivers

Nodes compliant with this specification MUST implement IS-05 v1.2 or higher.

Connection Management using IS-05 proceeds in the same manner as for any other transport, using MXL-specific transport parameters defined in [MXL Sender transport parameters](../APIs/schemas/sender_transport_params_mxl.json) and [MXL Receiver transport parameters](../APIs/schemas/receiver_transport_params_mxl.json).

The `mxl_domain_id` and `mxl_flow_id` transport parameters MUST be present in the IS-05 `active`, `staged`, and `constraints` endpoints of an MXL Sender and Receiver.

MXL Senders and Receivers MUST always use a single set of transport parameters in the transport parameters arrays in both the staged and active endpoints.

MXL Senders and Receivers MUST always use a single set of constraints in the constraints endpoint array.

### Transport Parameters

| Name           | Description                                                                                                                                                                                                                                  |
|----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `mxl_domain_id`| Specifies the MXL Domain ID where the MXL Flow will be located. The Sender and Receiver list allowed values in the `constraints` endpoint.               |
| `mxl_flow_id`  | Specifies the MXL Flow ID which will be used for the write or read operation. The Sender and Receiver list allowed values in the `constraints` endpoint. |

Where a UUID value is used, `mxl_domain_id` and `mxl_flow_id` MUST conform to the [MXL UUID schema](../APIs/schemas/mxl_uuid.json), matching the canonical textual UUID form defined in [MXL Addressability][MXL addressability].

Where a value is not yet determined, implementations MUST use `null`, consistent with [IS-05 *APIs: Server Side Implementation*][IS-05 uninitialised].

[IS-05 *APIs: Server Side Implementation*, *Use of auto*][IS-05 use of auto] allows `"auto"` in `/staged` so that the Sender or Receiver may select a transport parameter value itself. API implementations MUST NOT list `"auto"` as an option via the `/constraints` endpoint.

Support for `"auto"` indicates that an implementation supports automatic-selection semantics for that parameter in `/staged`. It does not guarantee that every staged configuration can be activated.

A Sender or Receiver MUST reject a staging request that supplies a parameter value which is invalid against the schema and constraints, or which cannot be applied in the current operating context (for example, an `mxl_domain_id` that refers to an MXL Domain the Node is not capable of accessing).

For an immediate activation, if `"auto"` cannot be resolved to a valid value the request MUST respond with an HTTP 500 error. For a scheduled activation, Controllers SHOULD verify the outcome after the expected activation time by inspecting `/active` and monitoring IS-04 resource versions, consistent with [IS-05 Scheduled Activations](https://specs.amwa.tv/is-05/releases/v1.1.2/docs/Behaviour.html#scheduled-activations).

#### Automatic resolution of `mxl_domain_id`

When `mxl_domain_id` is staged or activated with `"auto"`, the Sender or Receiver SHOULD first use any available local or contextual information (for example a single configured MXL Domain) to determine the MXL Domain id.

If a Receiver cannot determine the MXL Domain ID by such means, it SHOULD enumerate or query all MXL Domains available to the Node and search for an MXL Flow that satisfies the connection by matching the staged `mxl_flow_id`.

#### Sender Transport Parameters

- `mxl_flow_id` MUST accept `null`, including where the MXL Flow is not yet configured. It MUST support `"auto"` where the Sender resolves the MXL Flow identifier (for example when that ID is provided via configuration or is randomly generated, so a Controller need not supply the identifier when staging or activating). Where `"auto"` is used and cannot be resolved to a valid value for `/active` in the current operating context, the Sender MUST reject the request or activation.
- `mxl_domain_id` MUST accept `null` including where the MXL Domain is unknown a priori in a multi-domain system. It MUST support `"auto"` where a single MXL Domain applies or the Sender resolves the MXL Domain without a Controller supplied ID. Where `"auto"` is used and cannot be resolved to a valid value for `/active` in the current operating context, the Sender MUST reject the request or activation.

#### Receiver Transport Parameters

- `mxl_flow_id` MUST accept `null` for an unconfigured Receiver as the identifier may be unknown until the Receiver has been staged or activated, and MUST NOT accept `"auto"`.
- `mxl_domain_id` MUST accept `null` for an unconfigured Receiver and MUST support `"auto"` where the Receiver is able to automatically determine the Domain. Where `"auto"` is used and cannot be resolved to a valid value for `/active` in the current operating context, the Receiver MUST reject the request or activation.

Note that the `mxl_flow_id` need not be the same as the ID of the associated IS-04 Flow resource.

### Receivers

A staging or activation request on an MXL IS-05 Receiver MUST NOT include a transport file. A Receiver MUST accept a request that either omits the `transport_file` attribute or sets both `data` and `type` within the `transport_file` to `null`.

A successful activation resulting in `master_enable` becoming `true` MUST start the MXL read operation.

A successful activation resulting in `master_enable` becoming `false` MUST stop the MXL read operation.

### Senders

A successful activation resulting in `master_enable` becoming `true` MUST start the MXL write operation.

A successful activation resulting in `master_enable` becoming `false` MUST stop the MXL write operation.

## MXL Domain volume and identity mapping

The base directory where MXL Flows are stored is called an MXL Domain.
Multiple MXL Domains can co-exist on the same host.  
MXL Domains are mapped to media functions at deployment time through volume mapping.
This means that the path where an MXL Domain directory is located inside a container could be different from the path where the MXL Domain directory is located on the host.

Consider the following simple deployment example where we have two mxl-writer media functions and an mxl-reader media function.

```yml
services:
  writer-media-function-1:
    image: mxl-writer:latest
    volumes:
      - type: bind
        source: /Volumes/mxl/domain_1
        target: /domain_1
  writer-media-function-2:
    image: mxl-writer:latest
    volumes:
      - type: bind
        source: /Volumes/mxl/domain_2
        target: /domain_2
  reader-media-function:
    image: mxl-reader:latest
    volumes:
      - type: bind
        source: /Volumes/mxl/domain_1
        target: /domain_a
        read_only: true
      - type: bind
        source: /Volumes/mxl/domain_2
        target: /domain_b
        read_only: true
```

Without applying any strong identity to the MXL Domains the reader media function cannot resolve `domain_a` and `domain_b` as being the same as `domain_1` and `domain_2`.

All MXL Domains MUST hold a definition json file `domain_def.json` in their host directory.
The MXL Domain definition json object MUST respect the [MXL Domain definition schema](../APIs/schemas/mxl_domain_definition.json) where the following attributes are defined as:

* id - unique identity of the MXL Domain as an [MXL UUID](../APIs/schemas/mxl_uuid.json)
* label - label of the MXL Domain as a string
* description - description of the MXL Domain as a string
* tags - tags object for the MXL Domain

An example MXL Domain definition is provided in [Examples](../examples/).

It is assumed that media functions will be configured by an orchestrator with the MXL Domain's location on the local filesystem, allowing them to discover available mapped domains, and their identity, by checking the contents of each MXL Domain for the `domain_def.json` file.
The identity of each MXL Domain travels with each MXL Domain mapping inside each media function, meaning media functions can resolve MXL Domains to the same identity even when they have been mapped to different local paths inside the media function.

Given the above deployment example, this is the local path structure inside each of the media functions:

writer-media-function-1:

```txt
mxl-holder/
├── domain-01/
│   ├── domain_def.json
│   └── 4c02e7c0-ffeb-4595-8673-a265d103123a.mxl-flow/
│       └── flow_def.json
```

mxl-holder/domain-01/domain_def.json

```json
{
    "id": "1ac254d9-a5eb-475f-a2b6-3d02a5cfbc82",
    "label": "Red Studio",
    "description": "MXL Red Studio MXL Domain",
    "tags": {}
}
```

writer-media-function-2:

```txt
my-mxl-holder/
├── domain-02/
│   ├── domain_def.json
│   └── 3494dc19-22c2-4f00-865b-cdbeee7909f0.mxl-flow/
│       └── flow_def.json
```

my-mxl-holder/domain-02/domain_def.json

```json
{
    "id": "3310f209-9351-47c0-b9a2-14c59b6a4c23",
    "label": "Blue Studio",
    "description": "MXL Blue Studio MXL Domain",
    "tags": {}
}
```

reader-media-function:

```txt
base-mxl-holder/
├── domain-a/
│   ├── domain_def.json
│   └── 4c02e7c0-ffeb-4595-8673-a265d103123a.mxl-flow/
│       └── flow_def.json
├── domain-b/
│   ├── domain_def.json
│   └── 3494dc19-22c2-4f00-865b-cdbeee7909f0.mxl-flow/
│       └── flow_def.json
```

base-mxl-holder/domain-a/domain_def.json

```json
{
    "id": "1ac254d9-a5eb-475f-a2b6-3d02a5cfbc82",
    "label": "Red Studio",
    "description": "MXL Red Studio MXL Domain",
    "tags": {}
}
```

base-mxl-holder/domain-b/domain_def.json

```json
{
    "id": "3310f209-9351-47c0-b9a2-14c59b6a4c23",
    "label": "Blue Studio",
    "description": "MXL Blue Studio MXL Domain",
    "tags": {}
}
```

where `/domain-a` and `/domain-b` inside the reader media function resolve to the same identity as `/domain-1` and `/domain-2` mapped in the writer media functions.

## Controllers

A Controller MUST be able to discover MXL Senders and MXL Receivers by using the IS-04 Query API.

A Controller MUST be able to connect an MXL Receiver to an MXL Sender by using the IS-05 Connection API.

When a Controller makes a staging or activation request on an MXL IS-05 Receiver it MUST NOT provide a `transport_file`. The Controller MUST either omit the `transport_file` attribute or set both the `data` and `type` within `transport_file` to `null`.

Controllers MUST support the BCP-004-01 Receiver Capabilities mechanism in order to evaluate the MXL Flow compatibility between MXL Senders and MXL Receivers.


[RFC-2119]: https://tools.ietf.org/html/rfc2119 "Key words for use in RFCs"
[MXL]: https://tech.ebu.ch/dmf/mxl
[MXL addressability]: https://github.com/dmf-mxl/mxl/blob/main/docs/Addressability.md#uuid-format "MXL Addressability — UUID Format"
[IS-04]: https://specs.amwa.tv/is-04/
[IS-05]: https://specs.amwa.tv/is-05/
[IS-05 uninitialised]: https://specs.amwa.tv/is-05/releases/v1.1.2/docs/APIs_-_Server_Side_Implementation.html#uninitialised-senders-and-receivers "IS-05 APIs: Server Side Implementation — Uninitialised Senders and Receivers"
[IS-05 use of auto]: https://specs.amwa.tv/is-05/releases/v1.1.2/docs/APIs_-_Server_Side_Implementation.html#use-of-auto "IS-05 APIs: Server Side Implementation — Use of auto"
[BCP-004-01]: https://specs.amwa.tv/bcp-004-01/ "AMWA BCP-004-01 NMOS Receiver Capabilities"
[NMOS formats parameter register]: https://specs.amwa.tv/nmos-parameter-registers/branches/main/formats/ "NMOS Formats"
[NMOS media types parameter register]: https://specs.amwa.tv/nmos-parameter-registers/branches/main/media-types/ "NMOS Media Types"
