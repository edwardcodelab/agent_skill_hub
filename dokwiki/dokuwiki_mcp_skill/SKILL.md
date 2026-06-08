---
name: dokuwiki-mcp
description: How to use the DokuWiki MCP server for reading, writing, and managing wiki pages and media.
category: documentation
---

# dokuwiki MCP Usage Guide

Use this skill when you need to interact with the dokuwiki DokuWiki instance via MCP.

## When to Use
- Reading or writing wiki pages (`coding:*` namespace or others)
- Searching wiki content
- Managing media/attachments
- Checking ACLs or user permissions
- Bulk operations on pages or media

## Prerequisites
- dokuwiki MCP server must be enabled in `~/.hermes/config.yaml`
- Run `hermes mcp ls` to confirm the server appears with status ✓
- Run `hermes mcp test dokuwiki` to verify connectivity

## Core Tools (Most Used)

### Page Operations
| Tool | Purpose | Example |
|------|---------|---------|
| `mcp_dokuwiki_core_getPage` | Read page content | `page: "shopping:food_drink:starbuck_menu"` |
| `mcp_dokuwiki_core_savePage` | Create or update page | `page`, `text`, optional `summary` |
| `mcp_dokuwiki_core_appendPage` | Append to existing page | `page`, `text` |
| `mcp_dokuwiki_core_searchPages` | Full-text search | `query: "starbucks"` |
| `mcp_dokuwiki_core_getPageHTML` | Get rendered HTML | `page` |

### Media Operations
- `mcp_dokuwiki_core_getMedia`
- `mcp_dokuwiki_core_saveMedia`
- `mcp_dokuwiki_core_listMedia`

### Utility
- `mcp_dokuwiki_core_whoAmI` — Check current user
- `mcp_dokuwiki_core_getWikiTitle`
- `mcp_dokuwiki_core_getRecentPageChanges`

## Common Workflows

### 1. Create or Update a Page
```python
# Recommended pattern
content = """====== Page Title ======
Your DokuWiki markup here...
"""
result = mcp_dokuwiki_core_savePage(
    page="coding:namespace:page_name",
    text=content,
    summary="Updated via agent"
)
# result == "1" means success
```

### 2. Read a Page First (Safe Update)
Always read before overwriting to avoid data loss:
1. `mcp_dokuwiki_core_getPage(page=...)`
2. Modify content
3. `mcp_dokuwiki_core_savePage(...)`

### 3. Handle Large Content
**Pitfall**: Large pages can cause timeout (300s limit).

**Solution**:
- Split content into multiple `savePage` calls
- Or use `appendPage` for incremental updates
- Keep individual saves under ~8k–10k characters when possible

### 4. Search
```python
results = mcp_dokuwiki_core_searchPages(query="starbucks menu")
```

## Important Pitfalls

- **Namespace**: Always use full page ID (e.g. `coding:shopping:food_drink:starbuck_menu`)
- **DokuWiki syntax**: Use `^` for table headers, `|` for cells, `======` for titles.
- **No credentials in prompts**: Never log tokens or passwords.
- **Media uploads**: Use base64 for `saveMedia`.
- **ACLs**: Some pages may require specific user permissions.

## Verification
After writing:
1. Call `mcp_dokuwiki_core_getPage` to confirm content
2. Ask user to visually verify on the wiki site
3. Check `mcp_dokuwiki_core_getPageHistory` for revision count

## Related Skills
- `hermes-agent-skill-authoring` — For creating new skills
- `dokuwiki` related patterns in other skills


## Dokuwiki syntax
convert the input text as dokuwiki format, follow dokuwiki format as below:
Basic Text Formatting
DokuWiki supports **bold**, //italic//, __underlined__ and ''monospaced'' texts.
Of course you can **__//''combine''//__** all these.


This is some text with some linebreaks
Note that the two backslashes are only recognized at the end of a line
or followed by a whitespace \\this happens without it.

This is some text with some linebreaks\\ Note that the
two backslashes are only recognized at the end of a line\\
or followed by\\ a whitespace \\this happens without it.
You should use forced newlines only if really needed.

Links
DokuWiki supports multiple ways of creating links. External links are recognized
automagically: http://www.google.com or simply www.google.com - You can set
link text as well: [[http://www.google.com|This Link points to google]]. Email


Footnotes
You can add footnotes ((This is a footnote)) by using double parentheses.

Headlines
====== Headline Level 1 ======
===== Headline Level 2 =====
==== Headline Level 3 ====
=== Headline Level 4 ===
== Headline Level 5 ==
By using four or more dashes, you can make a horizontal line:
----

Media Files
You can include external and internal images, videos and audio files with curly brackets. Optionally you can specify the size of them.

Real size:                        {{wiki:dokuwiki-128.png}}
Resize to given width:            {{wiki:dokuwiki-128.png?50}}
Resize to given width and height: {{wiki:dokuwiki-128.png?200x50}}
Resized external image:           {{https://www.php.net/images/php.gif?200x50}}
By using left or right whitespaces you can choose the alignment.


{{ wiki:dokuwiki-128.png}}
{{wiki:dokuwiki-128.png }}
{{ wiki:dokuwiki-128.png }}
Of course, you can add a title (displayed as a tooltip by most browsers), too.
By adding ?linkonly you provide a link to the media without displaying it inline

{{wiki:dokuwiki-128.png?linkonly}}
dokuwiki-128.png This is just a link to the image.

Lists
Dokuwiki supports ordered and unordered lists. To create a list item, indent your text by two spaces and use a * for unordered lists or a - for ordered ones.

  * This is a list
  * The second item
    * You may have different levels
  * Another item

you can create table like this :
^ Heading 1      ^ Heading 2       ^ Heading 3          ^
| Row 1 Col 1    | Row 1 Col 2     | Row 1 Col 3        |
| Row 2 Col 1    | some colspan  ||
| Row 3 Col 1    | Row 3 Col 2     | Row 3 Col 3        |



## Quick Reference
- Start page: `start`
- Common namespace: `coding:`
- Success indicator: `{"result": "1"}` from `savePage` / `appendPage`
