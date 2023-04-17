---
tags: f/index
filepath: <% '"' + tp.file.path(true) + '"' %>
---
<%* 
/*
MARKLISMA - A Markdown Link Structure Maker, for use with Obsidian.md.
---> WARNING: THIS FILE IS PART OF THE MARKLISMA SCRIPT. CHECK FOR MORE INFO, LICENSE AND UPDATES IN THE REPOSITORY PAGE AT https://github.com/samysberg/marklisma .
*/
var currDirPath = await tp.file.folder(true)

if (currDirPath == "/") {
  currDirPath = ""
}

var getParentDirName = function(n) {
// IF n is "", this funcion would throw an error (by trying to get root's folder parent).
// n argument may be "" when this argument is a "p.file.folder" of a root's file
  if (n === "") {
// so, in this case the function just returns an empty string.
  return ""
  } else {
  return app.vault.getAbstractFileByPath(n).parent.name
  }
}

const lineCount = function(txt) {
    if (!txt) {
        return 0;
    }    
    return txt.split(/\n/).length -3;
}

const siblingsFiles = DataviewAPI.markdownTable(["Filename", "Folder", "Path"], (DataviewAPI.pages(`"${currDirPath}"`)
  .where(p => p.file.folder === currDirPath)
  // .where(p => p.file.name !== "index" && p.file.name !== "Untitled") // uncomment this line to test manual file creation from this template
  .where(p => p.file.name !== "index" )
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