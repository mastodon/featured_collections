---
slug: "xxxx"
authors: David Roetzel <david@joinmastodon.org> 
status: DRAFT
dateReceived: 1970-01-01
discussionsTo: https://forum.example/topics/xxxx
---
# FEP-xxxx: Consent-respecting Featured Collections

## Summary

This FEP describes both a new object type and mechanism to allow users to curate collections of other users (actors) and possibly other objects that they would like to recommend to others. These collections, sometimes referred to as "Starter Packs", help new users find interesting people and content to follow.

Users are able to opt in to being included in these collections and can remove themselves from them. Problematic collections can be reported and moderated just like other content.

## Background 

"Starter Packs", a feature pioneered by Bluesky, have proven to be a brilliant way to help new users to find exactly the right people to follow. This can be an important tool to combat the "empty feed" problem that many new users experience on the fediverse and that sometimes turns them away. As such, there has been a lot of interest in implementing Starter Packs on the fediverse.

But while the basic idea of a "Starter Pack" is quite simple, the federated nature of the network poses some challenges that need to be overcome. For example it should be possible to interact with remote "Starter Packs" that were created on a different server and possibly by a different software.

Last but not least, "Starter Packs" need to be handled with care as they can be an easy vector for harassment. Users need control over which "Starter Packs" they are included in and "Starter Packs" need to be subject to the same moderation procedures already in place for other types of content.

GoToSocial has pioneered "Interaction Policies" to model a user's preferences for different kinds of interactions. And in [FEP-044f] Mastodon has expanded on this idea with the addition of verifiable "stamps" to prove user's consent.

This FEP takes those concepts and applies them to a user-curated and federated collection of actors (or any kind of object really) called `FeaturedCollection`. The name was chosen because some platforms already announced they do not plan to use the term "Starter Pack" and to illustrate that other uses are possible.

## Representation of Featured Collections

Featured collections are represented by a new object type, `FeaturedCollection`. `FeaturedCollection` is a subtype of `OrderedCollection` and inherits all of its properties.

A `FeaturedCollection` MUST have the following properties:

* `type`: The type of object, MUST be `FeaturedCollection`
* `id`: IRI that uniquely identifies this object
* `name`: The name of the collection
* `summary`: A description of the collection
* `attributedTo`: The actor responsible for this collection
* `published`: The date and time at which the object was published
* `totalItems`: The number of items in this collection
* `orderedItems`: The list of items in this collection. Since this is a kind of `OrderedCollection` it MAY alternatively use pagination instead. This should only be used for larger collections if possible. Please see the note about limiting the number of entries below and the [note about paginated ordered collections](https://www.w3.org/TR/activitystreams-core/#h-paging) in [ActivityStreams].

In addition, a `FeaturedCollection` MAY have the following property:

* `sensitive`: Set to `true` if either description of the collection or individual items could be seen as being offensive or otherwise problematic
* `updated`: The date and time at which the object was updated *if* it was updated after initial creation
* `icon`: An image that represents the collection. This SHOULD be a square image that can be used in list views and similar alongside the `name`.
* `image`: A larger image that can be used as a page header or as part of a link preview. Similar considerations apply as with OpenGraph images.

It is worth emphasizing that both `icon` and `image` are separately optional. Providers of `FeaturedCollection`s may choose to supply both, only one, or neither. Applications displaying `FeaturedCollection`s may also elect to show or omit either or both images, depending on what makes sense in their UI design and the specific situation. This FEP considers these images decorative in nature, meaning they should not be the only source of important information.

The individual items in the `FeaturedCollection` are of the type `FeaturedItem` which is a subtype of `Object`. A `FeaturedItem` MUST have the following property:

* `featuredObject`: The `id` (IRI) of the actual object that is being featured
* `featuredObjectType`: The `type` of the object that is being featured

In the case that the featured object is an actor it MUST also include the following property:

* `featureAuthorization`: URI of the approval object (see section below for details)

Please note that initially the featured objects are expected to be actors. But the specification is intentionally open to also include other object types. The most obvious one that platforms might want to add in the future is `Hashtag`.

Example featured collection:

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://w3id.org/feps/xxxx"
  ],
  "type": "FeaturedCollection",
  "id": "https://fedi.example.com/users/alice/featured/23",
  "name": "Most interesting posters",
  "summary": "A selection of accounts that I follow because of their interesting content.",
  "attributedTo": "https://fedi.example.com/users/alice",
  "sensitive": false,
  "icon": {
    "type": "Image",
    "url": {
      "type": "Link",
      "mediaType": "image/jpeg",
      "href": "https://fedi.example.com/assets/alice_sp_23.jpg"
    }
  },
  "totalItems": 2,
  "orderedItems": [
    {
      "id": "https://fedi.example.com/users/alice/featured/23/items/1",
      "type": "FeaturedItem",
      "featuredObject": "https://fedi.example.com/users/jennifer",
      "featuredObjectType": "Person",
      "featureAuthorization": "https://fedi.example.com/users/jennifer/stamps/12"
    },
    {
      "id": "https://fedi.example.com/users/alice/featured/23/items/2",
      "type": "FeaturedItem",
      "featuredObject": "https://other.example.com/users/jim",
      "featuredObjectType": "Person",
      "featureAuthorization": "https://other.example.com/users/jim/stamps/21"
    }
  ],
  "published": "2025-08-14T12:12:12Z",
  "updated": "2025-08-14T13:17:25"
}
```

Very large lists of items do not make much sense from a UX perspective. E.g. not many users will want to blindly follow a couple of hundred of unknown accounts. And forcing remote servers to potentially fetch a lot of unknown actors is a vector for Denial of Service (DOS). That is why fediverse software SHOULD both impose a limit to the number of items that can be added to a featured collection and that they will handle when dealing with remote collections. The proposed maximum of items is 150 but implementations MAY have different limits.

## Featured Collections on Actors

Featured collections are created by individual actors. As such they SHOULD become part of the actor's `featured` collection. This way other servers on the fediverse can easily discover them.

## Federating Changes and Opportunistic Updating

When a user creates a new `FeaturedCollection`, this is then added to their actor's `featured` collection, an operation that can be federated as an `Add` activity to the user's followers.

Similarly, when a `FeaturedItem` is added to a `FeaturedCollection`, this can also be distributed in the form of an `Add` activity to all followers of the user that owns the collection and all actors that are already part of the collection.

`Remove` activities can be sent in case of removal from one of the mentioned collections.

This solves the distribution of featured collections and updates to them, but only to followers of their curator plus members of the collection.

Featured collections will eventually end up on servers without any followers of that original curator, though. This means that implementations SHOULD try to re-fetch these collections from time to time to make sure the content is still current. 

## Interaction Policies for Curated Collections

Users MUST be able to consent to being included in featured collections. To signal a user's preferences in that regard an `interactionPolicy` object, as first introduced by GoToSocial, MUST be added to the user's actor. This `interactionPolicy` MUST have a property `canFeature`.

Abbreviated example actor:

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://gotosocial.org/ns",
    "https://w3id.org/feps/xxxx"
  ],
  "id": "https://example.com/users/alice",
  "type": "Person",
  "interactionPolicy": {
    "canFeature": {
      "automaticApproval": [ "https://fedi.example.com/users/alice/followers" ],
      "manualApproval": [ "https://www.w3.org/ns/activitystreams#Public" ]
    }
  }
  // ...
}
```

