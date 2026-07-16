# Chapter 2 Scratchpad


## 2.2 Exact Match

**Edit Fields**

- Name: `Evaluation`
- `chatInput` String: `{{ $json.question }}`
- `sessionId`String: `{{$now.toMillis()}}`
- Include Other Input Fields: `On`, `All`

---

**Switch**

```{{ $('Evaluation').item.json.id }}``` – `Number` --> exists

---

**Edit Fields**
- Name: `Number of sources`
- `source_count` String:
```
{{
  /Sources:\s*\r?\n/.test($json.output)
    ? $json.output
        .split(/Sources:\s*\r?\n/)[1]
        .split(/\r?\n/)
        .filter(line => line.trim() !== '')
        .length
    : 0
}}
``` 

---

**Set Metrics**
- `Categorization`
- Expected Answer: `{{ $('Evaluation').item.json.expected_source_count }}`
- Actual Answer: `{{ $json.source_count }}`

Option:
- Metric name: `eval_source_count`

## 2.3 Similarity Match

**Edit Fields**
- Name: `Source names`
- `source_names` String:

```
{{
  (() => {
    const match = $json.output.match(/Sources:\s*\r?\n([\s\S]*)$/);

    if (!match) return [""];

    const sources = match[1]
      .split(/\r?\n/)
      .map(source => source.trim())
      .filter(Boolean)

      // Remove source reference prefixes like "- ^1", "^2", or "- ^-1"
      .map(source => source.replace(/^[-*]?\s*\^-?\d+\s*/, '').trim())

      // Replace Markdown links with only the visible link text
      .map(source => source.replace(/\[([^\]]+)\]\([^)]+\)/g, '$1').trim())

      // Remove trailing page info like ", page N/A"
      .map(source => source.replace(/,\s*page\s+N\/A$/i, '').trim())

      // Remove anything that became empty after cleanup
      .filter(Boolean)

      // Sort alphabetically, case-insensitive
      .sort((a, b) => a.localeCompare(b, undefined, { sensitivity: 'base' }));

    return sources.length ? sources : [""];
  })()
}}
```

---

**Set Metrics**

- Metric: `String Similarity`
- Expected Answer: `{{ $('Evaluation').item.json.expected_sources }}`
- Actual Answer: `{{ $json.source_names }}`
- Option:
	- Metric Name: `eval_source_match` 