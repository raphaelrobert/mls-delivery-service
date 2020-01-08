---
title: The Messaging Layer Security (MLS)delivery service
abbrev: MLS multi-device modes
docname: draft-robert-mls-delivery-service-00
category: info

ipr: trust200902
area: Security
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: R. Robert
    name: Raphael Robert
    organization: Wire
    email: raphael@wire.com
 -    
    ins: B. Beurdouche
    name: Benjamin Beurdouche
    organization: Inria
    email: benjamin.beurdouche@inria.fr

informative:

  MLSARCH:
       title: "Messaging Layer Security Architecture"
       date: 2018
       author:
         -  ins: E. Omara
            name: Emad Omara
            organization: Google
            email: emadomara@google.com
         -  
            ins: R. Barnes
            name: Richard Barnes
            organization: Cisco
            email: rlb@ipv.sx
         -
	    	  ins: E. Rescorla 
            name: Eric Rescorla
            organization: Mozilla 
            email: ekr@rtfm.com
         -
            ins: S. Inguva 
            name: Srinivas Inguva 
            organization: Twitter 
            email: singuva@twitter.com
         -
            ins: A. Kwon 
            name: Albert Kwon
            organization: MIT 
            email: kwonal@mit.edu
         -
            ins: A. Duric 
            name: Alan Duric
            organization: Wire 
            email: alan@wire.com 


  MLSPROTO:
       title: "Messaging Layer Security Protocol"
       date: 2018
       author:
         -  ins: R. Barnes
            name: Richard Barnes
            organization: Cisco
            email: rlb@ipv.sx
         -  ins: B. Beurdouche
            name: Benjamin Beurdouche
            organization: Inria
            email: benjamin.beurdouche@inria.fr
         -
            ins: J. Millican
            name: Jon Millican
            organization: Facebook
            email: jmillican@fb.com
         -
            ins: E. Omara
            name: Emad Omara
            organization: Google
            email: emadomara@google.com
         -
            ins: K. Cohn-Gordon
            name: Katriel Cohn-Gordon
            organization: University of Oxford
            email: me@katriel.co.uk
         -
            ins: R. Robert
            name: Raphael Robert
            organization: Wire
            email: raphael@wire.com

--- abstract

This document the Delivery Service part of Messaging Layer Security (MLS).

--- middle

# Introduction

The Delivery Service component of the MLS architecture is responsible for receiving and redistributing messages between group members. In the case of group messaging, the delivery service may also be responsible for acting as a "broadcaster" where the sender sends a single message to a group which is then forwarded to each recipient in the group by the DS. The DS is also responsible for storing and delivering initial public key material required by clients in order to proceed with the group secret key establishment process.

This document describes different operation modes of the Delivery Service in more detail.

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all
capitals, as shown here.

## Functions of the Delivery Service

The Delivery Service is responsible to perform the following functions:

 - Storing and serving Client Init Keys
 - Receiving messages for a specific group from clients
 - Ordering Commit Messages
 - Fanning out messages for a specific group to clients
 - Storing and serving assistive group metadata for clients

### Storing and serving Client Init Keys

Prior to engaging in an MLS group, clients MUST upload at least one ClientInitKey to the Delivery Service.
Clients SHOULD be able to check which ClientInitKeys are held by the Delivery Service and request the deletion of one or more keys.
Clients MUST be able to request a ClientInitKey of another client from the Delivery Service.

### Receiving messages

The DS MUST accept messages from clients that can prove membership of a specific group. The DS MUST confirm to clients whether a message has been accepted.
The DS can apply rate-limiting if needed.
Application messages and Proposals do not require ordering and can be fanned out immediately.
Commit messages require ordering as described in {{ordering-commit-messages}}.

### Ordering Commit Messages

In order to avoid forks of the group state among clients the DS MUST ensure that Commit Messages are ordered for a specific group. The DS MUST store the sequence number of the previous Commit Message and compare it to the sequence number of a new Commit Message. The sequence number of a new Commit Message MUST be greater than the previous sequence number by 1. The DS MUST reject any other sequence numbers and return the expected sequence number to the client instead. Clients MUST then fetch potentially missing messages form the DS and rebase their Commit Message accordingly before attempting to send the Commit Message to the DS again.

### Fanning out messages

The DS receives the roster of a group either from the sending client, or ca extract it from its locally stored group metadata during the fanout operation.
The DS MUST forward the messages to every client in the group.

## Authentication to the DS

Clients SHOULD authenticate to the DS to prove their membership of a specific group. The authentication SHOULD be deniable, i.e. the DS SHOULD not be able to cryptographically prove to a third party whether a client is part of a certain group.

## Group metadata asssistance

In large groups it might be impractical for clients to exchange the whole tree through Welcome messages. The DS can assist clients by storing the tree (including the leaf nodes and therefore the roster). Clients can update the tree on the DS (remote tree) incrementally when sending Commit messages. This way the remotre tree is in sync with the local view on the client side.
Clients can query the DS for part of or the entirety of the tree. Typically clients only need their direct path, their copath and the leaf nodes to issue Proposals and Commits. If the tree contains blank nodes the direct path and the copath need to be resolved accordingly, the same applies for unresolved leaf nodes. In order to consistently verify signature of tree nodes the whole tree is needed.

[[ OPEN ISSUE: Since Commit messages are opaque to the DS, we need another way to send the changes to the DS ]]

### Protecting the remote tree

[[ OPEN ISSUE: Describe options to encrypt the tree on the DS ]]

### Integrity of the remote tree

In order to prevent a malicious DS from tampering with nodes of the tree, clients can use the node hashes and signatures to verify the integrity within the tree. The Welcome messages contains a tree_hash field that Clients MUST verify.

# Security and Privacy Considerations

## Integrity

## Privacy


# IANA Considerations
This document makes no requests of IANA.
