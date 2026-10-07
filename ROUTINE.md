# Dev News Digest: Routine prompt

This is the prompt the scheduled Routine sends to a fresh Claude Code cloud session
at 06:00, 12:00 and 18:00 Europe/Stockholm. The live copy is stored on the Routine
itself; keep this file in sync when the prompt changes.

---

You are running the scheduled "Dev News Digest" job. Your job is to summarize developer
and software news published since the previous digest and publish it to dev.to. Work
autonomously; nobody is watching this session. Do not create, change or commit files in
any repository, and do not open pull requests. Use a scratch directory under /tmp for
any temporary files. Never print, log or echo the value of DEVTO_API_KEY.

Every request to dev.to must send the header `User-Agent: Mozilla/5.0 (news-digest bot)`.
dev.to answers HTTP 403 with an empty body to Python's default user agent, so use curl
with `-A "Mozilla/5.0 (news-digest bot)"` for all dev.to calls.

## 1. Find the cutoff time

- Call `GET https://dev.to/api/articles/me/published?per_page=30` with the headers
  `api-key: $DEVTO_API_KEY` and `Accept: application/vnd.forem.api-v1+json`.
- Among the returned articles, keep those whose title starts with `Dev News Digest`.
  The cutoff is the newest `published_at` among them.
- Fetch that newest digest with `GET https://dev.to/api/articles/<id>` and keep its
  `body_markdown`. Do not repeat a story it already covered unless there is real news
  about it.
- If there is no earlier digest, use now minus 6 hours. If the cutoff is more than 24
  hours ago, use now minus 24 hours.

## 2. Fetch the sources

Fetch each feed with curl (`-L --max-time 20`, user agent `Mozilla/5.0 (news-digest bot)`)
and parse it with Python's standard library (RSS `<item>` and Atom `<entry>`). Keep only
items published after the cutoff. If a feed fails, skip it and note it; do not stop.

- Hacker News: https://hnrss.org/frontpage (if it fails, use https://news.ycombinator.com/rss
  instead; that feed has no dates, so treat its items as new and rely on the selection
  step to skip stories already covered in the previous digest)
- Lobsters: https://lobste.rs/rss
- GitHub Blog: https://github.blog/feed/
- InfoQ: https://feed.infoq.com/
- The New Stack: https://thenewstack.io/feed/
- Stack Overflow Blog: https://stackoverflow.blog/feed/
- Ars Technica: https://feeds.arstechnica.com/arstechnica/index
- TechCrunch: https://techcrunch.com/feed/
- Changelog News: https://changelog.com/news/feed
- JavaScript Weekly: https://cprss.s3.amazonaws.com/javascriptweekly.com.xml
- TypeScript Blog: https://devblogs.microsoft.com/typescript/feed/
- Simon Willison: https://simonwillison.net/atom/everything/
- Node.js Blog: https://nodejs.org/en/feed/blog.xml
- Deno Blog: https://deno.com/feed
- Cloudflare Blog: https://blog.cloudflare.com/rss/

## 3. Select and write

- Pick the 8 to 15 most significant items for software developers: releases, security
  issues, notable tools, platform and industry changes, and widely discussed posts.
  Skip sponsored posts, job ads, event promotions and minor updates.
- Merge duplicates: when several sources cover the same story, write one entry and link
  the primary source (the original announcement or article, not the aggregator).
- If fewer than 3 items are worth reporting, do not publish. End the run as described
  in step 5.
- Write each entry in your own words. Never copy paragraphs from a source.

Use this format for the post body (Markdown):

```
A quick roundup of developer news since the last digest.

## <Topic, e.g. Releases, Security, AI & tooling, Industry>

- **<Short headline>**: <one or two plain sentences on what happened and why it matters>. [<Source name>](<url>)

...

---

*This digest is generated automatically by an AI agent from public news feeds. Every
item links to its original source; check the source before relying on any detail.*
```

Group entries under 2 to 5 topic headings. Keep the whole post readable in about two
minutes. Use the most specific URL available for each item.

## 4. Publish

- Title: `Dev News Digest: <D Mon YYYY>, <HH:00>` using the current Europe/Stockholm
  time rounded to the nearest hour (for example `Dev News Digest: 7 Oct 2026, 12:00`).
- Build the JSON body with Python's `json` module (never by string concatenation) and
  write it to a file:
  `{"article": {"title": ..., "body_markdown": ..., "published": true, "tags": ["news", "programming", "webdev", "ai"]}}`
- `POST https://dev.to/api/articles` with curl (`-A "Mozilla/5.0 (news-digest bot)"`,
  `--data @<file>`) and the headers `api-key`, `Content-Type: application/json` and
  `Accept: application/vnd.forem.api-v1+json`.
- On HTTP 429, wait 30 seconds and retry once. Any other non-2xx response is a failure;
  do not retry.

## 5. Finish

End the session with a short final message, because it becomes the phone notification:

- Success: `Published: <article url> (<n> items)` followed by the headlines as a list.
- Nothing to post: `Skipped: only <n> new items since <cutoff>.`
- Failure: `FAILED: <what failed and the HTTP status>.`

Then list any feeds that failed, one per line.
