---
tags: sometag
filepath: "<% tp.file.path(true) %>"
---
<%* 
/*
MARKLISMA - A Markdown Link Structure Maker, for use with Obsidian.md.
---> WARNING: THIS FILE IS PART OF THE MARKLISMA SCRIPT. CHECK FOR MORE INFO, LICENSE AND UPDATES IN THE REPOSITORY PAGE AT https://github.com/samysberg/marklisma .
*/
var currDirPath = await tp.file.folder(true)

var getParentDirName = function(n) {
  return app.vault.getAbstractFileByPath(n).parent.name
}

const lineCount = function(txt) {
    if (!txt) {
        return 0;
    }    
    return txt.split(/\n/).length -3;
}

const siblingsFiles = DataviewAPI.markdownTable(["Filename", "Folder", "Path"], (DataviewAPI.pages(`"${currDirPath}"`)
  .where(p => p.file.folder === currDirPath)
  .where(p => p.file.name !== "index" && p.file.name !== "Untitled")
  .map(p => [p.file.link, getParentDirName(p.file.path), p.file.folder])));

const offspringMdTable = DataviewAPI.markdownTable(["Filename", "Folder", "Path"], (DataviewAPI.pages(`"${currDirPath}"`)
  .where(p => p.file.folder !== currDirPath)
  .sort(p => p.file.folder)
  .map(p => [p.file.link, getParentDirName(p.file.path), p.file.folder])));
_%>

## Files in this Folder:

<%* if (lineCount(siblingsFiles) > 0) { %>
<%- siblingsFiles -%>
<%* } else { %>
No other files found in this folder.
<%* } %>

## Offspring:

<%* if (lineCount(offspringMdTable) > 0) { %>
<%- offspringMdTable -%>
<%* } else { %>
No Offspring files found.
<%* } %>