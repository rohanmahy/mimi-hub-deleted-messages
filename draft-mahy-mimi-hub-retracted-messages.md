---
title: "More Instance Messaging Interoperability (MIMI): Retracting MIMI Content Messages by the Hub Provider"
abbrev: "MIMI Hub-Retracted Messages"
category: info

docname: draft-mahy-mimi-hub-retracted-messages-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "More Instant Messaging Interoperability"
keyword:
 - MIMI content
 - retracting abusive messages
 - retracting unauthorized messages
 - deleting messages
 - deleting abusive messages
 - deleting unauthorized messages
 - hub-deleted
 - hub-retracted
venue:
  group: "More Instant Messaging Interoperability"
  type: "Working Group"
  mail: "mimi@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/mimi/"
  github: "rohanmahy/mimi-hub-deleted-messages"
  latest: "https://rohanmahy.github.io/mimi-hub-deleted-messages/draft-mahy-mimi-hub-retracted-messages.html"

author:
 -
    fullname: Rohan Mahy
    email: rohan.ietf@gmail.com

normative:

informative:

...

--- abstract

The More Instant Messaging Interoperability (MIMI) Protocol defines a way to signal potentially abusive messages to a provider, but provides no way for the provider to signal that an abusive message sent in a room should be retracted.
This document defines mechanisms for providers to signal this to in-room participants.


--- middle

# Introduction

The MIMI Protocol {{!I-D.ietf-mimi-protocol}} includes a mechanism to report allegedly abusive messages (defined in {{Section 5.9 of !I-D.ietf-mimi-protocol}}).
The plaintext of allegedly abusive messages is forwarded to the reporting service of the Hub provider for human and/or automated analysis.
This mechanism takes advantage of the franking mechanism (defined in {{Section 3.4.1 of !I-D.ietf-mimi-protocol}}) to verify that any allegedly abusive messages were actually sent by the indicated party.

If a reported message is found to be abusive, the reporting service could potentially take advantage of several remedies.
For example, the provider could ban or remove the sender from a specific room or rooms by sending external proposals.
The provider could send an out-of-band message (ex: an email, or an instant message in a dedicated room that purpose) to the administrators or moderators of the room in which the abuse was sent, and expect them to react in an appropriate manner.
In some cases, the abusive content may be illegal and/or sufficiently disturbing or disruptive (ex: hate speech, severe harassment, non-consensual sexual material, or sexual material involving children), that the content needs to be automatically retracted and removed from view by the participants in the room.
In many such cases, the provider even has a legal duty to act.

A hub provider may also discover that a specific participant has violated its terms of service generally, in a manner that causes their content to be a danger to other participants.
For example, an account which is fraudulently impersonating another user can be removed using an external remove proposal, but removing their content messages requires another mechanism.

This specification describes two new application components to signal messages to be deleted.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Mechanism

