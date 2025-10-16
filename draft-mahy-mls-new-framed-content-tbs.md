---
title: "A more efficient FramedContentTBS structure in Messsaging Layer Security (MLS)"
abbrev: "New MLS FramedContentTBS"
category: info

docname: draft-mahy-mls-new-framed-content-tbs-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Messaging Layer Security"
keyword:
 - GroupContext hash
 - FramedContent
 - FramedContentTBS
 - efficient signing
 - deterministic signing
venue:
  group: "Messaging Layer Security"
  type: "Working Group"
  mail: "mls@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/mls/"
  github: "rohanmahy/mls-new-framed-content-tbs"
  latest: "https://rohanmahy.github.io/mls-new-framed-content-tbs/draft-mahy-mls-new-framed-content-tbs.html"

author:
 -
    fullname: Rohan Mahy
    organization:
    email: rohan.ietf@gmail.com

normative:

informative:

...

--- abstract

Most MLS signatures are signed over the relatively large GroupContext structure.
This document defines a way to safely sign using a pre-hashed version of the GroupContext structure for better efficiency.

--- middle

# Introduction

In MLS {{!RFC9420}} all in-member application, commit, and proposal messages (whether sent via a PublicMessage, PrivateMessage or SemiPrivateMessage {{?I-D.mahy-mls-semiprivatemessage}}); and all external commits contain a signature of the `FramedContentTBS` structure.
This structure contains the full GroupContext of the group, which can store a large amount of group state.
(For example, in the MIMI protocol {{?I-D.ietf-mimi-protocol}} it contains the participant list for the group, which could contain thousands of URIs.)

The GroupContext only changes once per epoch, therefore it is an excellent candidate to pre-hash and cache for efficiency reasons.

This document defines an a replacement for the `FramedContentTBS` structure that uses a pre-hashed GroupContext, and an MLS extension for safely negotiating the use of the new struct.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Mechanism

This document defines the `new_framed_content_tbs` extension.
When present in a `LeafNode.capabilities.extensions` list, it indicates that the client supports this extension.
When present in the `required_capabilities.extension_types` list in `GroupContext.extensions` it indicates that every member of the group MUST use the new `FramedContentTBS` structure.

The `FramedContentTBS` structure is replaced with the new structure below. The only change is that the GroupContext is replaced with a hash (from the group's current MLS cipher suite) of the GroupContext.

~~~ tls
context_hash = RefHash(GroupContext);

struct {
    ProtocolVersion version = mls10;
    WireFormat wire_format;
    FramedContent content;
    select (FramedContentTBS.content.sender.sender_type) {
        case member:
        case new_member_commit:
            opaque context_hash<V>;
        case external:
        case new_member_proposal:
            struct{};
    };
} FramedContentTBS;
~~~

# Security Considerations

TODO Security


# IANA Considerations

This document requests the addition of a new MLS Extension Type value under the heading of "Messaging Layer Security".

RFC EDITOR: Please replace XXXX throughout with the RFC number assigned to this document.

The `new_framed_content_tbs` MLS Extension Type is used inside LeafNode and GroupContext objects.
In a LeafNode it indicates support the the extension.
In the `required_capabilities` of the GroupContext it indicates the extension is being used.

Value: 0x000b (suggested)

Name: new_framed_content_tbs

Message(s): GC,LN: This extension may appear in GroupContext and LeafNode objects

Recommended: Y

Reference: RFC XXXX

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
