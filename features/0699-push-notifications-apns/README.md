# Aries RFC 0699: Push Notifications apns Protocol 1.0

- Authors: [Timo Glastra](mailto:timo@animo.id) (Animo Solutions) & [Berend Sliedrecht](mailto:berend@animo.id) (Animo Solutions)
- Status: [PROPOSED](/README.md#proposed)
- Since: 2021-10-07
- Status Note: Initial version
- Start Date: 2021-05-05
- Tags: [feature](/tags.md#feature), [protocol](/tags.md#protocol)

> Note: This protocol is currently written to support native push notifications for iOS via [Apple Push Notification Service](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/sending_notification_requests_to_apns/).
> For the implementation for Android (using fcm), please refer to [0734: Push Notifications fcm](../0734-push-notifications-fcm/README.md)

## Summary

A protocol to coordinate a push notification configuration between two agents.

## Motivation

This protocol would give an agent enough information to send push notifications about specific events to an iOS device. This would be of great benefit for mobile wallets, as a holder can be notified when new messages are pending at the mediator. Mobile applications, such as wallets, are often killed and can not receive messages from the mediator anymore. Push notifications would resolve this problem.

## Tutorial

### Name and Version

URI: `https://didcomm.org/push-notifications-apns/1.0`

Protocol Identifier: `push-notifications-apns`

Version: `1.0`

Since apns only supports iOS, no `-ios` or `-android` is required as it is implicit.

### Key Concepts

When an agent would like to receive push notifications at record event changes, e.g. incoming credential offer, incoming connection request, etc., the agent could initiate the protocol by sending a message to the other agent.

This protocol only defines how an agent would get the token which is necessary for push notifications.

Each platform is has its own protocol so that we can easily use [0031: Discover Features 1.0](https://github.com/hyperledger/aries-rfcs/blob/main/features/0031-discover-features/README.md) and [0557: Discover Features 2.X](https://github.com/hyperledger/aries-rfcs/blob/main/features/0557-discover-features-v2/README.md) to see which specific services are supported by the other agent.

### Roles

**notification-sender**

**notification-receiver**

The **notification-sender** is an agent who will send the **notification-receiver** notifications. The **notification-receiver** can get and set their push notification configuration at the **notification-sender**.

### Services

This RFC focusses on configuring the data necessary for pushing notifications to iOS, via [apns](https://developer.apple.com/notifications/).

In order to implement this protocol, the [set-device-info](#set-device-info) and [get-device-info](#get-device-info) messages MUST be implemented by the **notification-sender** and [device-info](#device-info) message MUST be implemented by the **notification-receiver**.

#### Supported Services

The protocol currently supports the following push notification services

- [apns](https://developer.apple.com/notifications/) for iOS devices

### Messages

When a notification-receiver wants to receive push notifications from the notification-sender, the notification-receiver has to send the following message:

#### Set Device Info

Message to set the device info using the native iOS device token for push notifications.

```json
{
  "@type": "https://didcomm.org/push-notifications-apns/1.0/set-device-info",
  "@id": "<UUID>",
  "device_token": "<DEVICE_TOKEN>"
}
```

Description of the fields:

- `device_token` -- The token that is required by the notification provider (string, null)

It is important to note that the set device info message can be used to set, update and remove the device info. To set, and update, these values the normal messages as stated above can be used. To remove yourself from receiving push notifications, you can send the same message where all values MUST be `null`. If either value is `null` a `problem-report` MAY be sent back with `missing-value`.

#### Get Device Info

When a notification-receiver wants to get their push-notification configuration, they can send the following message:

```json
{
  "@type": "https://didcomm.org/push-notifications-apns/1.0/get-device-info",
  "@id": "<UUID>"
}
```

#### Device Info

Response to the get device info:

```json
{
  "@type": "https://didcomm.org/push-notifications-apns/1.0/device-info",
  "device_token": "<DEVICE_TOKEN>",
  "~thread": {
    "thid": "<GET_DEVICE_INFO_UUID>"
  }
}
```

This message can be used by the notification-receiver to receive their device info, e.g. `device_token`. If the notification-sender does not have this field for that connection, a `problem-report` MAY be used as a response with `not-registered-for-push-notifications`.

#### Adopted messages

In addition, the [`ack`](https://github.com/hyperledger/aries-rfcs/blob/08653f21a489bf4717b54e4d7fd2d0bdfe6b4d1a/features/0015-acks/README.md) message is adopted into the protocol for confirmation by the notification-sender. The ack message SHOULD be sent in response to any of the set-device-info messages.

### Sending Push Notifications

When an agent wants to send a push notification to another agent, the payload of the push notifications MUST include the `@type` property, and COULD include the `message_tag` property, to indicate the message is sent by the notification-sender. Guidelines on notification messages are not defined.

```json
{
  "@type": "https://didcomm.org/push-notifications-apns",
  "message_tag": "<MESSAGE_TAG>",
  ...
}
```

Description of the fields:

- `@type` -- Indicator of what kind of notification it is. (This could help the notification-receiver with parsing if a notification comes from another agent, for example)
- `message_tag` -- Optional field to connect the push notification to a DIDcomm message. As defined in [0334: jwe-envelope](https://github.com/hyperledger/aries-rfcs/tree/main/features/0334-jwe-envelope) or [0019: encryption-envelope](https://github.com/hyperledger/aries-rfcs/tree/main/features/0019-encryption-envelope).

## Drawbacks

Each service requires a considerable amount of domain knowledge. The RFC can be extended with new services over time.

The `@type` property in the push notification payload currently doesn't indicate which agent the push notification came from. In e.g. the instance of using multiple mediators, this means the notification-receiver does not know which mediator to retrieve the message from.

## Prior art

- This RFC is based on the implementation of the [`AddDeviceInfoMessage`](https://github.com/hyperledger/aries-framework-dotnet/blob/9bc6346a21da263083bbac8dd8227cc941c95ea9/src/Hyperledger.Aries.Routing/AddDeviceInfoMessage.cs) in Aries Framework .NET

## Unresolved questions

None

## Implementations

The following lists the implementations (if any) of this RFC. Please do a pull request to add your implementation. If the implementation is open source, include a link to the repo or to the implementation within the repo. Please be consistent in the "Name" field so that a mechanical processing of the RFCs can generate a list of all RFCs supported by an Aries implementation.

| Name / Link | Implementation Notes |
| ----------- | -------------------- |
|             |                      |