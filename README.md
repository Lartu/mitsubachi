<p align="center">
  <img src="https://raw.githubusercontent.com/Lartu/mitsubachi/master/images/mitsubachi-logo.png">
  <br>
  <img src="https://img.shields.io/badge/license-MIT-gold?style=flat-square">
  <img src="https://img.shields.io/badge/version-1.0-green.svg?style=flat-square">
  <img src="https://img.shields.io/badge/miba-!mitsubachi-orange.svg?style=flat-square">
</p>

**Mitsubashi** is a tiny and open chat protocol, designed to be minimal, easy to implement,
easy to use and easy to understand. Mitsubashi supports nicknames, nickname changing,
user-to-user messaging and channel / group (known in Mitsubashi as "_message lists_") chat.

I designed the Mitsubashi protocol because I found that most common, open chat protocols
(IRC, XMPP, etc.) are to complex to implement properly and completely. Writing a very
bare-bones IRC client is easy, but implementing the whole protocol is not. XMPP messages
are way to heavy to be reliable on low-latency, low-bandwidth networks. The protocol is
also very complex for an amateur, casual client / server writer to implement. Mitsubashi
aims to address all these issues.

## The Protocol

