# Globalization (G11n)

`offer-service` uses direct integration into translation services in order to fulfill translation requirements for
dynamic text content. In lower environment, and where appropriate, the default language for copy is English.

## Translation Schema

```postgresql
CREATE TABLE translations (
      id varchar,                                    -- copy container id. E.g. offer id
      path varchar,                                  -- json path to copy
      language_code varchar,                         -- canonicalized language code
      updated TIMESTAMPTZ NOT NULL DEFAULT NOW(),    -- last time this entry was changed
      copy_contents text[2],                         -- [live_translation, most_recent_translation]
      PRIMARY KEY (id, path, language_code)
);
```

## Translation Flow

### Request New Translation

**`offer-service`**

1. Save Offer
2. Compare field copy to the most recent translation
3. If copy matches most recent english request, stop, otherwise continue
4. Submit translation request using `TranslationServiceClient`

***

### Update Translations

**`offer-service-g11n-worker`**

1. Listen for incoming translation completions
2. Split id on first `.` as `{offerId}.{path}`
3. Set `copy_contents[1]` to new translated value for `id = {offerId} AND path = {path}` for given language

***

### Push Recent Translations to Live

**`offer-service-translations-worker`**

1. Query all translation entries for given offer id
2. Check that all non-English variants have `updated > en.updated`
3. If they do, set `copy_contents[0] = copy_contents[1]` for all `id = {offerId}` entries
