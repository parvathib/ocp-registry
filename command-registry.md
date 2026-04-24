
# OCP Command Registry

This document contains an informative list of Command Codes defined by OCP specifications. This document serves as only a reference with short descriptions. The source of the normative references of the command codes and their definitions are in the OCP specifications themselves.


## Command Code Registry

| Command Code | Command Name             | Defining Specification                             | Description                                                        |
| :----------- | :----------------------- | :------------------------------------------------- | :----------------------------------------------------------------  |
| 0x00         | Reserved                 | -                                                  | Reserved for future use                                            |
| 0x01-0x20    | Caliptra VDM Commands    | [Caliptra VDM Commands](https://github.com/chipsalliance/caliptra-mcu-sw/blob/main/docs/src/external_mctp_vdm_cmds.md) (OCP Vendor ID: 42623) | Caliptra-defined commands using SPDM Vendor Defined Messages with OCP Vendor ID |
| 0x21-0xFF    | Reserved                 | -                                                  | Reserved for future assignment                                     |

## Command Assignment Process

New OCP specifications that require command codes **MUST**:

1. Request command code assignment from the OCP Security Working Group
2. Provide a command name and brief description
3. Reference the defining specification
4. Update this registry upon approval

## Reserved Command Range Governance

### Caliptra VDM Commands (0x01-0x20)

The command range 0x01–0x20 is reserved in this OCP registry and assigned to the Caliptra Working Group. From a governance perspective, these are **not** OCP-defined commands. OCP only reserves the command range in its registry and assigns it to Caliptra. The Caliptra Working Group retains full ownership of the definition, specification, and implementation of all commands within this range.

Accordingly, commands in this range are treated as **Caliptra-defined commands using SPDM Vendor Defined Messages with the OCP-assigned Vendor ID (42623)**, ensuring clear ownership and normative authority while leveraging the OCP registry as a shared namespace.

- **Assigned to:** Caliptra Working Group
- **Vendor ID:** OCP (42623)
- **Normative Authority:** Caliptra Working Group
- **Reference:** See [Caliptra VDM Commands](https://github.com/chipsalliance/caliptra-mcu-sw/blob/main/docs/src/external_mctp_vdm_cmds.md) for the full list of command definitions


## Error Code Registry

| Error Code | Error Name                    | Description                                           |
| :--------- | :---------------------------- | :---------------------------------------------------- |
| 0x00       | SUCCESS                       | Command completed successfully                        |
| 0x01       | GENERAL_ERROR                 | Unspecified error occurred                            |
| 0x02       | INVALID_PARAMETER             | One or more command parameters are invalid            |
| 0x03       | INVALID_LENGTH                | Specified length field is outside valid range         | 
| 0x04       | INVALID_IDENTIFIER            | Specified identifier not found or not available       |
| 0x05       | OPERATION_FAILED              | Requested operation could not be completed            |
| 0x06       | INSUFFICIENT_RESOURCES        | Insufficient memory or processing resources           |
| 0x07       | UNSUPPORTED_OPERATION         | Requested operation not supported by implementation   |
| 0x08       | DEVICE_NOT_READY              | Device not in appropriate state for command           |
| 0x09       | INVALID_COMMAND_VERSION       | Command version not supported                         |
| 0x0A       | INVALID_PAYLOAD_SIZE          | Command payload size exceeds limits                   |
| 0x0B       | TIMEOUT                       | Operation timed out                                   |
| 0x0C       | ACCESS_DENIED                 | Insufficient privileges for requested operation       |
| 0x0D       | RESOURCE_UNAVAILABLE          | Required resource is not accessible                   |
| 0x0E       | POLICY_VIOLATION              | Command violates device or system policy              |
| 0x0F       | INVALID_STATE                 | Device or resource in invalid state for operation     |
| 0x10-0x7F  | Reserved (Generic)            | Reserved for future generic error assignments         |
| 0x80-0xBF  | Reserved (Security)           | Reserved for security-specific error codes            |
| 0xC0-0xFF  | Reserved (Project-Specific)   | Reserved for project-specific error codes             |

### Error Code Categories

#### Generic Errors (0x00-0x7F)
These error codes can be used across all OCP projects and are not specific to any particular domain. They cover common failure conditions that apply to any command-based protocol.

#### Security-Specific Errors (0x80-0xBF)
Reserved for security-specific error conditions such as:
- Cryptographic operation failures
- Certificate validation errors
- Authentication/authorization failures
- Security policy violations

#### Project-Specific Errors (0xC0-0xFF)
Reserved for errors specific to particular OCP projects (e.g., networking, storage, compute).

### Error Assignment Process

New OCP specifications that require error codes **MUST**:

1. Request error code assignment from the appropriate OCP working group
2. Provide error name and detailed description
3. Reference the defining specification
4. Specify the intended category (Generic, Security, Project-Specific)
5. Update this registry upon approval

### Error Handling Best Practices

#### For OCP Command Implementations
- **MUST** use appropriate OCP error codes for all failure conditions
- **SHOULD** use generic error codes when possible for broader compatibility
- **SHOULD** provide meaningful error details when possible

## Transport Bindings

### Transport Binding Requirements

Each transport protocol binding **MUST** define:

1. How to frame OCP command requests and responses
2. How to convey OCP success responses with full payload
3. How to convey OCP error codes, using either the transport's native error mechanisms or as part of the response.

### SPDM Transport Binding

#### Command Framing
- **Request**: Use SPDM VENDOR_DEFINED_REQUEST format
- **Response**: Use either SPDM VENDOR_DEFINED_RESPONSE (success) or SPDM ERROR (failure)

#### Standard Fields
- StandardID field **MUST** contain 4 (IANA)
- VendorID field **MUST** contain 42623 (OCP)

#### Success Response Structure
- Use SPDM VENDOR_DEFINED_RESPONSE format
- First byte of VendorDefinedReqPayload/VendorDefinedRespPayload **MUST** contain the CommandVersion
- Second byte of VendorDefinedReqPayload/VendorDefinedRespPayload **MUST** contain the CommandCode
- OCP command request/response structures form the remainder of the payload

#### Error Response Structure
- Use SPDM ERROR message format
- Set Param1 to 0xFF ("Vendor or Standards-Defined") and Param2 to 0x4 ("IANA")
- Use the ExtendedErrorData format for a vendor or other standards-defined ERROR response message
- Within ExtendedErrorData, set Len to 4 and VendorID to 42623 (OCP)
- Within OpaqueErrorData, set the first byte to the length of the error code. Shall be 1.
- Within OpaqueErrorData, set the second byte to the OCP error code.

#### Protocol-Level Errors
- Use standard SPDM ERROR message format
- Set appropriate SPDM ErrorCode (e.g., VendorDefined, InvalidRequest)
- Follow SPDM error response procedures

### Future Transport Bindings

Other transport protocols **MAY** define their own bindings provided they:

- Maintain semantic equivalence of request and response structures
- Preserve all required fields and their meanings
- Implement appropriate error reporting using transport-native mechanisms
- Document any transport-specific adaptations

Future transport bindings **MAY** use different approaches such as:
- **Native Error Fields**: Include OCP error code in response structure
- **Status Responses**: Return either full response or error-only response
- **Exception Mechanisms**: Use transport-native exception handling

### Transport-Agnostic Error Handling

OCP error codes are defined independently of transport protocols. Each transport binding **MUST** specify how to convey OCP error codes using the transport's native error mechanisms.
