---
title: 'Message acceptance'
prev: basics-addresses.md
up: basics-workflow.md
next: basics-alteration.md
---

Message acceptance
==================

Just after when Sympa's
[`sympa_msg.pl`](/gpldoc/man/sympa_msg.8.html) daemon fetches an incoming
message from the incoming spool ([``$SPOOLDIR``](../layout.md#spooldir)`/msg`
directory), it executes some sorts of inspection on the message,
and will ignore the message that does not meet certain requirements.

Additionally, list owners may choose customized policies based on
[authorization scenario](basics-scenarios.md) so that they can control
message acceptance.

Checkpoints built in Sympa
--------------------------

Some of requirements by Sympa are mandatory and the others are optional.
The messages ignored according to any of requirements will be kept in
the `bad` directory in incoming spool and will no longer be processed.

### Existence of mandatory header fields

  * If the message did not have `Messsage-ID:` header field, it will be
    ignored.

  * If the message did not have available sender address, it will be
    ignored.
    The "sender address" is an originator address of the message suggested
    by message header fields.
    By default, it is taken from `From:` header field (See also
    "[`sender_headers`](/gpldoc/man/sympa.conf.5.html#sender_headers)"
    parameter).

### Loop prevention

  * Some sorts of automated messages may be ignored.
    This feature is enabled by default (See
    [`reject_mail_from_automates_feature`](/gpldoc/man/list_config.5.html#reject_mail_from_automates_feature)).

      - If sender address (see above) includes any of well-known mailbox
        names, it may be ignored (See
        [`loop_prevention_regex`](/gpldoc/man/list_config.5.html#loop_prevention_regex)).

      - If message header contains a field signifying automated message,
        message will be ignored.
        In particular, `Auto-Submitted:` (RFC 3834),
        `Conntent-Identifier:` (nonstandard) and `X400-Conntent-Identifier:`
        (RFC 2156) are supported.

      - If incoming message has `X-Loop:` header field including the same address
        as target list, the message will be ignored.

    On the other hand, Sympa adds `Auto-Submitted:`
    field to the messages generated by mailing list system itself, and
    adds `X-Loop:` adds `X-Loop:` header field to the messages delivered by
    Sympa itself.

  * If already known message ID was found, the message will be ignored.
    [`sympa_msg.pl`](/gpldoc/man/sympa_msg.8.html) keeps message IDs of
    recently delivererd messages and detects duplication.
    This feature cannot be disabled.

See also "[Loop prevention](/gpldoc/man/sympa.conf.5.html#loop-prevention)".

### The other criteria

  * Sympa can ignore message that looks containing
    "[mail commands](../mail-commands.md)" (See
    "[`misaddressed_commands`](/gpldoc/man/sympa.conf.5.html#misaddressed_commands)" parameter).

  * Sympa can ignore message that excceeds specified message size (See
    "[`max_size`](/gpldoc/man/list_config.5.html#max_size)" parameter).

  * Optional Antivirus plug-in may ignore the message containing potentially
    insecure content (See "[Antivirus plug-in](../customize/antivirus.md)").

Check by authorization scenario
-------------------------------

The list owner can choose `send` authorization scenario to control
who can send to the list (See also
"[Authorization scenarios](basics-scenarios.md)" and
"[`send`](/gpldoc/man/list_config.5.html#send)" parameter).

This scenario may return one of following results:

  * `do_it`

    Accepting message.
    Afterwards, the message will be altered as neccesity (see
    "[Message alteration](basics-alterations.md)") and delivered to
    list members (see "[Message delivery](basics-delivery.md)").

  * `editor`

    Forwarding to moderators.
    The message will be forwarded to list moderator(s) so that they may
    re-post it to the list by theirselves.

  * `editorkey`

    Holding message for moderation.
    The message will be held in the moderation spool and wait for moderation
    by list moderator.
    Only the messages authorized by moderators will be delivered through
    the list.

  * `request_auth`

    Holding message for confirmation.
    The message will be held in the authorization spool and wait for
    confirmation by the sender.
    Only the messages authorized by sender will step forward to further
    processing such as delivery.

  * `reject`

    Denying message.
    The message will no longer be delivered to list members.
    Unless "`quiet`" modifier is specified, deliovery status notification
    will be sent back to the sender.

