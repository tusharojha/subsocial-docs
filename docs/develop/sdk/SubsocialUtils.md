---
id: subsocial-utils
title: Subsocial Utils
displayed_sidebar: developSidebar
---

<head>
  <title>Utility Methods Provided By The Subsocial SDK</title>
</head>

This section covers the utilities that are provided by the `@subsocial/utils` package.
It contains a set of helper functions to help you develop in the Subsocial Ecosystem.

There are several sections which are divided by their functionality. The table below contains links to each section:

| Functionality                 | Summary                                                           |
| ----------------------------- | ----------------------------------------------------------------- |
| [Accounts](#accounts)         | Parse a Substrate address to a Subsocial address.                 |
| [Balances](#balances)         | Format balances to readable formats.                              |
| [Markdown](#markdown)         | Process markdown texts to different formats.                      |
| [Summarize](#summarize)       | Summarize text and markdown to display properly in post previews. |
| [Twitter](#twitter)           | Utilities to help you integrate posts from Twitter to your app.   |
| [Slug](#slug)                 | Create a slug from a content body to improve SEO.                 |
| [Social Links](#social-links) | Manages links from other social media platforms.                  |

## Accounts

Each substrate account has a different address depending on the chain. This is because each chain has an `address prefix`, which may be the same or different from each other. For example, the `Subsocial` parachain and `Soonsocial` testnet have the same `address prefix`, which is `28`. But `Polkadot` has a different prefix, which is `0`. This makes the same account have a different address format for `Subsocial` and `Polkadot`. You can manually convert addresses to any format that you want by using the [Subscan Address Converter](https://polkadot.subscan.io/tools/format_transform).

### toSubsocialAddress

A simple function to convert any substrate address format into Subsocial's address format.

```
toSubsocialAddress(address?: string): string
```

Example:

```javascript
import { toSubsocialAddress } from '@subsocial/utils'
const subsocialAddress = toSubsocialAddress(anySubstrateAddress)
```

## Balances

Most blockchains doesn't support decimals, so `BigInt` is used to store token balances. This `BigInt` numbers are paired with a constant usually named `decimals` which parses the balance into its real value. For example, if user A has 1 token Z, where the `decimals` of token Z is 10, then the value stored in the blockchain for that user's token balance is `10000000000` (with 10 zeros).

However, this is not the value that user wants to see, so we need to format it first to its decimal version. To do so, there are several formatting functions in `@subsocial/utils` that can be used. To simplify things, the value stored on the blockchain will be named `blockchain value`, and the decimal version will be named `real value`.

### simpleFormatBalance

Formats `blockchain value` into a human readable `real value` based on the decimals given.

```
simpleFormatBalance(balance: BN | string | number, decimals?: number, currency?: string, withUnit: boolean = true): string
```

```javascript
import { simpleFormatBalance } from '@subsocial/utils'
const balance = '100000000000000000000' // 1_000_000_000_000_000_000 (18 zeros)
simpleFormatBalance(balance, 10) // 100,000,000.0000 Unit
simpleFormatBalance(balance, 10, 'SUB', 'SUB') // 100,000,000.0000 SUB
simpleFormatBalance(balance, 10, 'SUB', false) // 100,000,000.0000
```

### balanceWithDecimal

Parses `real value` into `blockchain value` based on the decimals given.

```
balanceWithDecimal(balance: string | number, decimal: number): BigNumber
```

```javascript
import { balanceWithDecimal } from '@subsocial/utils'
const balance = '1'
balanceWithDecimal(balance, 10).toString() // 10_000_000_000 (10 zeros)
```

### convertToBalanceWithDecimal

Formats `blockchain value` into `real value` based on the decimals given.

```
convertToBalanceWithDecimal(balance: string | number, decimal: number): BigNumber
```

```javascript
import { convertToBalanceWithDecimal } from '@subsocial/utils'
const balance = '10000000000' // 10_000_000_000 (10 zeros)
convertToBalanceWithDecimal(balance, 10).toString() // 1
```

### formatBalanceWithoutDecimals

Formats `real value` to have a number separator and token symbol

```
formatBalanceWithoutDecimals(balance: BigNumber, symbol: string): string
```

```javascript
import { formatBalanceWithoutDecimals } from '@subsocial/utils'
import BigNumber from 'bignumber.js'

const balance = new BigNumber(100_000)
formatBalanceWithoutDecimals(balance, 'SUB') // 100,000 SUB
```

### toShortMoney

Formats `real value` to a short format. For example, `1_000_000` will be shortened to `1M`

```
type ShortMoneyProps = {
  num: number
  prefix?: string // Any string placed before the amount
  suffix?: string // Any string placed after the amount
  fractions?: number // Number of digits after the decimal point
}
toShortMoney(arg: ShortMoneyProps): string
```

```javascript
import { toShortMoney } from '@subsocial/utils'
toShortMoney({ num: 100_000_000 }) // 100.0M
toShortMoney({ num: 1_000_000_000 }) // 1.0B
```

## Markdown

Subsocial supports any format of content, but for apps built by the Subsocial team, such as PolkaVerse, we use markdown (md) for content. Because of this, we have several utils function related to processing md text.

### mdToText

Converts markdown text into plain text. One example use case for this is displaying previews, where you don't want to have any formatting.

```
mdToText(md: string | undefined): string | undefined
```

```javascript
import { mdToText } from '@subsocial/utils'
const mdText = '# title [link to subsocial](https://subsocial.network)'
console.log(mdToText(mdText)) // title link to subsocial
```

### mdToHtml

Converts markdown text into html.

```
mdToHtml(md: string | undefined): string | undefined
```

```javascript
import { mdToHtml } from '@subsocial/utils'
const mdText = '# title [link to subsocial](https://subsocial.network)'
console.log(mdToHtml(mdText)) // <h1>title <a href="https://subsocial.network">link to subsocial</a></h1>
```

## Summarize

This section contains simple helper functions to summarize long texts. One example is to show a summary instead of a full post in the post list section.

### summarize

Truncates plain text so it doesn't exceed the limit given. The truncation process will not truncate a word if possible.

```
type SummarizeOpt = {
  limit?: number // the max length of text
  omission?: string // the suffix that's added if the text is longer than the limit
}
summarize(text: string, opts: SummarizeOpt): string | undefined
```

```javascript
import { summarize } from '@subsocial/utils'
const text = 'Lorem ipsum dolor sit amet'
summarize(mdText, { limit: 7 }) // Lore...
summarize(mdText, { limit: 10 }) // Lorem... - this doesn't include second word entirely as it will exceed the limit.
summarize(mdText, { limit: 10, suffix: '' }) // Lorem ipsum - removes the suffix
```

### summarizeMd

The same as `summarize`, but it converts md text into plain text first before going to the `summarize` function.
The return of this function is an object, with a `summary` field, which is the same as the summarized text, and `isShowMore` which indicates if that text is truncated or not.

```
type SummarizeOpt = {
  limit?: number // the max length of text
  omission?: string // the suffix that's added if the text is longer than the limit
}
summarize(mdText: string, opts: SummarizeOpt): { summary: string; isShowMore: boolean; }
```

```javascript
import { summarizeMd } from '@subsocial/utils'
const text = '# Lorem ipsum dolor sit amet'
summarizeMd(mdText, { limit: 7 }) // { summary: 'Lore...', isShowMore: true }
summarizeMd(mdText, { limit: 10 }) // { summary: 'Lorem...', isShowMore: true } - this doesn't include second word entirely as it will exceed the limit.
summarizeMd('Lorem', { limit: 10 }) // { summary: 'Lorem', isShowMore: false } - text is not truncated
```

## Twitter

Helpers to integrate Subsocial posts that originated from Twitter.

### parseTwitterTextToMarkdown

Parses plain text to markdown that add links to Twitter for several formats (e.g. tags and mentions)

```
parseTwitterTextToMarkdown(text: string): string
```

```javascript
import { parseTwitterTextToMarkdown } from '@subsocial/utils'
parseTwitterTextToMarkdown('Hi from @SubsocialChain') // Hi from [@SubsocialChain](https://twitter.com/SubsocialChain)
```

## createTwitterURL

Generates a Twitter url from the id and username that are given.

```
createTwitterURL(tweet: { username: string; id: string }): string
```

```javascript
import { createTwitterURL } from '@subsocial/utils'
createTwitterURL({ username: 'SubsocialChain', id: '1622592114419724290' }) // https://twitter.com/SubsocialChain/status/1622592114419724290
```

## Slug

These utility functions will help you to create a content slug for your app. A content slug is where you use, for example, a post title as part of the url of your page, which can help your app gain better SEO (Search Engine Optimization).

### createPostSlug

Converts content that has the `title` or `body` attribute and its `id` to a url content slug.
It primarily uses `title`, but if that doesn't exist, it will use `body`.
If the `title` or `body` is too long (the limit is 60), then it will go through the `summarize` function first to truncate any excess text.

```
type HasTitleOrBody = {
  title?: string,
  body: string
}
createPostSlug(postId: string, content?: HasTitleOrBody): string
```

```javascript
import { createPostSlug } from '@subsocial/utils'
createPostSlug('1', { title: 'My First Subsocial App' }) // my-first-subsocial-app-1
createPostSlug('1', { body: 'My body text' }) // my-body-text-1
```

### getPostIdFromSlug

Gets the `id` from the slug generated by `createPostSlug`.

```
getPostIdFromSlug(slug: string): string | undefined
```

```javascript
import { getPostIdFromSlug } from '@subsocial/utils'
getPostIdFromSlug('my-first-subsocial-app-1') // 1
```

## Social Links

Manage links from other social media platforms.

### extractVideoId

Extracts the video id of YouTube videos.

```
extractVideoId(url: string): string | false
```

```javascript
import { extractVideoId } from '@subsocial/utils'
extractVideoId('https://www.youtube.com/watch?v=vC9VAylxuJY') // vC9VAylxuJY
extractVideoId('https://youtu.be/vC9VAylxuJY') // vC9VAylxuJY
```

### getEmbedUrl

Generates an embed url for a supported video platform url.
Currently supported platforms: `vimeo`, `youtube`, `youtu.be`, `soundcloud`
The list of supported platforms is also available in the variable `allowEmbedList` exported in the utils package.

```
getEmbedUrl(url: string, embed: string | undefined): string | undefined
```

```javascript
import { getEmbedUrl } from '@subsocial/utils'
getEmbedUrl('https://www.youtube.com/watch?v=vC9VAylxuJY', 'youtube') // https://www.youtube.com/embed/vC9VAylxuJY
getEmbedUrl('https://youtu.be/vC9VAylxuJY', 'youtu.be') // https://www.youtube.com/embed/vC9VAylxuJY
```

### isSocialBrandLink

Checks whether the link is from the specified social platform brand.

```
isSocialBrandLink(brand: SocialBrand, link: string): boolean
```

```javascript
import { isSocialBrandLink } from '@subsocial/utils'
isSocialBrandLink('twitter', 'https://twitter.com/SubsocialChain') // true
isSocialBrandLink(
  'linkedIn',
  'https://www.linkedin.com/company/subsocialnetwork'
) // true
isSocialBrandLink('gitHub', 'https://www.linkedin.com/company/subsocialnetwork') // false
```

### getLinkBrand

Gets the platform where the link is originated. If link's platform is not supported, it will return `website` as the default.

```
getLinkBrand(link: string): SocialBrand
```

```javascript
import { getLinkBrand } from '@subsocial/utils'
getLinkBrand('https://twitter.com/SubsocialChain') // twitter
getLinkBrand('https://www.linkedin.com/company/subsocialnetwork') // linkedIn
```
