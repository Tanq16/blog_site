---
title: How Telegram is a Secure Instant Messaging Platform
date: 2022-01-20 12:00:00 +0500
categories: [Computers Newbified]
tags: [productivity,telegram,privacy]
---

The following are sections of text from Telegram's FAQs and documentation. These have been hand selected by me to give details on the most important things relating to security and privacy. This does not contain much technical-speak but is more general in terms of concepts and meant to be a privacy discussion rather than a technical implementation.

>Read more about Telegram on their FAQs [here](https://telegram.org/faq).
{: .prompt-tip }

## Do you process data requests?

- Secret chats use end-to-end encryption, thanks to which we don't have any data to disclose.
- To protect the data that is not covered by end-to-end encryption, Telegram uses a distributed infrastructure. Cloud chat data is stored in multiple data centers around the globe that are controlled by different legal entities spread across different jurisdictions. The relevant decryption keys are split into parts and are never kept in the same place as the data they protect. As a result, several court orders from different jurisdictions are required to force us to give up any data. Thanks to this structure, we can ensure that no single government or block of like-minded countries can intrude on people's privacy and freedom of expression.
- Telegram can be forced to give up data only if an issue is grave and universal enough to pass the scrutiny of several different legal systems around the world.
- To this day, we have disclosed 0 bytes of user data to third parties, including governments.

## So how do you encrypt data?

- We support two layers of secure encryption. Server-client encryption is used in Cloud Chats (private and group chats), Secret Chats use an additional layer of client-client encryption. All data, regardless of type, is encrypted in the same way - be it text, media or files.
- Our encryption is based on 256-bit symmetric AES encryption, 2048-bit RSA encryption, and Diffie-Hellman secure key exchange.

## Why should I trust you?

- Telegram is open source, anyone can check our source code, protocol and API, see how everything works and make an informed decision. Telegram supports verifiable builds, which allow experts to independently verify that our code published on GitHub is the exact same code that is used to build the apps you download from App Store or Google Play.
- On top of that, Telegram's primary focus is not to bring a profit, so commercial interests will never interfere with our mission.

## How secure is Telegram?


Telegram is more secure than mass market messengers like WhatsApp and Line. We are based on the `MTProto` protocol, built upon time-tested algorithms to make security compatible with high-speed delivery and reliability on weak connections. We are continuously working with the community to improve the security of our protocol and clients.

## What if I'm more paranoid than your regular user?


We've got you covered. Telegram's special secret chats use end-to-end encryption, leave no trace on our servers, support self-destructing messages and don't allow forwarding. On top of this, secret chats are not part of the Telegram cloud and can only be accessed on their devices of origin.

## How do self-destructing messages work?

- The Self-Destruct Timer is available for all messages in Secret Chats and for media in private cloud chats.
- To set the timer, simply tap the clock icon (in the input field on iOS, top bar on Android), and then choose the desired time limit. The clock starts ticking the moment the message is displayed on the recipient's screen (gets two check marks). As soon as the time runs out, the message disappears from both devices.
- We will try to send a notification if a screenshot is taken.
- Please note that the timer in Secret Chats only applies to messages that were sent after the timer was set. It has no effect on earlier messages.

## Can I be certain that my conversation partner doesn't take a screenshot?

- Unfortunately, there is no bulletproof way of detecting screenshots on certain systems (most notably, some Android and Windows Phone devices).
- We will make every effort to alert you about screenshots taken in your Secret Chats, but it may still be possible to bypass such notifications and take screenshots silently.
- We advise to share sensitive information only with people you trust. After all, nobody can stop a person from taking a picture of their screen with a different device or an old school camera.

## Why not just make all chats 'secret'?

- All Telegram messages are always securely encrypted. Messages in Secret Chats use client- client encryption, while Cloud Chats use client-server/server-client encryption and are stored encrypted in the Telegram Cloud. This enables your cloud messages to be both secure and immediately accessible from any of your devices even if you lose your device altogether.
- The problem of restoring access to your chat history on a newly connected device (e.g. when you lose your phone) does not have an elegant solution in the end-to-end encryption paradigm. At the same time, reliable backups are an essential feature for any mass-market messenger.
- To solve the backup problem, some applications (like WhatsApp and Viber) allow decryptable backups that put their users' privacy at risk - even if they do not enable backups themselves. Other apps ignore the need for backups altogether and leave their users vulnerable to data loss.
- We opted for a third approach by offering two distinct types of chats. Telegram disables default system backups and provides all users with an integrated security-focused backup solution in the form of Cloud Chats.
- Meanwhile, the separate entity of Secret Chats gives you full control over the data you do not want to be stored.
- This allows Telegram to be widely adopted in broad circles, not just by activists and dissidents, so that the simple fact of using Telegram does not mark users as targets for heightened surveillance in certain countries.
- We are convinced that the separation of conversations into Cloud and Secret chats represents the most secure solution currently possible for a massively popular messaging application.

## Can I get Telegram's server- side code?

- All Telegram client apps are fully open source. We offer verifiable builds both for iOS and Android - this technology allows to independently verify that the application you download from the app stores was built using the exact same code that we publish.
- By contrast, publishing the server code doesn't provide security guarantees neither for Secret Chats nor for Cloud Chats. This is because - unlike with the client-side code - there's no way to verify that the same code is run on the servers.
- As for Secret Chats, you don't need the server-side code to check their integrity - the point of end-to-end encryption is that it must be solid regardless of how the servers function.
- The encryption and API used on Telegram's servers are fully documented and open for review by security experts.

## Telegram uses the camera or microphone in the background!

- Telegram can use the microphone in the background if you minimize the app when making a call, recording a video, or recording a voice/video message.
- Permission monitors on Samsung and Xiaomi can inadvertently flag and notify you that Telegram requested access to camera in the background. This happens when the app requests info about the camera - it isn't using the camera. Unfortunately it may look the same to the Samsung and Xiaomi permission monitors.
- Camera info is requested by the app when you tap on the attachment button, or start recording a video or a video message. If you do this and quickly close the app, the already initiated request may try to run asynchronously when the app is already in the background, or be sent when the system wakes up the app to show a notification about a new message. In any case, these requests are only for the camera info, the app never uses the camera itself in the background.
- Anyone can check Telegram's open source code and confirm that the app is not doing anything behind their back. We also offer reproducible builds that can help you prove that the version you downloaded from App Store or Google Play is built from the exact same source code we publish.

## Cloud Chats

- Telegram is a cloud service. We store messages, photos, videos and documents from your cloud chats on our servers so that you can access your data from any of your devices anytime without having to rely on third-party backups.
- All data is stored heavily encrypted and the encryption keys in each case are stored in several other data centers in different jurisdictions. This way local engineers or physical intruders cannot get access to user data.

## Secret Chats

- Secret chats use end-to-end encryption. This means that all data is encrypted with a key that only you and the recipient know.
- There is no way for us or anybody else without direct access to your device to learn what content is being sent in those messages. We do not store your secret chats on our servers. We also do not keep any logs for messages in secret chats, so after a short period of time we no longer know who or when you messaged via secret chats.
- For the same reasons secret chats are not available in the cloud - you can only access those messages from the device they were sent to or from.

## Media in Secret Chats

- When you send photos, videos or files via secret chats, before being uploaded, each item is encrypted with a separate key, not known to the server. This key and the file's location are then encrypted again, this time with the secret chat's key - and sent to your recipient. They can then download and decipher the file.
- This means that the file is technically on one of Telegram's servers, but it looks like a piece of random indecipherable garbage to everyone except for you and the recipient. We don't know what this random data stands for and we have no idea which particular chat it belongs to.
- We periodically purge this random data from our servers to save disk space.

## Public Chats

- In addition to private messages, Telegram also supports public channels and public groups. All public chats are cloud chats.
- Like everything on Telegram, the data you post in public communities is encrypted, both in storage and in transit but everything you post in public will be accessible to everyone.
