__OBIP__: TS-Reports

__Title__: 3rd Party Search Content reporting

__Author__: Tyler Smith <tyler@ob1.io>

__Discussions-To__: Github Issues, #3rd-party-search on Slack, or <tyler@ob1.io>  

__Status__: Draft

__Type__: Standards Track

__Created__: 10/03/2017

__Copyright__: MIT


## Abstract
A specification for the API a search provider should comply with to handle client reports for content.


## Motivation
Search providers have variying content policies and need a way to get reports of infringing content. This Proposal suggests an API for managing these reports.

It is upon providers to handle reports according to their policies.

## Report endpoint announcement

The report endpoint should be announced in the search provider's init response if it exists.

#### Example:

```json
{
  "name": "Example Search",

  "logo": "https://example.com/logo.png",

  "links": {
    "self": "",
    "listings": "https://example.com/search/listings",
    "reports": "https://example.com/reports"
  },

  "options": {},
  "sortBy": {}
}
```

## Request

Reports should be sent to the announced endpoint as an HTTP POST with a JSON body with the following parameters:

Value | Required | Description
--- | --- | ---
`peerID` | true | The Peer ID of the reported content
`slug` | false | The slug of the reported listing
`reason` | false | The reason the content is being reported

#### Example:

```
POST https://example.com/reports
Content-Type: application/json

{
  "peerID": "QmXFMkpBBpL4zcYAArVAecLyypFrRzp2Co4q9oXUtzF7XF",
  "slug": "inappropriate-content-listing",
  "reason": "This listing is inappropriate. It is illegal or undesirable for your search engine."
}
```