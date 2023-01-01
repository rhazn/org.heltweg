---
title: "Analyzing twitter: Import tweets with NodeJS and the twitter API"
date: 2019-10-25
slug: "import-tweets-with-nodejs-and-the-twitter-api"
description: "Work with the twitter api and nodejs, importing tweets for later use"
tags: ["software-engineering"]
---

# A tweet in the database is worth two in the API

Working with tweets from the twitter API probably means importing data into your own database - the standard API does not provide historical data (only the last seven days) and has various rate limits.

So regardless of the final goal in this blog we'll explore importing tweets from the API into a database for future use. All done with NodeJS, written in Typescript and utilizing MongoDB as data store.

# Big numbers, big problems

Once you authenticate with the API and pull in the first tweets (for example using [the twitter module on npm](https://www.npmjs.com/package/twitter)) you will notice tweets contain ids as numbers and "id_str" which is the same id just as string:

```json
{
 "created_at": "Wed Oct 10 20:19:24 +0000 2018",
 "id": 1050118621198921728,
 "id_str": "1050118621198921728",
 "text": "To make room for more expression, we will now count all emojis as equal—including those with gender‍‍‍ ‍‍and skin t… https://t.co/MkGjXf9aXm",
 "user": {},  
 "entities": {}
}
```

The reason for this is that some languages (Javascript being one of them) can not work with big numbers. For example JS numbers are internally 64-bit floats and only use the first 53 bits for the integer value. Javascript provides the static property Number.MAX_SAFE_INTEGER as 9007199254740991 which is smaller than the id in the example tweet already.

To work with tweet ids we need a way to handle bigger numbers and use the "id_str". [big.js](https://www.npmjs.com/package/big-js) provides that functionality and is used in all following code examples.

# Saving tweets

Saving tweets in MongoDB is easy. Since we are using typescript we can rely on the excellent (Typegoose library)[https://github.com/typegoose/typegoose] to create models for tweets and interact with MongoDB:

```typescript
import { prop, Typegoose, index } from "@hasezoey/typegoose";

@index({ "entities.user_mentions.screen_name": 1 })
export class TwitterStatus extends Typegoose {
    @prop({ required: true, unique: true, index: true })
    id_str!: string;

    @prop({ required: true })
    full_text!: string;

    @prop({ required: true })
    entities!: { user_mentions: { screen_name: string }[] }

    @prop({ required: true })
    created_at!: string;
}

export const TwitterStatusModel = new TwitterStatus().getModelForClass(TwitterStatus, { schemaOptions: { strict: false } });
```

Notice I only defined some properties I wanted to use in this model & the index is also related to my use case. You might need to change those depending on the project.

If schemaOptions define strict as false (see the last line) typegoose saves the whole JSON of the tweet in MongoDB, not just defined fields.

# Import logic

To optimize the amount of tweets you can crawl from the API in the limits twitter provides an excellent resource on using the since_id and max_id parameters correctly here: [https://developer.twitter.com/en/docs/tweets/timelines/guides/working-with-timelines](https://developer.twitter.com/en/docs/tweets/timelines/guides/working-with-timelines).

In summary this means:
* set the since_id to the highest tweet id your application has already imported defining a lower bound for the imported tweets
* set the max_id to the max_id from the last import and subtract 1 defining the upper bound
* import tweets while setting max_id to the lowest id in the returned list until no new ones are returned, moving the upper bound closer to the lower bound
* once no new tweets are returned set max_id to undefined to remove the upper bound for future imports



If you want to crawl all mentions for an account you can keep track of your crawl status with this model:
```typescript
import { prop, Typegoose } from "@hasezoey/typegoose";

export class TwitterCrawlStatus extends Typegoose {
    @prop({ required: true, unique: true, lowercase: true, trim: true })
    account!: string;

    @prop({ trim: true })
    sinceId?: string;

    @prop({ trim: true })
    maxId?: string;

    @prop({ trim: true })
    overallMaxId?: string;
}

export const TwitterCrawlStatusModel = new TwitterCrawlStatus().getModelForClas(TwitterCrawlStatus);
```

A basic algorithm without any safeguards against failing that uses that logic and imports all mentions for a specific account follows:

```typescript
    while(true) {
        const twitterCrawlStatus = await TwitterCrawlStatusModel.findOne({ account: account };

        if (!twitterCrawlStatus) {
            twitterCrawlStatus = await TwitterCrawlStatusModel.create({ account: account });
            await twitterCrawlStatus.save();
        }

        const tweets = await twitterService.getMentions(
            account,
            twitterCrawlStatus.sinceId ? Big(twitterCrawlStatus.sinceId) : undefined,
            twitterCrawlStatus.maxId ? Big(twitterCrawlStatus.maxId).minus(1) : undefined,
        );

        if (tweets.length > 0) {
            await TwitterStatusModel.bulkWrite(tweets.map(tweet => {
                return {
                    updateOne: {
                        filter: { id_str: tweet.id_str },
                        update: { $set: tweet },
                        upsert: true
                    }
                }
            }));

            const lowestId = (getLowestId(tweets) as Big);
            const highestId = (getHighestId(tweets) as Big);

            twitterCrawlStatus.maxId = lowestId.toFixed();

            if (!twitterCrawlStatus.overallMaxId || Big(twitterCrawlStatus.overallMaxId).lt(highestId)) {
                twitterCrawlStatus.overallMaxId = highestId.toFixed();
            }
        } else {
            twitterCrawlStatus.sinceId = twitterCrawlStatus.overallMaxId;
            twitterCrawlStatus.maxId = undefined;
        }

        await twitterCrawlStatus.save();

        if (tweets.length === 0) {
            break;
        }
    }
```

# The twitter service

The twitter service itself is just a minimalist wrapper around the twitter npm module:

```typescript
import * as Twitter from "twitter";
import { Status } from "twitter-d";
import Big from "big.js";

export class TwitterService {
    private client: Twitter;

    constructor(
        consumerKey: string,
        consumerSecret: string,
        bearerToken: string
    ) {
        this.client = new Twitter({
            consumer_key: consumerKey,
            consumer_secret: consumerSecret,
            bearer_token: bearerToken
        });
    }

    public async getMentions(
        account: string,
        sinceId?: Big | undefined,
        maxId?: Big | undefined
    ): Promise<Status[]> {
        return await this.client.get("search/tweets", {
            q: `@${account} -filter:retweets`,
            result_type: "recent",
            count: 100,
            include_entities: true,
            tweet_mode: "extended",
            since_id: sinceId ? sinceId.toFixed(0) : undefined,
            max_id: maxId ? maxId.toFixed(0) : undefined
        }).then(response => {
            return response.statuses;
        });
    }
}
```

{{< aboutme >}}