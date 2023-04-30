---
tag: marklismafile
---

# General Status

## Files with "index" filename

- Initial vault state:
	- documents/family/logs/inuse/index.md
	- documents/family/reference/index.md
	- documents/personal/logs/inuse/digital/index.md
	- documents/work/logs/index.md
	- errortest/indexBeforeMoc/index.md
	- errortest/indexWithinIndex/index.md
	- errortest/indexWithinIndex/indexedFolder/index.md

- Current query state:
```dataviewjs
dv.table(["Filename Link", "Path"],
  dv.pages()
    .where(p => p.file.name === "index")
    .map(p => [p.file.link, p.file.folder]));
```

## Files with "MOC" filename

- Initial vault state:
	- documents/personal/logs/inuse/picture/MOC.md
	- documents/personal/reference/MOC.md
	- documents/work/MOC.md
	- errortest/indexBeforeMoc/mocFolder/MOC.md

- Current query state:
```dataviewjs
dv.table(["Filename Link", "Path"],
  dv.pages()
    .where(p => p.file.name === "MOC")
    .map(p => [p.file.link, p.file.folder]));
```

# Error check queries

## Nested "index" files

- Initial vault state: "errortest/indexWithinIndex/indexedFolder/index.md"
```dataviewjs

const nestedIndexFiles = [];

const indexedFolders = dv.pages()
.where(p => p.file.name === "index").file.folder

const indexFilesPath = dv.pages()
.where(p => p.file.name === "index").file.path

// Check if the "s" string starts with any
const countStrStartsWithArr = function(s, arr) {
  let counter = 0;
  for (let i = 0; i < arr.length; i++){
    let h = arr[i];
    if ( s.startsWith(h) ) {
      counter += 1;
    } 
  }
  return counter;
}

const chkNestedIndex = function(ifa, ida) {
// ifa: index file array // ida: index dir array
  for (let i = 0; i < ida.length; i++) {
    if ( countStrStartsWithArr(ifa[i], ida) > 1 ) {
      nestedIndexFiles.push(ifa[i])
    }
  }
}

chkNestedIndex(indexFilesPath, indexedFolders)
dv.paragraph("- Current query state: " + `"${nestedIndexFiles}"`)

```

## MOC within indexed tree

- Initial vault state: "errortest/indexBeforeMoc/mocFolder/MOC.md"
```dataviewjs

const mocWithinIndexed = [];

const indexedFolders = dv.pages()
.where(p => p.file.name === "index").file.folder

const mocFilesPath = dv.pages()
.where(p => p.file.name === "MOC").file.path

// Check if the "s" string starts with any
const countStrStartsWithArr = function(s, arr) {
  let counter = 0;
  for (let i = 0; i < arr.length; i++){
    let h = arr[i];
    if ( s.startsWith(h) ) {
      counter += 1;
    } 
  }
  return counter;
}

const chkMocWithinIndexed = function(mfa, ida) {
// mfa: moc file array // ida: index dir array
  for (let i = 0; i < mfa.length; i++) {
    if ( countStrStartsWithArr(mfa[i], ida) > 0 ) {
      mocWithinIndexed.push(mfa[i])
    }
  }
}

chkMocWithinIndexed(mocFilesPath, indexedFolders)
dv.paragraph("- Current query state: " + `"${mocWithinIndexed}"`)
```

# Edge cases

## Empty FM

[[documents/work/reference/Fugiat Elit Aliqua Cillum.md]]
- Initial vault state:
```
---
---
```

[[documents/work/reference/Magna Excepteur.md]]
- Initial vault state:
```
---

---
```

## Empty FM line

[[documents/personal/reference/picture/Est Dolor.md]]
- Initial vault state:
```
---
tags: preexisting/first preexisting/second
aliases: 
publish: true
status: Done

---
```

# Feature checks

## FM mingling

- Pre-existing frontmatter SHOULD BE KEPT THE SAME in [[documents/personal/reference/picture/Est Dolor.md]]
- Pre-existing frontmatter SHOULD BE MODIFIED at  [[documents/family/reference/index.md]] and [[documents/personal/reference/MOC.md]] (initial vault state):
```
---
tags: preexisting/first preexisting/second
aliases: 
publish: true
status: Done
dependencies:
  - coderunner
  - libwhatever
due: 2023-05-04
stakeholders: [lucy, khalil, hannah, mike]
weight: 67
real date created: 2023-04-12, 08:20.
date created: 2023-04-12, 09:30.
date updated: 2023-04-12, 14:30.
---
```

**⚠  CHECK the post-marklisma FM status in the files.**


## Full index
- regular pre-existing/replaced index, with children and offspring.

[[documents/family/logs/inuse/index]]

## Index with no sibling

[[documents/family/reference/index]]

## Index with no offspring (but with child folder)

[[documents/personal/logs/inuse/digital/index]]

## Index limiting MOC recursive generation

[[documents/work/logs/index]]

## Full MOC
- regular pre-existing/replaced MOC, with children and siblings, with children index.

[[documents/work/MOC]]

## MOC with no sibling

[[documents/personal/reference/MOC]]

## MOC with no children structure files (but with child folder)

[[documents/personal/logs/inuse/picture/MOC]]