The two properties of the `canFeature` object can be used to signal who is always allowed to feature this actor in a featured collection (`automaticApproval`) and who might do so but would need a manual approval (`manualApproval`).

The value in both cases MUST be an array consisting of actor objects, `Collection` of actor objects or the special collection `https://www.w3.org/ns/activitystreams#Public`. Note that the latter might also be represented as `as:Public` or simply `Public`. Implementations SHOULD handle all three possible representations.

In practice, the only values that SHOULD be used are the `id` of the actor itself (see below), the `followers` and the `following` collection of the actor and `https://www.w3.org/ns/activitystreams#Public`.

Actors not specifically mentioned or included in one of the collections are never allowed to feature the actor.

The absence of an `interactionPolicy` MUST be treated as missing consent and the affected actors MUST NOT be added to featured collections ever.

To make a policy of never wanting to be featured explicit, `interactionPolicy.canQuote.automaticApproval` SHOULD contain the actor's `id` as its single value. This is because an empty array is equivalent to a missing property under JSON-LD canonicalization. 

In any case this general policy is just that, a general policy, and MUST NOT be confused with actual consent. This means that one can use this policy to determine which actors may be added to featured collections, but one always has to check if the approval for a specific featured collection was really given. See the next section for details.

Note that this is modeled closely after interaction policies for quote posts (see [FEP-044f]).

## Obtaining consent

While interaction policies signal an actor's general preferences any attempt to include an actor in a featured collection MUST ask for consent explicitly.

To do so a new activity type `FeatureRequest` is introduced. It has two mandatory properties:

* `object`: The actor to be included in the featured collection
* `instrument`: The featured collection

When adding an actor to a featured collection the owner of said collection MUST send a `FeatureRequest` activity to the actor that is about to be added.

Example `FeatureRequest`:

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://w3id.org/feps/xxxx"
  ],
  "id": "https://fedi.example.com/users/alice/featured/23/requests/2",
  "type": "FeatureRequest",
  "object": "https://other.example.com/users/bob",
  "instrument": "https://fedi.example.com/users/alice/featured/23"
}
```

In response the actor can either issue an `Accept` or a `Reject` activity. In case of automatic approval, those can be issued immediately. In case of manual approval, a user needs to be notified and asked before the answer can be sent.

In both cases, `Accept` or `Reject`, the object of the activity is the `FeatureRequest`. In case of an `Accept` the activity MUST also include a `result` property pointing to a `FeatureAuthorization`.

Example `Accept` activity:

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams"
  ],
  "type": "Accept",
  "to": "https://fedi.example.com/users/alice",
  "actor": "https://other.example.com/users/bob",
  "object": "https://fedi.example.com/users/alice/featured/23/requests/2",
  "result": "https://other.example.com/users/bob/stamps/1024"
}
```