After an `AbuseReport` is sent (see {{Section 5.9 of !I-D.ietf-mimi-protocol}}), the hub provider determines if allegedly abusive messages are abusive or not according to the plaintext contents of the message, possibly the context, and the room policy (which is within the hub's policy envelope).

If the abusive message(s) were MIMI content {{!I-D.ietf-mimi-content}}, the hub can inform clients in the room to mark the abusive or other unauthorized messages as retracted.
The hub could also retract messages for other reasons, such as upon discovering that messages were sent while an account was compromised, or that the sender of the messages opened an account under false pretenses (ex: impersonating the identity of another).

## Removing specific messages by message ID

The hub signals that messages should be retracted by sending one or more AppEphemeral proposal containing a `hub_retracted_messages` component with a list of MIMI Content message IDs to retract/delete.
Each proposal contains messages with the same retracted timestamp and (optional) reason code.
More than one AppEphemeral proposal with the `hub_retracted_messages` component type can be present in the same Commit (for example to delete different messages with different `reason_code`s).

Members of the MLS group receiving this proposal, verify that the proposal signature is valid, and the proposal sender corresponds to a user in the participant list with a role containing the `canDeleteOtherMessage` for any type of message, or the `canDeleteOtherReaction` if the messages within `hub_retracted_component.retracted_messages` are all reactions.

~~~ tls
uint8[32] MessageId;

struct {
    uint64 hub_retracted_timestamp;
    IdentityUri remover_uri;
    optional<AbuseType> reason_code;
    MessageId retracted_messages<V>;
    /* SignWithLabel(., "HubRetractedComponentTBS", */
    /*                   HubRetractedComponentTBS ) */
    opaque signature<V>;
} HubRetractedComponent;

struct {
    uint64 hub_retracted_timestamp;
    IdentityUri remover_uri;
    optional<AbuseType> reason_code;
    MessageId retracted_messages<V>;
} HubRetractedComponentTBS;

HubRetractedComponent hub_retracted_component;
~~~

## Removing ranges of messages sent by a specific participant

In the case where the provider discovers a client trying to commit fraud or deception, the provider may wish to take immediate action without waiting for every message sent to be reported.
For example, a user who is impersonating a government economy or health spokesperson, or a recruiter at a well-known firm could send messages causing panic or phishing, but ordinary viewers may have no reason to report the messages, believing them to be from a legitimate source.

Instead it is useful to label all messages from the fraudulent source as suspect and retract them, possibly starting at a specific time (for example, if an account was compromised).

The hub signals that a range of messages sent by a particular sender should be retracted by sending one or more AppEphemeral proposal.
Each proposal contains a `reason_code`; and a `hub_retracted_range` component with an `abusive_sender_uri`, and optionally a `starting_timestamp` indicating when the sender began to be untrustworthy.
More than one AppEphemeral proposal with the `hub_retracted_range` component type can be present in the same Commit.
A single Commit MUST NOT contain `hub_retracted_range` proposals for the same `abusive_sender_uri`.

Members of the MLS group receiving this proposal, verify that the proposal signature is valid, and the proposal sender corresponds to a user in the participant list with a role containing the `canDeleteOtherMessage` capability.
Then they retract all messages (including reactions, edits, deletes, and replies) sent by the `abusive_sender_uri`, starting from and including the `starting_timestamp` if present, or since the formation of the room if `starting_timestamp` is absent.

~~~ tls
struct {
    uint64 hub_retracted_timestamp;
    IdentityUri remover_uri;
    optional<AbuseType> reason_code;
    IdentityUri abusive_sender_uri;
    optional<uint64> starting_timestamp;
    /* SignWithLabel(., "HubRetractedRangeComponentTBS", */
    /*                   HubRetractedRangeComponentTBS ) */
    opaque signature<V>;
} HubRetractedRangeComponent;

struct {
    uint64 hub_retracted_timestamp;
    IdentityUri remover_uri;
    optional<AbuseType> reason_code;
    IdentityUri abusive_sender_uri;
    optional<uint64> starting_timestamp;
} HubRetractedRangeComponentTBS;
~~~


# Security Considerations

The security of this system depends on including only systems authorized to send external proposals in the `external_senders` MLS extension in the GroupContext; and correctly setting the roles and capabilities in {{!I-D.ietf-mimi-room-policy}} for the entity and role that sends external proposals with the components described in this specification.

In the case of `hub_retracted_messages`, the security of retraction also relies on the security of the franking system in MIMI protocol.
It also assumes that the hub provider can come up with the correct message ID (which is straightforward when the content is in the MIMI content format).


# IANA Considerations

RFC EDITOR: Please replace XXXX throughout with the RFC number assigned to this document.

This document registers the following two MLS application components per
{{Section 7.6 of !I-D.ietf-mls-extensions}}.

## hub_retracted_messages app component

- Value: TBD1 (suggested value 0x0050)
- Name: hub_retracted_messages
- Where: AE
- Recommended: Y
- Reference: RFCXXXX

## hub_retracted_messages app component

- Value: TBD2 (suggested value 0x0051)
- Name: hub_retracted_range
- Where: AE
- Recommended: Y
- Reference: RFCXXXX



--- back

