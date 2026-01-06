# Workflow
n8n = automation platform -> web or docker
postgresql = database -> local or docker

### 1 page
**table schema**
- **titles**: id, name
- **contents**: id, titles_id, page, data

**n8n flow:**
1. POST to website -> return HTML
2. distribute HTML boxes to a table
3. parse into text & divide (titles, sources, content) into columns

**postgres flow (loop for every title):**
1. insert title & get title_id
2. insert content (title_id, page, data)

### All Genesis
**table schema**
- **books**: books_id, number, chapter, verse
- **titles**: titles_id, books_id, name
- **contents**: contents_id, titles_id, page, data

**flow**
1. CODE generate book, chapter, verse
2. LOOP for every combination: (book, chapter, verse)
3. POSTGRES insert (book, chapter, verse) -> table BOOKS `return verse_id`
4. HTTP post request using (book, chapter, verse)
5. HTML extract main content
6. HTML parse into tables (title, page array, data array)
8. POSTGRES insert titles + verse_id -> table TITLES `return titles_id`
9. expand array'd cells into independent rows
10. POSTGRES insert all rows + titles_id -> table CONTENTS
> useful debugging tool using 2 nodes = RESET (delete all tables -> create all tables)
