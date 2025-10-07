# Featured Collections

## Thinking about federated Starter Packs

Bluesky Starter Packs are a cool feature where users can curate lists of accounts that they would like to recommend to others. They have been especially useful to new users looking to fill their empty feeds. This a problem that is also prevalent on the fediverse. That is why Mastodon would like to implement a similar feature.

We have thought about what to include in a first implementation. We have more ideas but want to keep it as simple as possible at first.

Turns out, this is not very simple though, because we think we cannot compromise on the following:

* Starter Packs must be federated. When Alice creates a Starter Pack on her server a.com, Bob on his server b.com should eventually be able to see and interact with Alice's Starter Pack.
* It must be possible to opt-out of being featured in a Starter Pack and also to remove yourself from a Starter Pack at any later point in time. This has to work across server boundaries.
* It must be possible to report Starter Packs in the same way as you can report Posts, e.g. for violation of server rules.

This is why we spent some time thinking about how to federate all of this. The result is a draft for a new FEP.

We welcome feedback on the FEP draft as well as more general feedback and discussion of this upcoming feature.

## FEP draft

You can find the FEP draft in this pull request: #1

Please feel free to add your comments to the PR.

## General feedback / discussion

Please use the [discussions](https://github.com/mastodon/featured_collections/discussions) for any questions or ideas you have about this new feature.

Also please have look at the existing discussions. We might have a question or two we would like some feedback on.

## Timeline

We are looking at including the first version of this in Mastodon 4.6. Development is slated to start in November 2026 so it would be ideal to leave feedback before then.

Thank you all very much.
