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

The Receiver MUST express its limitations or preferences regarding the flows that it supports consuming by declaring Receiver Capabilities in accordance with the [BCP-004-01][] specification. The Receiver SHOULD express its capabilities as precisely as possible, to enable a Controller to determine, with high confidence, the Receiver's compatibility with available MXL flows. It is not always practical for the parameter constraints to enumerate every type of flow a Receiver can or cannot consume; however, they SHOULD describe as many commonly used operating points as practical, along with any preferences.

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
| `mxl_domain_id`| Specifies the MXL domain id where the MXL flow will be located. The Sender and Receiver list allowed values in the `constraints` endpoint.               |
| `mxl_flow_id`  | Specifies the MXL flow id which will be used for the write or read operation. The Sender and Receiver list allowed values in the `constraints` endpoint. |

MXL Senders and Receivers MUST NOT allow the special value `auto` for `mxl_domain_id` or `mxl_flow_id`.

Note that the `mxl_flow_id` need not be the same as the ID of the associated IS-04 Flow resource.

### Receivers

A `PATCH` request on the **/staged** endpoint of an MXL IS-05 Receiver is not expected to contain a transport file in the `transport_file` attribute.

A successful activation resulting in `master_enable` becoming `true` MUST start the MXL read operation.

A successful activation resulting in `master_enable` becoming `false` MUST stop the MXL read operation.

### Senders

A successful activation resulting in `master_enable` becoming `true` MUST start the MXL write operation.

A successful activation resulting in `master_enable` becoming `false` MUST stop the MXL write operation.

## Controllers

A controller MUST be able to discover MXL Senders and MXL Receivers by using the IS-04 Query API.

A controller MUST be able to connect an MXL Receiver to an MXL Sender by using the IS-05 Connection API.

When a controller makes a `PATCH` request on the **/staged** endpoint of an MXL IS-05 Receiver it MUST NOT provide the `transport_file` attribute.

Controllers MUST support the BCP-004-01 Receiver Capabilities mechanism in order to evaluate the flow compatibility between MXL Senders and MXL Receivers.


[RFC-2119]: https://tools.ietf.org/html/rfc2119 "Key words for use in RFCs"
[MXL]: https://tech.ebu.ch/dmf/mxl
[IS-04]: https://specs.amwa.tv/is-04/
[IS-05]: https://specs.amwa.tv/is-05/
[BCP-004-01]: https://specs.amwa.tv/bcp-004-01/ "AMWA BCP-004-01 NMOS Receiver Capabilities"
[NMOS formats parameter register]:  https://specs.amwa.tv/nmos-parameter-registers/branches/main/formats/ "NMOS Formats"
[NMOS media types parameter register]:  https://specs.amwa.tv/nmos-parameter-registers/branches/main/media-types/ "NMOS Media Types"
