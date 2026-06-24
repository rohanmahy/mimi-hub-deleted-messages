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

The More Instant Messaging Interoperability (MIMI) Protocol defines a way to signal potentially abusive messages to a provider, but provides no way for the provider to signal that an abusive message sent in a room should no longer be shown.
This document defines a mechanism for providers to signal this to in-room messages.


--- middle

# Introduction

The MIMI Protocol {{!I-D.ietf-mimi-protocol}} includes a mechanism to report allegedly abusive messages (defined in {{Section 5.9 of !I-D.ietf-mimi-protocol}}).
The plaintext of allegedly abusive messages is forwarded to the reporting service of the Hub provider for human and/or automated analysis.
This mechanism takes advantage of the franking mechanism (defined in {{Section 3.4.1 of !I-D.ietf-mimi-protocol}}) to verify that any allegedly abusive messages were actually sent by the indicated party.

If a reported message is found to be abusive, the reporting service could potentially take advantage of several remedies.
For example, provider could ban or remove the sender from a specific room or rooms by sending external proposals.
The provider could send out-of-band messages (ex: an email, or an instant message in a dedicated room that purpose) the administrator or moderator of the room in which the abuse was sent, and expect them to .
In some cases, the abusive content may be illegal and/or sufficiently disturbing or disruptive (for example hate speech, severe harassment, non-consensual sexual material, or sexual material involving children), that the content should probably be automatically removed from view by the participants in the room.
In many such cases, the provider even has a legal duty to act.

A hub provider may also discover that a specific participant has violated its terms of service generally, in a manner that causes their content to be a danger to other participants.
For example, an account which is fraudulently impersonating another user can be removed using an external remove proposal, but removing their content messages requires another mechanism.

This specification describes two new application components to signal messages to be deleted.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Mechanism

After an `AbuseReport` is sent (see {{Section 5.9 of !I-D.ietf-mimi-protocol}}), the hub provider determines if the allegedly abusive message(s) are abusive or not according to the room policy (which is within the hub's policy envelope).

If the abusive message(s) were MIMI content {{!I-D.ietf-mimi-content}}, the hub can inform clients in the room to mark the abusive or other unauthorized messages as deleted.
The hub could also delete messages for other reasons, such as upon discovering that messages were sent while an account was compromised, or that the sender of the messages opened an account under false pretenses (ex: impersonating the identity of another).

## Removing specific messages by message ID

The hub signals that messages should be deleted by sending one or more AppEphemeral proposal containing a `hub_deleted_messages` with a list of MIMI Content message IDs to delete.
Each proposal contains messages with the same deletion timestamp and (optional) reason code.
More than one AppEphemeral proposal with the `hub_deleted_messages` component type can be present in the same Commit.

Members of the MLS group receiving this proposal, verify that the proposal signature is valid and corresponds to a user in the participant list with a role containing the `canDeleteOtherMessage` for any type of message, or the `canDeleteOtherReaction` if the messages within `hub_deleted_component.delete_messages` are all reactions.

~~~ tls
uint8[32] MessageId;

struct {
    uint64 hub_deleted_timestamp;
    IdentityUri remover_uri;
    optional<AbuseType> reason_code;
    MessageId deleted_messages<V>;
    /* SignWithLabel(., "HubDeletedComponentTBS", */
    /*                   HubDeletedComponentTBS ) */
    opaque signature<V>;
} HubDeletedComponent;

struct {
    uint64 hub_deleted_timestamp;
    IdentityUri remover_uri;
    optional<AbuseType> reason_code;
    MessageId deleted_messages<V>;
} HubDeletedComponentTBS;

HubDeletedComponent hub_deleted_component;
~~~

## Removing ranges of messages sent by a specific participant




Deletes all messages from the `starting_timestamp` onward (or from all time) sent by the `sender_uri`.


~~~ tls
struct {
    uint64 hub_deleted_timestamp;
    IdentityUri remover_uri;
    optional<AbuseType> reason_code;
    IdentityUri sender_uri;
    optional<uint64> starting_timestamp;
    /* SignWithLabel(., "HubDeletedRangeComponentTBS", */
    /*                   HubDeletedRangeComponentTBS ) */
    opaque signature<V>;
} HubDeletedRangeComponent;

struct {
    uint64 hub_deleted_timestamp;
    IdentityUri remover_uri;
    optional<AbuseType> reason_code;
    IdentityUri sender_uri;
    optional<uint64> starting_timestamp;
} HubDeletedRangeComponentTBS;
~~~


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
