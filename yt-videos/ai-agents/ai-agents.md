This file contains the code blocks and prompts used to build the AI agent in this video.

Please note that not every step from the video is included here. Most of the steps involve selecting, configuring, or connecting different elements and don't require anything to be shared. The sections below include the steps where code or prompts are required, listed in the same order they appear in the video.

## Filter Stories from the Last Two Days

```javascript
const rows = $input.all();

const now = new Date();
const cutoff = new Date(now);
cutoff.setDate(cutoff.getDate() - 2);

function canonicalizeUrl(rawUrl) {
try {
    const u = new URL(rawUrl);
    u.search = '';
    u.hash = '';
    let path = u.pathname.replace(/\/$/, '');
    return u.origin + path;
} catch (e) {
    return rawUrl;
}
}

const recentStories = rows
.filter(item => item.json && item.json.date) // skip empty/malformed items (e.g. the "Always Output Data" placeholder)
.filter(item => {
    const rowDate = new Date(item.json.date);
    return rowDate >= cutoff;
})
.map(item => ({
    url: canonicalizeUrl(item.json.url),
    headline: item.json.headline,
    summary: item.json.summary
}));

return [{ json: { recentStories } }];
```

## AI Agent Prompts

### User Prompt

```text
Give me today's AI news digest covering developments from the last 24 hours.

Stories already covered in the last 2 days (check your candidates against this list in Step 3):
{{ JSON.stringify($json.recentStories) }}
```

### System Prompt

```text
You are an AI news curator. Your job is to produce a concise daily digest of the most important AI development news from the last 24 hours.

STEP 1 — SEARCH
Use the Tavily search tool to find news. Do not rely on your own knowledge for current events, since it may be outdated.
- Favor results from established, reputable news and tech publications (e.g. major tech news outlets, wire services, mainstream business/tech press). Avoid low-quality aggregators, content farms, or unverified blogs.
- You have a STRICT MAXIMUM of 3 search calls total. Do not exceed this under any circumstances.
- Use your first 1-2 searches broadly (e.g. "AI news today") before narrowing to specific angles. Do not run a new search just to find one more source confirming a story you already have enough evidence for.
- After 3 searches, move on to Step 2 with whatever candidates you have, even if fewer than 6-8.

STEP 2 — DRAFT
From your search results, select the 5-6 most significant, distinct stories — prioritize the stories that are most widely covered or most impactful, not niche or narrow-interest items. Prioritize major model/product releases, significant funding rounds, regulatory developments, notable research breakthroughs, and major company announcements. Exclude opinion pieces and rumors without named sources. If multiple sources cover the same event, treat it as one story and keep the most authoritative source.

For each selected story, draft: headline (plain language, not the original title), a 2-3 sentence summary, source name, url, and date.

STEP 3 — CHECK AGAINST ALREADY-COVERED STORIES
You will be given a list of stories already covered in the last 2 days, each with a url, headline, and summary. Evaluate each of your drafted candidates independently against this list:
- If a candidate's url exactly matches one in the list, exclude it.
- If a candidate covers the same underlying event as one in the list — even with a different url, different outlet, or reworded headline — exclude it, unless there's a materially significant new development since it was last covered (e.g. "in talks" is now "confirmed"). In that case, include it and note what's new in the summary.
- If a candidate doesn't match anything in the list, keep it.
- Some days will have partial overlap. Judge each candidate on its own; don't exclude or include the whole set based on a few matches.
- If every candidate is excluded, that's a valid outcome — return an empty array.

Return between 0 and 6 objects. Never pad with weaker stories or duplicates to hit a target count.
```

## Structure the AI Agent Output

```json
{
"stories": [
    {
    "headline": "example headline",
    "summary": "example summary",
    "source": "example source",
    "url": "https://example.com",
    "date": "2026-07-28"
    }
]
}
```

## Process the Generated Stories

```javascript
const stories = $input.first().json.output.stories;

if (!stories || stories.length === 0) {
return [{ json: { hasStories: false, stories: [], sheetRows: [] } }];
}
;

return [{ json: { hasStories: true, stories } }];
```

## Format the Email Digest

```javascript
const stories = $json.stories;

const today = new Date().toLocaleDateString('en-US', { 
month: 'long', day: 'numeric', year: 'numeric' 
});

const storiesHtml = stories.map(s => `
<tr>
    <td style="padding: 0 0 16px 0; text-align: left;">
    <p style="font-size: 15px; color: #1a1a1a; line-height: 1.6; margin: 0; text-align: left;">
        <b>${s.headline}</b>: ${s.summary} <a href="${s.url}" style="color: #1a73e8;">${s.source}</a>
    </p>
    </td>
</tr>
`).join('');

const emailBody = `
<table width="100%" cellpadding="0" cellspacing="0" style="max-width: 650px; font-family: -apple-system, Segoe UI, Roboto, sans-serif;">
<tr>
    <td style="text-align: left;">
    <h2 style="color: #1a1a1a; border-bottom: 1px solid #ccc; padding-bottom: 10px; text-align: left; margin: 0 0 16px 0;">
        AI Daily Digest | ${today}
    </h2>
    <table width="100%" cellpadding="0" cellspacing="0">
        ${storiesHtml}
    </table>
    </td>
</tr>
</table>
`;

const subject = `AI News Digest — ${today}`;

return [{ json: { emailBody, subject } }];
```
