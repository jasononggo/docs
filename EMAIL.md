# Enhance email security

## Set up SPF records to prevent email spoofing.

Reference: [Help prevent email spoofing with SPF records](https://support.google.com/a/answer/33786?hl=en&ref_topic=7556597)

## Set up DKIM to prevent email spoofing

Reference: [Set up DKIM to prevent email spoofing](https://support.google.com/a/answer/174124?hl=en)

## Set up DMARC policy to manage suspicious emails

### How DMARC works
DMARC helps email senders and receivers verify incoming messages by authenticating the sender's domain. **DMARC also defines the action to take on suspicious incoming messages.**

Put into practice, if you relay email with DMARC policy reject 100% the recipient will not know.

Reference: [Enhance security for forged spam (DMARC)](https://support.google.com/a/answer/2466580?hl=en&ref_topic=2759254&visit_id=637246565085389670-3422249965&rd=1)

Add a TXT record to the domain host with DMARC policy quarantine 100% or DMARC policy reject 100%. It is recommended to use the former. Do note, in case of relaying email, the sender and the recipient do not know if the email has been rejected.

```
# DMARC policy quarantine 100%
# quarantine: Mark the messages as spam and move to recipient's spam folder.
v=DMARC1; p=quarantine; pct=100; rua=mailto:dmarc@companydomain.com

# DMARC policy reject 100%
# reject: Tell receiving servers to reject the message. When this happens, the receiving server should send a bounce to the sending server.
# Do note, in case of relaying email, the sender and the recipient do not know if the email has been rejected.
v=DMARC1; p=reject; pct=100; rua=mailto:dmarc@companydomain.com
```

### Testing

Relay email from companydomain.com to a test email using gmail.com `Google`, outlook.com `Microsoft`, rocketmail.com `Yahoo`.
Check the raw message of the each relayed email received by gmail.com, outlook.com, rocketmail.com, and make sure to find `dmarc=pass` in the raw message.

```
...
ARC-Authentication-Results:
...
  dmarc=pass
...
```

## Increase email security with (optional) MTA-STS and TLS reporting

Reference: [Increase email security with MTA-STS and TLS reporting](https://support.google.com/a/topic/9261406?hl=en&ref_topic=7556597)

**TLS reporting**

Before your domain enforces MTA-STS encryption and authentication, set your policy to testing mode. Check the daily reports to identify and fix any connection issues with your domain. Then, change your policy to enforce mode.

**MTA-STS**

Like all mail providers, Gmail uses Simple Mail Transfer Protocol (SMTP) to send and receive messages. SMTP alone does not provide security, and **many SMTP servers don’t have added security to prevent malicious attacks**. Thus being said, you should analyze the TLS reporting before taking decision.

# Set up SMTP relay service with the Sender Rewriting Scheme

Reference: [SMTP Relay: Route outgoing non-Gmail message through Google[(https://support.google.com/a/answer/2956491?hl=en)

From the technical point of view, you have 2 options to automate forwarding email:
1. Use SMTP server to forward the email and end up with for example `FROM: catchall@companydomain.com` header on every forwaded email

Business-wise, you should not use SMTP server to forward the email. Do note, most of non-technical user rely on `FROM:` header to identify who send the email and possibly query the email for future use. For example, automated forwading email with `FROM: catchall@companydomain.com` would prove harder to query mail sent by a specific individual.

2. Use SMTP relay service with the sender rewriting scheme to relay the email and end up with for example `FROM: sender@gmail.com via companydomain.com`

> The sender rewriting scheme provides for recovering the original envelope address, so that if a bounce does arrive, it can be forwarded along the reverse path—with an empty envelope sender this time. While there are other workarounds, SRS is a fairly general one.

Do note, if you use G Suite SMTP relay service and relay email from domain you do not own (such as yahoo.com), In the Allowed senders section, select the users who are allowed to send messages through the SMTP relay service: Any addresses (not recommended). If you did not, your user with @companydomain.com email will not receive the relayed email except @companydomain.com / G Suite SMTP relay service will not relay the email.
