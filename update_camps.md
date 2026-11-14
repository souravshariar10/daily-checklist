# Updating the creator campaigns

The site reads `camps.json` every time it loads (cache-busted), so to update Red's campaign list edit that file, commit and push. He does not need to do anything.

## Format

```json
{
  "updated": "2026-11-15T02:30:00+06:00",
  "platforms": {
    "creator_wire": [{"id": "camp_gecko", "n": "Gecko Terminal", "url": "https://...", "end": "2026-11-26"}],
    "nucleus": [], "kaito": [], "rarelist": [], "echohunt": []
  },
  "removed": ["camp_old_id"],
  "removed_names": [{"platform": "creator_wire", "name": "atoma"}]
}
```

- Platform keys are the item ids in `data` inside index.html: `creator_wire`, `nucleus`, `kaito`, `rarelist`, `echohunt`.
- `id` must stay stable for a campaign forever (it keys Red's own dates and dismissals). New ids: `camp_<short_name>`.
- `n` is the shown name, `url` is the Open link, and `end` is `YYYY-MM-DD` or `""` if unknown. Use the real end date only.
- `removed` takes campaigns out of Red's list by id.
- `removed_names` runs once per browser. It removes Red's own hand-added campaigns whose name contains the text, under that platform.

## What the page does

1. A missing campaign is added to the end of its platform. If Red already added one by hand with the same link or name, that one is taken over instead of making a duplicate.
2. Name and link are kept in sync with the file.
3. Dates: a non-empty Ends date that Red set himself in Customize wins. Otherwise the date comes from `end`, then from `DEFAULT_DEADLINES`.
4. If Red deletes a managed campaign in Customize, it goes into the `redcheck_dismissed_camps` list and is not added back. Reset to default clears that list.
5. Campaigns are hidden from the day after their end date from the main list but stay visible in Customize. Ended campaigns are not added again.
6. If the fetch fails, the last fetched copy (`redcheck_remote_camps`) and then the built-in defaults are used.

## Checklist for an update

1. Edit `camps.json` and bump `updated`. Move ended campaigns out of `platforms` and into `removed` once they are about 3 days past their end.
2. Keep `DEFAULT_CAMPS` and `DEFAULT_DEADLINES` in index.html roughly in sync, as the offline fallback.
3. Check it: `python3 -m json.tool camps.json`
4. Commit, push, then confirm it is live: `curl -s "https://souravshariar10.github.io/daily-checklist/camps.json?t=$(date +%s)"`

Do not touch the watermark, the Google Analytics tag (G-86S790XD9G), or the rule that keeps My tasks out of the progress, streak and calendar.

## Sources for dates (public, no login)

- Rarelist: `https://rarelist.io/api/v1/campaigns/public` (`planned_end_date` / `end_date`, `status`)
- Nucleus: `https://api.nucleus.codes/v1/projects/get_all_carousels` (`end_time`, `status`)
- EchoHunt: `https://app.echohunt.ai/api/echohunt/campaigns?lang=en` (`endAt`, `status`)
- Kaito Studio and Creator Wire need a login. Use official X posts and Red's briefs.
