<pre>
OBIP: X
Title: 3rd Party Peer Endorsements
Author: Tyler Smith <tyler@ob1.io>
Discussions-To: Github Issues
Status: Draft
Type: Standards Track
Created: 02/28/2018
Copyright: MIT
Replaces: 8
</pre>

## Abstract
A change to the OBIP8 Verified Moderators system to make it a more generic and flexible service. It outlines a specification for a system allowing 3rd parties to provide a list of peers that it is endorsing in some way, with the nature of the endorsements being defined by the 3rd party itself.

## Motivation
Allow users on OpenBazaar to have more information about the identity and reputation of peers that they interact with. If a 3rd party can provide accurate endorsements then users can make safer decisions.

## Endorsement endpoint

To provide endorsements a provider must have a public endpoint returning data with the following schema:

#### Example

```json
{
  "data": {
    "name": "BazaarCo",
    "description": "Peers Endorsed by BazaarCo",
    "link": "https://bazaarco.example.com/endorsements"
  },
  "types": [
    {
      "name": "standard",
      "description": "A peer that has been vetted by BazaarCo",
      "badge": "https://bazaarco.example.com/standard_badge.png"
    },
    {
      "name": "small_bonded",
      "description": "A peer that has bonded $1000 with BazaarCo",
      "badge": "https://bazaarco.example.com/small_bonded_badge.png"
    },
    {
      "name": "accredited_investor_us",
      "description": "A peer that has been confirmed as a US accredited investor",
      "badge": "https://bazaarco.example.com/accredited_investor_us.png"
    },
    {
      "name": "real_life_lawyer_ca",
      "description": "A peer that has been confirmed as a licensed lawyer in Canada",
      "badge": "https://bazaarco.example.com/real_life_lawyer_ca.png"
    },
    {
      "name": "known_scammer",
      "description": "A peer that has been reported and confirmed as a scammer",
      "badge": "https://bazaarco.example.com/known_scammer.png"
    },
  ],
  "peers": [
    {
      "id": "QmXFMkpBBpL4zcYAArVAecLyypFrRzp2Co4q9oXUtzF7XF",
      "type": "standard"
    },
    {
      "id": "QmVFNEj1rv2d3ZqSwhQZW2KT4zsext4cAMsTZRt5dAQqFJ",
      "type": "small_bonded"
    }
  ]
}
```

#### Response Explanation

- `data` contains general overview information about the service to be displayed to the user in the client.
  - `name` is the name of the endorsement service.
  - `description` is a brief explanation of what this service is offering.
  - `link` is a URI to a document explaining further information about the service.
- `types` is an array detailing the types of endorsements being offered.
  - `name` is the name of the endorsement type.
  - `description` is a brief explanation of what this type represents.
  - `badge` is a URI pointing to a badge icon to display on endorsed listings and profiles.
- `peers` is the actual list of endorsed peers.
  - `id` is the ID of the peer being endorsed.
  - `type` is a reference to the `name` of the endorsement type for this peer.

#### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-06/schema#",
  "title": "Peer Endorsements",
  "description": "A set of endorsed peers",
  "type": "object",
  "properties": {
    "data": {
      "description": "General overview information about the service",
      "type": "object",
      "properties": {
        "name": {
          "description": "The name of the endorsement service",
          "type": "string"
        },
        "description": {
          "description": "A brief explanation of what this service is offering",
          "type": "string"
        },
        "link": {
          "description": "A URI to a document explaining further information about the service",
          "type": "string"
        }
      },
      "required": ["name", "description", "link"]
    },
    "types": {
      "description": "Types of endorsements",
      "type": "array",
      "minItems": 1,
      "items": {
        "type": "object",
        "properties": {
          "name": {
            "description": "The name of the endorsement type",
            "type": "string"
          },
          "description": {
            "description": "A brief explanation of what this type represents",
            "type": "string"
          },
          "badge": {
            "description": "A URI pointing to a badge icon to display on endorsed listings and profiles",
            "type": "string"
          }
        },
        "required": ["name", "description", "badge"]
      }
    },
    "peers": {
      "description": "Endorsed peer list",
      "type": "array",
      "minItems": 1,
      "items": {
        "type": "object",
        "properties": {
          "id": {
            "description": "The ID of the peer",
            "type": "string"
          },
          "type": {
            "description": "The type of endorsement for this peer",
            "type": "string"
          }
        },
        "required": ["id", "type"]
      }
    }
  },
  "required": ["data", "types", "peers"]
}
```