Example `Reject` activity:

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams"
  ],
  "type": "Reject",
  "to": "https://fedi.example.com/users/alice",
  "actor": "https://other.example.com/users/bob",
  "object": "https://fedi.example.com/users/alice/featured/23/requests/2",
}
```

When the server that issued the `FeatureRequest` receives an `Accept` it SHOULD add a new `FeaturedItem` to the `FeaturedCollection` in which case the `featureAuthorization` property MUST include the `result` of the `Accept` activity.

Example `FeaturedItem` resulting from the `Accept` above:

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://w3id.org/feps/xxxx"
  ],
  "id": "https://fedi.example.com/users/alice/featured/23/items/2",
  "type": "FeaturedItem",
  "object": "https://other.example.com/users/bob",
  "featureAuthorization": "https://other.example.com/users/bob/stamps/1024"
}
```

In case of a `Reject`, a new `FeaturedItem` MUST NOT be created and nothing is added to the `FeaturedCollection`.

## Verification

The `FeatureAuthorization` obtained through the `Accept` activity as described in the previous section serves as an "approval stamp", an object that can be used to verify that approval to be included in a featured collection was given.

A `FeatureAuthorization` MUST include the following properties:

* `type`: The type of object, MUST be `FeatureAuthorization`
* `id`: IRI that uniquely identifies this object
* `interactingObject`: The featured collection 
* `interactionTarget`: The actor that was featured
 
Example `FeatureAuthorization`:

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://gotosocial.org/ns",
    "https://w3id.org/feps/xxxx"
  ],
  "id": "https://other.example.com/users/bob/stamps/1024",
  "type": "FeatureAuthorization",
  "interactingObject": "https://fedi.example.com/users/alice/featured/23",
  "interactionTarget": "https://other.example.com/users/bob"
}
```

When processing a `FeaturedCollection` from a remote server the `FeatureAuthorization` of every `FeaturedItem` MUST be validated. If it is missing, cannot be resolved or the hosting service does not match the actor's the item MUST be ignored or removed from the collection before it is being displayed to users. 

## Revocation

Actor's can opt out of being featured after the fact. In that case they MUST issue an `Delete` activity with the `FeatureAuthorization` as `object`.

Example `Delete` activity:

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams"
  ],
  "type": "Delete",
  "to": "https://fedi.example.com/users/alice",
  "actor": "https://other.example.com/users/bob",
  "object": "https://other.example.com/users/bob/stamps/1024"
}
```

When a `Delete` activity for a `FeatureAuthorization` is received the affected `FeaturedItem` MUST be removed from the `FeaturedCollection`.

## Moderation

Featured collections can include language or imagery that are against a given server's rules. As such it MUST be possible to report them and to handle reports received.

Just like with other objects, a report is federated as a `Flag` activity and `FeaturedCollection` MAY be added to the list of reported objects:

Example activity:

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams"
  ],
  "id": "https://other.example.com/reports/17324",
  "type": "Flag",
  "actor": "https://other.example.com/actor",
  "to": "https://fedi.example.com/users/alice",
  "content": "Inappropriate language in collection description",
  "objects": [
    "https://fedi.example.com/users/alice",
    "https://fedi.example.com/users/alice/featured/23"
  ]
}
```

## Implementation Guidelines

The mechanisms described offer a lot of flexibility and thus can lead to some complexity in implementations. But implementations do not have to be complex to work. There is a spectrum of possibilities.

At the lower end of that spectrum sit implementations that do not want to offer their users any control or agency at all. If you think it should always be possible for users to be featured, you can add the same interaction policy to every actor and simply always return positive authorizations. This can be a fully automated process with little overhead.

Similarly an implementation could decide to never allow their users to be added. So either no, or a special interaction policy can be included and authorizations always denied.

At the other end of the spectrum lie implementations that offer their users full flexibility. This would include fine-grained settings for both manual and automatic approval, leading to complex interaction policies. And for manual approval they would probably need a special UI notifying users of a request to be featured with affordances to either accept or deny that request.

Of course there is a lot of middle ground here. A reasonable approach that sits somewhat in the middle could use automatic approval only and offer users a single setting for their preference with a handful of options to chose from. To make up for the lack of manual approval, removing oneself from a featured collection could be made very easy.

## References

- Christine Lemmer-Webber, Jessica Tallon, Erin Shepherd, Amy Guy, Evan Prodromou, [ActivityPub], 2018
- James M Snell, Evan Prodromou, [ActivityStreams], 2017
- Claire, [FEP-044f], 2025

[ActivityPub]: https://www.w3.org/TR/activitypub/
[ActivityStreams]: https://www.w3.org/TR/activitystreams-core/
[FEP-044f]: https://codeberg.org/fediverse/fep/src/branch/main/fep/044f/fep-044f.md

## Copyright

CC0 1.0 Universal (CC0 1.0) Public Domain Dedication

To the extent possible under law, the authors of this Fediverse Enhancement Proposal have waived all copyright and related or neighboring rights to this work.
