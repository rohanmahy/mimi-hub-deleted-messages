---
title: "More Instance Messaging Interoperability (MIMI): Signaling Deletion of MIMI Content Messages by the Hub Provider"
abbrev: "MIMI Hub-Deleted Messages"
category: info

docname: draft-mahy-mimi-hub-deleted-messages-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "More Instant Messaging Interoperability"
keyword:
 - MIMI content
 - deleting messages
 - deleting abusive messages
 - deleting unauthorized messages
 - hub-deleted
venue:
  group: "More Instant Messaging Interoperability"
  type: "Working Group"
  mail: "mimi@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/mimi/"
  github: "rohanmahy/mimi-hub-deleted-messages"
  latest: "https://rohanmahy.github.io/mimi-hub-deleted-messages/draft-mahy-mimi-hub-deleted-messages.html"

author:
 -
    fullname: Rohan Mahy
    email: rohan.ietf@gmail.com

normative:

informative:

...

--- abstract

TODO Abstract


--- middle

# Introduction

TODO Introduction


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Mechanism

After an `AbuseReport` is sent (see {{Section 5.9 of !I-D.ietf-mimi-protocol}}), the hub provider determines if the allegedly abusive message(s) are abusive or not according to the room policy (which is within the hub's policy envelope).

If the abusive message(s) were MIMI content {{!I-D.ietf-mimi-content}}, the hub can inform clients in the room to mark the abusive or other unauthorized messages as deleted.
The hub could also delete messages for other reasons, such as upon discovering that messages were sent while an account was compromised, or that the sender of the messages opened an account under false pretenses.

The hub signals that messages should be deleted by sending an AppEphemeral proposal containing a `hub_deleted_component` with a list of MIMI Content message IDs to delete.

Members of the MLS group receiving this proposal, verify that the proposal signature is valid and corresponds to a user in the participant list with a role containing the `canDeleteOtherMessage` for any type of message, or the `canDeleteOtherReaction` if the messages with `hub_deleted_component.delete_messages` are all reactions.

~~~ tls
uint8[32] MessageId;

struct {
    uint64 hub_deleted_timestamp;
    IdentityUri remover_uri;
    MessageId deleted_messages<V>;
    /* SignWithLabel(., "HubDeletedComponentTBS", */
    /*                   HubDeletedComponentTBS ) */
    opaque signature<V>;
} HubDeletedComponent

struct {
    uint64 hub_deleted_timestamp;
    IdentityUri remover_uri;
    MessageId deleted_messages<V>;
} HubDeletedComponentTBS

HubDeletedComponent hub_deleted_component;
~~~


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
