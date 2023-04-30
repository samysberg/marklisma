---
tags: f/moc
filepath: <% '"' _%> <% tp.file.path(true)%> <%_ '"' %>
---
<%*
/*
MARKLISMA - A Markdown Link Structure Maker, for use with Obsidian.md.
---> WARNING: THIS FILE IS PART OF THE MARKLISMA SCRIPT. CHECK FOR MORE INFO, LICENSE AND UPDATES IN THE REPOSITORY PAGE AT https://github.com/samysberg/marklisma .
*/
var currDirPath = await tp.file.folder(true)
var childDirsArr = await app.vault.getAbstractFileByPath(`${currDirPath}`).children.filter(p => p.children).map(p => p.path)

if (currDirPath == "/") {
// As Templater's `tp.file.folder(true)` returns "/" for the root folder, and this "/" string can't be used in the comparison with `p.file.folder.toString()` (at the `siblingsFiles` function block). so, current block makes a sanitized/compatible `currDirPath` , named as `dvCompatCurrDirPath`.
  var dvCompatCurrDirPath = ""
} else {
  var dvCompatCurrDirPath = currDirPath
}

var getParentDirDvObj = function(n) {
// IF n is "", this funcion would throw an error (by trying to get root's folder parent).
// n argument may be "" when this argument is a "p.file.folder" of a root's file
  if (n === "") {
// so, in this case the function just returns an empty string.
  return ""
  } else {
  return app.vault.getAbstractFileByPath(n).parent
  }
}

const lineCount = function(txt) {
    if (!txt) {
        return 0;
    }
    return txt.split(/\n/).length -3;
}

const siblingsFiles = await DataviewAPI.markdownTable(["Filename", "File's Folder", "Folder Path"], (DataviewAPI.pages(`"${currDirPath}"`)
  .where(p => p.file.folder.toString() === dvCompatCurrDirPath)
  .where(p => p.file.name !== "MOC")
  .map(p => [p.file.link, getParentDirDvObj(p.file.path).name, p.file.folder])));

const childStructMdTable = await DataviewAPI.markdownTable(["Filename", "File's Folder", "Folder Path"], (DataviewAPI.pages(`"${currDirPath}"`)
  .where(p => getParentDirDvObj(p.file.folder).path === currDirPath)
  .where(p => p.file.name === "MOC" || p.file.name === "index")
  .map(p => [p.file.link, getParentDirDvObj(p.file.path).name, p.file.folder])));
-%>

## Files in this Folder:

<%*
if (lineCount(siblingsFiles) > 0) {
  tR += siblingsFiles
} else {
  tR += "No other files found in this folder."
}
%>

## Children Folders Structure Files:

<%*
if (lineCount(childStructMdTable) > 0) {
  tR += childStructMdTable
} else {
  tR += "No Children Folders Structure Files found."
}
%>