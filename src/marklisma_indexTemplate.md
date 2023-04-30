---
tags: f/index
filepath: <% '"' _%> <% tp.file.path(true)%> <%_ '"' %>
---
<%* 
/*
MARKLISMA - A Markdown Link Structure Maker, for use with Obsidian.md.
---> WARNING: THIS FILE IS PART OF THE MARKLISMA SCRIPT. CHECK FOR MORE INFO, LICENSE AND UPDATES IN THE REPOSITORY PAGE AT https://github.com/samysberg/marklisma .
*/
var currDirPath = await tp.file.folder(true)

if (currDirPath == "/") {
// templater's `tp.file.folder(true)` returns "/" for the root folder, however this string cant be used in the comparison with `p.file.folder.toString()` (at the `siblingsFiles` function block). so, current block makes a sanitized/compatible `currDirPath` , named as `dvCompatCurrDirPath`.
  var dvCompatCurrDirPath = ""
} else {
  var dvCompatCurrDirPath = currDirPath
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

const siblingsFiles = await DataviewAPI.markdownTable(["Filename", "File's Folder", "Folder Path"], (DataviewAPI.pages(`"${currDirPath}"`)
  .where(p => p.file.folder === dvCompatCurrDirPath)
  .where(p => p.file.name !== "index" )
  .map(p => [p.file.link, getParentDirName(p.file.path), p.file.folder])));

const offspringMdTable = await DataviewAPI.markdownTable(["Filename", "File's Folder", "Folder Path"], (DataviewAPI.pages(`"${currDirPath}"`)
  .where(p => p.file.folder !== currDirPath)
  .sort(p => p.file.folder)
  .map(p => [p.file.link, getParentDirName(p.file.path), p.file.folder])));
-%>

## Files in this Folder:

<%*
if (lineCount(siblingsFiles) > 0) {
  tR += siblingsFiles
} else {
  tR += "No other files found in this folder."
}
%>

## Offspring:

<%*
if (lineCount(offspringMdTable) > 0) {
  tR += offspringMdTable
} else {
  tR += "No Offspring files found."
}
%>