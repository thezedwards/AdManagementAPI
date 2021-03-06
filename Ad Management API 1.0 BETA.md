![IAB Tech Lab](https://drive.google.com/uc?id=10yoBoG5uRETSXRrnJPUDuONujvADrSG1)

# OpenRTB Ad Management API v1.0 Beta Specification

**OpenRTB 3.0 and AdCOM Companion Specification**

**BETA Draft**

July 24, 2018


**Executive Summary**

The new IAB Tech Lab Ad Management API Specification is part of the [OpenRTB 3.0 Beta release](iabtechlab.com/openmedia). The Ad Management API Spec will have a 60-day public comment period, running through September 24, 2018. Beta testing by the IAB Tech Lab OpenRTB Working Group will be ongoing. Industry members are encouraged to actively participate throughout the beta period, and to get involved with the OpenRTB Working Group. The group will evaluate and incorporate the feedback received, as well as real world insights from beta testing, to produce final specs that will be ready for full industry adoption by the end of the year. 

Ad management occurs when a buyer (or a representative party) submits creatives for creative approval, supply platforms approve or disapprove of those creatives, and buyers receive feedback accordingly. Before the publication of this technical standard, supply platforms have relied on proprietary methods and tools for ad management, or none at all. 

This process is important for a few reasons: to ensure creatives comply with content guidelines, are malware-free, and function correctly. Creative submission is also a technical necessity for certain emerging formats such as digital out-of-home (DOOH) and programmatic TV. In an ad quality review process, analysts or automated processes check the ad's landing page, tracking tags, text, the creative itself, and more. The exact nature of the review process is up to each supply platform. The results of these checks are made available for buying platforms to consume and modify bidding behavior accordingly.

The OpenRTB Working Group has identified a need to standardize the creative submission and ad management process to reduce pain points for buyers and sellers in the digital advertising industry. Using a standard approach allows for ease of integration between buy-side and supply-side platforms. Supply platforms using the standardized Ad Management API gain increased control over the creatives that serve on their platforms. Both supply platforms and demand platforms benefit from increased bidding efficiency, as demand platforms can avoid invalid bids and notify customers of defects with their ads. This can unblock revenue that would otherwise be received if buyers knew that they must make adjustments. Submission of ads in advance also benefits all parties as it reduces approval delays that may exist with current workflows. 

The Ad Management API specification support all major scenarios known at time of publication for both bidding and markup delivery. For bidding, this refers to whether supply platforms permit ads to serve by default ("permissive bidding") or require explicit approval before serving ("restrictive bidding"). For markup delivery, this refers to whether the markup is included in each bid or whether bids refer to pre-uploaded markup by ID.

**About IAB Tech Lab**

The IAB Technology Laboratory is a nonprofit research and development consortium charged with producing and helping companies implement global industry technical standards and solutions. The goal of the Tech Lab is to reduce friction associated with the digital advertising and marketing supply chain while contributing to the safe growth of an industry.

The IAB Tech Lab spearheads the development of technical standards, creates and maintains a code library to assist in rapid, cost-effective implementation of IAB standards, and establishes a test platform for companies to evaluate the compatibility of their technology solutions with IAB standards, which for 18 years have been the foundation for interoperability and profitable growth in the digital advertising supply chain.

**Original Author of the Ad Management API Specification Proposal**

Ian Trider, Director, RTB Platform Operations, Centro and IAB Tech Lab Commit Group Member

**IAB Tech Lab OpenRTB Commit Group Members**

Ian Trider, Centro; Curt Larson, Sharethrough; Jim Butler, AOL; Haskell Garon, Google; Neal Richter, Rakuten Marketing; Allen Dove, SpotX; Pierre Nicolas, Criteo; Bill Simmons, DataXu

**IAB Tech Lab OpenRTB Working Group Members**

[https://iabtechlab.com/working-groups/openrtb-working-group/](https://iabtechlab.com/working-groups/openrtb-working-group/) 

**IAB Tech Lab Contact**

Jennifer Derke, Director of Product, Programmatic & Data, IAB Tech Lab
[openrtb@iabtechlab.com](mailto:openrtb@iabtechlab.com)

# Table of Contents

- [Introduction and Background](#intro)
  - [Dependencies](#dependencies)
  - [Bidding Scenarios](#biddingscenarios)
    - [Permissive Bidding](#permissivebidding)
    - [Restrictive Bidding](#restrictivebidding)
  - [Markup Delivery Methods](#markupdeliverymethods)
    - [Ad Markup In Bid Response](#admarkupinbidresponse)
    - [Exchange-hosted Ad Markup](#exchangehostedadmarkup)
  - [Rationale For This Specification](#rationale)
    - [Restrictive Bidding, Supplier-hosted Ad Markup](#restrictivebiddingexchangehostedadmarkup)
    - [Restrictive Bidding, Ad Markup In Bid Response](#restrictivebiddingadmarkupinbidresponse)
    - [Permissive Bidding, Ad Markup In Bid Response](#permissivebiddingadmarkupinbidresponse)
    - [Emerging Formats](#emergingformats)
- [API Conventions](#apiconventions)
  - [Status Codes](#statuscodes)
  - [Sparse Fieldsets](#sparsefieldsets)
  - [Dates And Times](#datesandtimes)
- [Endpoints](#endpoints)
- [Authentication](#authentication)
- [Resource Representations](#resourcerepresentations)
  - [Webhook Registration](#webhookregistration)
  - [Collection Of Ads](#collectionofads)
  - [Ad](#ad)
  - [Audit](#audit)
- [Webhooks](#webhooks)
  - [Authentication](#authentication)
  - [Webhook Calls](#webhookcalls)
- [Substitution Macros](#substitutionmacros)
- [Typical Synchronization Flow](#typicalsynchronizationflow)
- [Appendix A: Integration Checklist](#appendixa_integrationchecklist)
- [Appendix B: Examples](#appendixb_examples)
  - [Minimal Implementation](#minimalimplementation)
    - [Bidder Ad Submission](#bidderadsubmission1)
    - [Bidder Receives A Webhook Call From Exchange](#bidderreceivesawebhookcallfromexchange1)
    - [Bidder Polls For Updates](#bidderpollsforupdates1)
  - [Typical Implementation](#typicalimplementation)
    - [Bidder Ad Submission](#bidderadsubmission2)
    - [Bidder Polls For Updates](#bidderpollsforupdates2)
- [Appendix C: Resources](#appendixc_resources)


# Introduction and Background <a name="intro"></a>

The purpose of this specification is to provide a standardized means for demand and supply partners to communicate with each other regarding ads that will be used in bidding. The specification provides for a RESTful API to be implemented by supply partners.

Exchanges have vastly different policies with regards to creatives, and this specification makes no attempt to enforce any set of business rules. Rather, such rules are a matter for bidders and exchanges to communicate with each other as a part of integration. The terms "required", "recommended", and "optional" in this specification refer only to technical compliance, not business policy. Likewise, "must" and "will" refer to requirements for a technically compliant implementation.

Wherever possible, this specification makes reference to OpenRTB as it is assumed this is the standard "common language" of bidding that will be used between exchanges and bidders.  While the most common expected use case is that of a DSP interacting with an ad exchange, the OpenRTB Ad Management API is not limited to this purpose and can be used to transmit ad and audit information between partners in more complex supply chains. Wherever the term "bidder" is used, this should be understood to mean more generically "demand partner", and wherever the term "exchange" is used, this should be understood to mean more generically "supply partner." For example, an ad network may have a server-to-server integration with an ad exchange to source demand. In this case, the ad network is the supply partner and the exchange is the demand partner.

## Dependencies <a name="dependencies"></a>

This specification makes use of [AdCOM 1.x](https://github.com/InteractiveAdvertisingBureau/AdCOM) for the definition of the Ad object and its children. 

This specification is not inherently constrained to partners who use OpenRTB during bidding. In fact, this specification is also suitable for management of ads used in non-biddable transactions as well. **While this specification is released as a part of the OpenRTB 3.0 framework, implementation of OpenRTB 3.0 is not required to use this specification.** It can easily be used alongside OpenRTB 2.x bidding implementations.

**NOTE:** Out of convenience, this specification will refer to some shorthand terms for expected common scenarios. These are outlined in the following two sections.

## Bidding Scenarios <a name="biddingscenarios"></a>

### Permissive Bidding <a name="permissivebidding"></a>

In a permissive bidding scenario, the exchange allows a new ad to win impressions until (and if) a point in time occurs at which the ad is deemed to be disapproved. Bidders are expected to stop bidding with disapproved ads, and exchange may discard bid responses using such a creative.

### Restrictive Bidding <a name="restrictivebidding"><a/>

In a restrictive bidding scenario, the exchange does not allow new ads to win impressions until (and if) a point in time occurs at which the ad is deemed to be approved. Exchanges may discard bids made using such a ad until approval.

An exchange using restrictive bidding may choose to allow new ads to enter its audit queue by simply having the bidder start bidding with such an ad ("submission via bidding"), however bids may be discarded by the exchange until it is approved. Submission via bidding allows bidders to participate in auctions even if they have not implemented support for this API, however optimal performance is ensured by doing so.

## Markup Delivery Methods <a name="markupdeliverymethods"></a>

### Ad Markup In Bid Response <a name="admarkupinbidresponse"></a>

In this scenario, the bidder is expected to include the markup/URL for an ad (along with its ID) in each bid response.

The ad submitted by the bidder using this API is expected to be a faithful example of creative markup that behaves in a way that is representative of markup that will be used during bidding. It is expected that the exact markup will vary (for example, there will be varying components for an impression tracking URL, cachebusters, click URL, product IDs and images in dynamic creative, etc.). It is up to exchanges to communicate which deviations used during bidding will be deemed to constitute a material change to the creative (and thus may trigger a change in audit status).

### Exchange-hosted Ad Markup <a name="exchangehostedadmarkup"></a>

In this scenario, the exchange holds the ad markup. During bidding, the bidder makes reference to the markup on file using the "mid" field in the Bid object in the bid response (OpenRTB v3.x) or similar (non-OpenRTB or previous versions) and does not include the actual markup. The exchange provides a means for the bidder to specify custom macros in the markup for which values will be provided at bid time, so that variable components of the markup (impression tracking URLs, cachebusters, click URLs, etc.) may be substituted.

Note that "ad markup" refers to merely the contents of the adm field (for display ads) or similar (e.g. VAST); typically, such markup will contain references to third-party servers for assets, etc. 

**NOTE:** It is expected that exchanges will make clear to their demand partners as a matter of exchange policy whether permissive or restrictive bidding in place and whether bidders should return markup in bid responses or whether exchange-hosted ad markup applies.

## Rationale For This Specification <a name="rationale"></a>

There are a number of reasons why this specification was created. Consider the following sets of scenarios that exist for exchanges:

### Restrictive Bidding, Exchange-hosted Ad Markup <a name="restrictivebiddingexchangehostedadmarkup"></a>

In this scenario, there is an absolute need for an ad management API. This need is currently being filled by proprietary APIs, and DSPs must implement multiple proprietary specs to do business.

### Restrictive Bidding, Ad Markup In Bid Response <a name="restrictivebiddingadmarkupinbidresponse"></a>

In this scenario, when exchanges use "submission via bidding" there is an artificial delay before ads can start to spend -- they are often quarantined and bids for a given ad ID are disallowed from bidding until they have been reviewed by the exchange. The result is that money is "left on the table" while waiting for the audit to clear, campaigns do not begin on time, and (if the exchange and/or bidder is unable or unwilling to implement multiple bids in bid responses) bids are submitted that will be discarded, resulting in opportunity loss as the bidder may have had other viable bids for ads that are already approved. As a result, the rate of discarded bids can grow to be extremely high resulting in significant revenue loss.

The presence of a proprietary ad management API avoids the pitfalls of submission via bidding, but requires costly engineering resources, requiring demand partners to maintain multiple implementations for each exchange.

### Permissive Bidding, Ad Markup In Bid Response <a name="permissivebiddingadmarkupinbidresponse"></a>

This pattern is the most common among exchanges today. However, there are still problems that an ad management API helps rectify. It is not unusual for an exchange to block certain ad IDs platform-wide as a violation of their ad quality policy or due to technical reasons. This information is not communicated to the bidder, which continues to bid with an ad that will not be accepted.  If the exchange and/or bidder is unable or unwilling to implement multiple bids in bid responses, there is a great deal of potential revenue that is lost due to bids being filtered. This is particularly noteworthy with video, where technical problems with ads occur commonly, and as a result, exchanges implement blocks on defective ads. An ad management API makes it possible for the exchange to provide feedback to the bidder which it can incorporate to modify its bidding (e.g. discontinue bidding with ad IDs that the exchange has deemed to be unacceptable). 

### Emerging Formats <a name="emergingformats"></a>

New formats are emerging that are beginning to be transactable via real-time bidding. Particularly, digital out-of-home and programmatic TV are here or on the near horizon. These new formats often have stringent ad approval requirements that necessitate an API.

# API Conventions <a name="apiconventions"></a>

This API adheres to many of the conventions of RESTful APIs. The base protocol used for communication is HTTPS, and JSON is used to represent the body of requests and responses. Requests should be made with an HTTP headers of "Accept: application/json" and "Content-Type: application/json" to indicate that the body of the request will be JSON and that JSON is expected in return. Compression may also be negotiated/used as indicated by HTTP headers (see OpenRTB 3.0 for an example of how this is done).

Breaking changes are restricted to major versions of this specification (1.x, 2.x, etc.).

Almost all fields are optional at a technical level, however exchanges may mandate the presence of certain fields as a business requirement.  Both exchanges and bidders must gracefully deal with the presence of unexpected or unknown fields. Exchanges are recommended to store fields regardless of their relevance to the exchange, as they may hold significance to a bidder. For example, on exchanges which use their own ad IDs, bidders may wish to store their ID in the extension object. 

It is expected that a given exchange operates using either it's own ad IDs or bidder IDs, but not both, for the prevention of ambiguity.

All requests must result in the manipulated object(s) being returned in response. 

## Status Codes <a name="statuscodes"></a>

HTTP status codes are used by the exchange to express the status of requests made by the bidder:

<table>
  <tr>
    <th>Code</th>
    <th>Name</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>200</td>
    <td>OK</td>
    <td>The request was successful.</td>
  </tr>
  <tr>
    <td>400</td>
    <td>Bad Request</td>
    <td>The request could not be interpreted successfully.</td>
  </tr>
  <tr>
    <td>401</td>
    <td>Unauthorized</td>
    <td>The request did not contain correct authentication information.</td>
  </tr>
  <tr>
    <td>404</td>
    <td>Not Found</td>
    <td>The resource does not exist.</td>
  </tr>
  <tr>
    <td>429</td>
    <td>Too Many Requests</td>
    <td>The bidder has exceeded the rate limit set by the exchange and must wait before trying again.</td>
  </tr>
  <tr>
    <td>500</td>
    <td>Internal Service Error</td>
    <td>The exchange has encountered technical difficulties.</td>
  </tr>
</table>


## Sparse Fieldsets <a name="sparsefieldsets"></a>

AdCOM objects returned in responses may contain a sparse fieldset to save bandwidth, where this specification indicates this is acceptable (see "Endpoints" below). In this case fields should be assumed by bidders to be unchanged if not present. Bidders can determine changes made by an exchange by comparing to the object it previously sent. Sparse fieldsets must include at a minimum the following fields, if the object in question is sent:

* Ad object
    * id
    * init (initial ad creation only)
    * lastmod
* Audit object
    * status
    * lastmod

## Dates and Times <a name="datesandtimes"></a>

All dates/times must be specified in the format of ISO 8601 ([using the profile defined by W3C](https://www.w3.org/TR/NOTE-datetime)). More specifically, dates/times must be expressed as complete date plus hours, minutes and seconds, and must always be expressed in UTC. Values should always be rounded **down** to the nearest second to so that when queries are made using a filter, ads are not missed due to rounding.

# Endpoints <a name="endpoints"></a>

Bidders will interact with the Ad Management API by making HTTP calls to specific endpoints. Exchanges will specify a **base URL** (denoted using the {baseUrl} placeholder in this document) and a **bidder ID** representing a given bidder/demand partner (denoted using the {bidderId} placeholder in this document). All endpoints are relative to this base URL. The base URL must include the major version of the specification implemented in the form of "v#". For example, an exchange implementing version 1.0 of this API may define its base URL as `https://api.exchange.com/management/v1`. For example, assuming a bidder ID of 492 the ads endpoint will be reached at `https://api.exchange.com/management/v1/bidder/492/ads`.

Per "API Conventions" above, breaking changes require the major version number to be iterated, and partners must deal gracefully with unexpected fields, so the minor version number is not required in the URL and its omission provide ease of upgrade. 

<table>
  <tr>
    <th>Endpoint</th>
    <th>Methods supported</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>{baseUrl}/bidder/{bidderId}/webhook</td>
    <td>GET, PUT, PATCH</td>
    <td><strong>GET:</strong> returns the webhook registration object for a given bidder. <br />
<strong>PUT:</strong> replaces the webhook registration object for a given bidder. <br />
<strong>PATCH:</strong> replaces only the specified fields in the webhook registration object for a given bidder. <br />

No POST method is supplied as only a single  webhook registration object may exist.</td>
  </tr>
  <tr>
    <td>{baseUrl}/bidder/{bidderId}/ads</td>
    <td>GET, POST</td>
    <td><strong>GET:</strong> returns a collection of ads for a given bidder. This response may be sparse at the exchange's discretion (see "API conventions"). <br />

An "auditStart" filter, at a minimum, must be set on the query string to constrain the number of ads returned, else exchanges may choose to return a 400 status code. Exchanges may limit the number of ads in the returned collection at their discretion. If the result set is a subset of available ads, this will be indicated in the result (see "Collection of ads"). Bidders may fetch the remaining ads by examining the last modification date of the Audit object of the final ad in the collection (the most recently updated ad) and use this as the "auditStart" filter for a subsequent request, repeating until the bidder has gathered all updates. <br />

The available filters are: <br />

**auditStart:** Beginning date/time for the "lastmod" value from the Audit object of returned ads (date/time greater than this value). (Required) <br />
**auditEnd:** Ending date/time for the "lastmod" value from the Audit object of returned ads (date/time less than or equal to this value). (Optional, now is assumed if omitted) <br />

See "API conventions" regarding date format. <br />

For example:  <br />

`/ads?auditStart=2018-06-05T17:51:54Z`


<strong>POST:</strong> submits a single ad. The body must contain a only an Ad object (and its children). Returns a collection of ads containing the ad submitted, including any fields or child objects provided by the exchange. This response may be sparse at the exchange's discretion (see "API conventions").</td>
  </tr>
  <tr>
    <td>{baseUrl}/bidder/{bidderId}/ads/{id}</td>
    <td>GET, PUT, PATCH</td>
    <td><strong>GET:</strong> Returns a collection of ads resource containing a single ad in its entirety. <br />
<strong>PUT:</strong> replaces the ad object in its' entirety, and returns a collection of ads containing the specified ad, including any fields or child objects provided by the exchange. This response may be sparse at the exchange's discretion (see "API conventions"). <br />
<strong>PATCH:</strong> replaces only the specified fields, and returns a collection of ads containing the specified ad, including any fields or child objects provided by the exchange. This response may be sparse at the exchange's discretion (see "API conventions").</td>
  </tr>
</table>


# Authentication <a name="authentication"></a>

HTTP Basic authentication is used. Exchanges will provide bidders with a username and password. This username and password are combined with a colon (username:password) and base64 encoded. The result is used in the Authorization header in all calls the bidder makes to the API, e.g:

`Authorization: Basic <base64 encoded value of "username:password">`

# Resource Representations <a name="resourcerepresentations"></a>

Resources are represented using JSON.

## Webhook Registration <a name="webhookregistration"></a>

A webhook registration resource is an object containing details of a bidder webhook which an exchange may use to notify the bidder upon changes to ads.

<table>
  <tr>
    <th>Attribute</th>
    <th>Type</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>hookurl</td>
    <td>string</td>
    <td>The bidder's webhook URL.</td>
  </tr>
  <tr>
    <td>username</td>
    <td>string</td>
    <td>The username that should be used in Authorization headers.</td>
  </tr>
  <tr>
    <td>password</td>
    <td>string</td>
    <td>The password that should be used in Authorization headers.</td>
  </tr>
</table>


 

## Collection Of Ads <a name="collectionofads"></a>

A collection of ads is an object containing one or more ads with additional metadata.

<table>
  <tr>
    <th>Attribute</th>
    <th>Type</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>count</td>
    <td>integer</td>
    <td>The number of ads in this collection. </td>
  </tr>
  <tr>
    <td>more</td>
    <td>integer</td>
    <td>A boolean flag indicating that this collection is a subset; the number of ads returned has been limited by exchange policy. See "Endpoints" above. May be omitted when not needed.</td>
  </tr>
  <tr>
    <td>ads</td>
    <td>object array</td>
    <td>An array of ad resources. On GET, sorted by oldest to newest date of "lastmod" from the Audit object.</td>
  </tr>
</table>


## Ad <a name="ad"></a>

An ad resource is an object representing each unique ad that will be used by the bidder. It is a AdCOM 1.x ad object with relevant child objects. See the [AdCOM specification](https://github.com/InteractiveAdvertisingBureau/AdCOM) for details.

Only the bidder may modify the ad object and its' children, excepting the Audit object and these fields from the Ad object:

* "id": Set by the exchange (only for exchanges who express ads in terms of their IDs)

* "init": Set by the exchange on initial submission of the ad

* "lastmod": Set by the exchange on the most recently modification to the ad

## Audit <a name="audit"></a>

Subordinate to the Ad object is a AdCOM 1.x audit object. See the [AdCOM specification](https://github.com/InteractiveAdvertisingBureau/AdCOM) for details. Among the many objects that may be present subordinate to the Ad object, it is specifically noted in this specification because its behaviour in the context of ad management is worth elaborating on.

Only the exchange may modify the audit object. Upon initial ad upload, the exchange must initialize the "init" and "lastmod" field to the date/time of the submission of the ad. The "lastmod" field must be updated at any change of this object.

# Webhooks <a name="webhooks"></a>

Exchanges may choose to support sending webhook calls to bidders to notify them of changes to ad audit status, and bidders may choose to receive such calls. Bidders register a webhook URL by manipulating the webhook registration object. 

It is recommended to use webhooks as a means of reducing the required polling frequency substantially. Bidders should still poll occasionally to ensure that all changes are collected (to account for failures during webhook calls, etc.), but at a much lower frequency (such as once per day).

## Authentication <a name="authentication"></a>

Authentication is performed in a similar fashion to that described above on the exchange side, with the registered username and password" being used as a part of the Basic auth header when making calls to the bidder, e.g.:

`Authorization: Basic <base64 encoded value of "username:password">`

## Webhook Calls <a name="webhookcalls"></a> 

If an exchange supports webhooks and a bidder has registered a webhook registration object, upon status change for an ad or ad(s), the exchange will POST a collection of ads to the bidder's "hookurl", with an authorization header as described above.The JSON in the POST may be sparse, containing only changes. The bidder must respond 204 No Content if the webhook is successfully processed. In the event of failures such as timeouts, non-2xx status codes, etc., it is recommended the exchange make at least 3 further attempts with a delay of 30 seconds between each request before abandoning the attempt. It is recommended that exchanges record permanent failures for offline analysis.

# Substitution Macros <a name="substitutionmacros"></a>

In the "exchange-hosted ad markup", since bidders do not include their ad markup in the bid response, the exchange must provide the facility for custom macros so that the bidder can provide any variable details (such as cachebusters, impression ID, product IDs, etc.) needed to serve the ad. These custom macros are substituted by the exchange when serving. The strings which may be used are addressed by the transaction specification used for bidding, for example OpenRTB 3.0. 

The values for these custom macros are supplied by the bidder during bidding.

# Typical Synchronization Flow <a name="typicalsynchronizationflow"></a>

Authentication is not shown here for brevity.

During initial integration, the bidder registers a webhook URL by making an HTTP PUT call to:

`{baseUrl}/bidder/{bidderId}/webhook`

... with a webhook registration object.

As ads are created, the bidder places the ads into a queue for submission. The bidder makes one or more HTTP POST calls to:

`{baseUrl}/bidder/{bidderId}/ads`

... with a body containing a single Ad resource. Exchanges will periodically update the audit object inside the ad with modifications to the **status** (e.g. to set ads as approved or denied), and any other fields as a result of the exchange's assessment of the ad. Bidders should expect that that fields in the audit object could change at any time.

As ads change audit status, the exchange makes HTTP POST calls to the "hookurl" that is registered, containing a collection of ads.

Bidders may also choose to periodically poll for updates (thereby preventing missed information if webhook calls fail) by making HTTP GET calls to:

`{baseUrl}/bidder/{bidderId}/ads?auditStart={timestamp}`

Bidders should record the audit "lastmod" for each ad, and choose an auditStart value based on the most recently observed audit update date/time.

Alternatively, bidders may query for the status of a particular ad by making an HTTP GET call to:

`{baseUrl}/bidder/{bidderId}/ads/{id}`

Bidders should make use of the information they receive to change their bidding behaviour appropriately.

If the bidder has made local changes to an ad, the bidder makes an HTTP PATCH or PUT call to:

`{baseUrl}/bidder/{bidderId}/ads/{id}`

... to update the ad record at the exchange. On an exchange with restrictive bidding, typically, this would result in the ad returning to status 1, "pending audit", but this is a matter of exchange business rules. For example, an exchange might deem only a subset of changes to be significant enough to require a re-audit (such as a change in landing page domain). It is up to exchanges to communicate what deviations constitute a material change to the ad (and thus may trigger a change in audit status).

# Appendix A: Integration Checklist <a name="appendixa_integrationchecklist"></a>

To facilitate integration, exchanges should provide a document similar to the below to inform bidders about the specifics of an exchange's implementation of this spec.

<table>
  <tr>
    <th>Exchange</th>
    <td></td>
  </tr>
  <tr>
    <th>Base URL</th>
    <td></td>
  </tr>
  <tr>
    <th>Bidder ID</th>
    <td></td>
  </tr>
  <tr>
    <th>Version Used</th>
    <td></td>
  </tr>
  <tr>
    <th>Rate Limit</th>
    <td></td>
  </tr>
  <tr>
    <th>Max Ads per Response</th>
    <td></td>
  </tr>
  <tr>
    <th>Markup and Bidding Policy</th>
    <td></td>
  </tr>
  <tr>
    <th>Ad IDs of Record</th>
    <td>(Bidder or Exchange)</td>
  </tr>
  <tr>
    <th>Attributes Required</th>
    <td></td>
  </tr>
  <tr>
    <th>Notes</th>
    <td>Additional notes of relevance regarding this implementation.</td>
  </tr>
</table>

# Appendix B: Examples <a name="appendixb_examples"></a>

## Minimal Implementation <a name="minimalimplementation"></a>

<table>
  <tr>
    <th>Exchange</th>
    <td>SuperAds</td>
  </tr>
  <tr>
    <th>Base URL</th>
    <td>https://api.superads.com/management/v1</td>
  </tr>
  <tr>
    <th>Bidder ID</th>
    <td>496</td>
  </tr>
  <tr>
    <th>Version Used</th>
    <td>v1.0</td>
  </tr>
  <tr>
    <th>Rate Limit</th>
    <td>100 requests per minute</td>
  </tr>
  <tr>
    <th>Max Ads per Response</th>
    <td>100</td>
  </tr>
  <tr>
    <th>Markup and Bidding Policy</th>
    <td>Permissive bidding, ad markup in bid response</td>
  </tr>
  <tr>
    <th>Ad IDs of Record</th>
    <td>Bidder</td>
  </tr>
  <tr>
    <th>Attributes Required</th>
    <td>Required in Ad object: adomain, iurl, one of display or video
Required in Display object: w, h</td>
  </tr>
</table>


This exchange is not interested in receiving the markup itself, merely a representative inspection image ('iurl') to be reviewed by auditors. This is representative of the situation on some mobile exchanges at time of publication.

**NOTE:** Such an implementation would not provide the option for the exchange to scan markup for malicious content. While the API supports such an implementation, a more sophisticated implementation is recommended.

### Bidder Ad Submission <a name="bidderadsubmission1"></a>

POST `https://api.superads.com/management/v1/bidder/496/ads`

```json
{
  "id": "557391",
  "adomain": "advertiser.com",
  "iurl": "http://cdn.dsp.com/iurls/557391.gif",
  "display": {
    "w": 300,
    "h": 250
  }
}
```

Response:

```json
{
  "count": 1,
  "ads": [
    {
      "id": "557391",
      "init": "2018-06-05T17:51:52Z",
      "lastmod": "2018-06-05T17:51:52Z",
      "audit": {
        "status": 2,
        "lastmod": "2018-06-05T17:51:52Z"
      }
    }
  ]
}
```

### Bidder Receives A Webhook Call From Exchange <a name="bidderreceivesawebhookcallfromexchange1"></a>

In this case, notifying it that an ad has been disapproved.

POST `{hookurl}`

```json
{
  "count": 1,
  "ads": [
    {
      "id": "557391",
      "lastmod": "2018-06-05T17:51:52Z",
      "audit": {
        "status": 4,
        "feedback": "Content disallowed by exchange policy.",
        "lastmod": "2018-06-06T12:36:27Z"
      }
    }
  ]
}
```

### Bidder Polls For Updates <a name="bidderpollsforupdates1"></a>

The bidder is requesting all ads whose status has changed since the most recent audit status change observed on last poll (for this example, 2018-06-06T11:00:13Z). In this example, there are more ads that have changed than the maximum the exchange will return in a single call.

GET `https://api.superads.com/management/v1/bidder/496/ads?auditStart=2018-06-06T11:00:13Z`

```json
{
  "count": 100,
  "more": 1,
  "ads": [
    {
      "id": "557391",
      "lastmod": "2018-06-05T17:51:52Z",
      "audit": {
        "status": 4,
        "feedback": "Content disallowed by exchange policy.",
        "lastmod": "2018-06-06T12:36:27Z"
      }
    },
    {
      "id": "557533",
      "lastmod": "2018-06-06T12:51:49Z",
      "audit": {
        "status": 3,
        "lastmod": "2018-06-06T12:51:49Z"
      }
    },
    { ... },
    {
      "id": "557398",
      "lastmod": "2018-06-05T18:22:42Z",
      "audit": {
        "status": 3,
        "lastmod": "2018-06-06T17:43:11Z"
      }
    }
  ]
}
```

GET `https://api.superads.com/management/v1/bidder/496/ads?auditStart=2018-06-06T17:43:11Z`

```json
{
  "count": 2,
  "more": 0,
  "ads": [
    {
      "id": "557231",
      "lastmod": "2018-06-05T19:37:14Z",
      "audit": {
        "status": 4,
        "feedback": "Content disallowed by exchange policy.",
        "lastmod": "2018-06-06T17:45:36Z"
      }
    },
    {
      "id": "557599",
      "lastmod": "2018-06-03T16:21:29Z",
      "audit": {
        "status": 3,
        "lastmod": "2018-06-06T17:47:03Z"
      }
    }
  ]
}
```

## Typical Implementation <a name="typicalimplementation"></a>

<table>
  <tr>
    <th>Exchange</th>
    <td>AdvancedAds</td>
  </tr>
  <tr>
    <th>Base URL</th>
    <td>https://api.advancedads.com/admgmt/v1</td>
  </tr>
  <tr>
    <th>Bidder ID</th>
    <td>34</td>
  </tr>
  <tr>
    <th>Version Used</th>
    <td>v1.0</td>
  </tr>
  <tr>
    <th>Rate Limit</th>
    <td>100 requests per minute</td>
  </tr>
  <tr>
    <th>Max Ads per Response</th>
    <td>100</td>
  </tr>
  <tr>
    <th>Markup and Bidding Policy</th>
    <td>Restrictive bidding, ad markup in bid response</td>
  </tr>
  <tr>
    <th>Ad IDs of Record</th>
    <td>Bidder</td>
  </tr>
  <tr>
    <th>Attributes Required</th>
    <td>Required in Ad: adomain, cat, one of display or video <br />
Required in Display: one of adm or banner, one of w + h or wratio + hratio, type <br />
Required in Banner: img, link <br />
Required in Video: one of adm or curl, api, mime, type</td>
  </tr>
</table>


### Bidder Ad Submission <a name="bidderadsubmission2"></a>

POST `https://api.advancedads.com/admgmt/v1/bidder/34/ads`

```json
{
  "id": "557391",
  "cat": "653",
  "adomain": "ford.com",
  "display": {
    "w": 300,
    "h": 250,
    "secure": 1,
    "adm": "<!-- Markup -->"
  }
}
```

Response:

```json
{
  "count": 1,
  "ads": [
    {
      "id": "557391",
      "init": "2018-06-05T17:51:52Z",
      "lastmod": "2018-06-05T17:51:52Z",
      "audit": {
        "status": 1,
        "lastmod": "2018-06-05T17:51:52Z"
      }
    }
  ]
}
```

Given the bidding policy of the exchange and the initial audit status returned, the bidder cannot use this ad in bidding until it receives an update to the audit status.

### Bidder Polls For Updates <a name="bidderpollsforupdates2"></a>

GET `https://api.advancedads.com/admgmt/v1/bidder/34/ads?auditStart=2018-06-06T11:00:13Z`

```json
{
  "count": 4,
  "more": 0,
  "ads": [
    { ... },
    {
      "id": "557391",
      "lastmod": "2018-06-05T17:51:52Z",
      "audit": {
        "status": 3,
        "feedback": "Corrected category. Added missing attribute.",
        "corr": {
          "cat": "1",
          "attr": 6
        },
        "lastmod": "2018-06-06T12:36:27Z"
      }
    }
  ]
}
```

In the above scenario, the exchange has approved the ad but has updated it to reflect that, in the its opinion, the ad should be classified as category Automotive, and has auto-play in-banner video. The equivalent webhook call would be similar (see above example).

# Appendix C: Resources <a name="appendixc_resources"></a>

Interactive Advertising Bureau Technology Laboratory (IAB Tech Lab)  
[www.iabtechlab.com](https://www.iabtechlab.com)

OpenMedia Specification Stack    
[https://iabtechlab.com/openmedia](https://iabtechlab.com/openmedia)

AdCOM Project on Github  
[https://github.com/InteractiveAdvertisingBureau/AdCOM](https://github.com/InteractiveAdvertisingBureau/AdCOM)

OpenRTB v3.0 Specification  
[https://github.com/InteractiveAdvertisingBureau/openrtb](https://github.com/InteractiveAdvertisingBureau/openrtb)
