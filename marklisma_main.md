<%*
/*
MARKLISMA
A Markdown Link Structure Maker, for use with Obsidian.md.

---> WARNINGS:
- THIS CODE WILL PROBABLY NOT PROPERLY WORK WITHOUT HACKING TEMPLATER'S MAX_DEPTH VARIABLE, IN THE RUNNING .obsidian FOLDER. HELP IS NEEDED TO MAYBE MAKE THE HACK UNNECESSARY.
- THIS IS EXPERIMENTAL CODE, NOT THOROUGHLY TESTED, AND IT MAY LEAD DO DATA LEAK, DAMAGE, OR LOSS.
- THIS CODE IS INTENTED TO RUN AS A SCRIPT (WITH THE TEMPLATER PLUGIN) IN THE OBSIDIAN.MD APPLICATION.
- CHECK FOR MORE INFO, LICENSE AND UPDATES IN THE REPOSITORY PAGE AT https://github.com/samysberg/marklisma .
- REMEMBER TO CHANGE THE PATH OF THE VARIABLES "ixTmt" AND "mocTmt" TO SUIT YOUR USE CASE.
*/

// block below: Obsdian's "abstract path" of the index and MOC templater templates:
var ixTmt = await app.vault.getAbstractFileByPath(
  "templater/marklisma_indexTemplate.md"
);
var mocTmt = await app.vault.getAbstractFileByPath(
  "templater/marklisma_mocTemplate.md"
);

// block below: sorted arrays of folder paths (as strings).
// "sort((b,a)..." is intentional, in order do have the furthest folders parsed before their parents.
var allDirsArr = this.app.vault
  .getAllLoadedFiles()
  .filter((i) => i.children)
  .flatMap((folder) => folder.path)
  .sort((b, a) =>
    a.localeCompare(b, "en-us", {
      ignorePunctuation: false,
      numeric: true,
      sensitivity: "variant",
      caseFirst: "upper",
    })
  );
var ixSrcDirsArr = await DataviewAPI.pages()
  .where((p) => p.file.name == "index" && p.file.ext == "md")
  .file.folder.values.sort((b, a) =>
    a.localeCompare(b, "en-us", {
      ignorePunctuation: false,
      numeric: true,
      sensitivity: "variant",
      caseFirst: "upper",
    })
  );
var mocSrcDirsArr = await DataviewAPI.pages()
  .where((p) => p.file.name == "MOC" && p.file.ext == "md")
  .file.folder.values.sort((b, a) =>
    a.localeCompare(b, "en-us", {
      ignorePunctuation: false,
      numeric: true,
      sensitivity: "variant",
      caseFirst: "upper",
    })
  );

// block below: initializing some vars.
var mocTgtDirsArr = [];
var badHierLog = [];

// block below: just a sleep function.
function sleep(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

// block below: checks if a string starts with any of the array elements.
const chkStrStartsWithArr = function (s, arr) {
  if (arr.some((w) => s.startsWith(w))) {
    return true;
  } else {
    return false;
  }
};

// block below: counts how many array elements (from index dir array) matches with start of a string (a given index dir).
//  (if > 2 matches, the "s" path is nested inside de arr)
// TODO: use "let" below?
const countStrStartsWithArr = function (s, arr) {
  let counter = 0;
  for (let i = 0; i < arr.length; i++) {
    let h = arr[i];
    if (s.startsWith(h)) {
      counter += 1;
    }
  }
  return counter;
};

// block below: make array of folders (by filtering out index file paths) in which to create MOC file.
// t = array of tree (all) dirs // n = array of negated dirs
const mkTgtMocDirArr = function (t, n) {
  for (let z of t) {
    if (!chkStrStartsWithArr(z, n)) {
      mocTgtDirsArr.push(z);
    }
  }
};

// block below: hierarchy tree error checks.
const chkBadSrcHier = function (ida, mda) {
  // mda: index dir array // mda: moc dir array
  // 1st check: are there indexes within indexed folder?
  for (let i = 0; i < ida.length; i++) {
    if (countStrStartsWithArr(ida[i], ida) > 1) {
      badHierLog.push(ida[i] + " has an index file inside indexed tree");
    }
  }
  // 2nd check: Are there MOC sons/brothers of index?
  for (let j = 0; j < mda.length; j++) {
    if (countStrStartsWithArr(mda[j], ida) > 0) {
      badHierLog.push(mda[j] + " has a MOC file inside indexed tree");
    }
  }
  if (badHierLog.length < 1) {
    return true;
  } else {
    return false;
  }
};

// block below: makes new index files (replacing the previously existing ones).
const mkNewIndexFile = async function (a) {
  // a: string = folder path (which contains index file).
  // NOTE THAT index files may already have frontmatter from creation. This function just updates the existing frontmatter).
  // NOTE THAT new index files are not created by this script; it acts only updating the existing ones (despite using tp.file.create_new for that).
  let ixTgtFiAbp = await app.vault.getAbstractFileByPath(a + "/index.md");
  var previousFmObj = {};
  // These frontmatter operations are pretty much the same as in the mkNewMocFile function, where they are commented.
  await app.fileManager.processFrontMatter(ixTgtFiAbp, (fm) => {
    previousFmObj = fm;
  });
  let prevFmObjTrimd = await Object.fromEntries(
    Object.entries(previousFmObj).filter(
      ([k, v]) => v != null && !k.startsWith("date")
    )
  );
  prevFmObjTrimd["date updated"] = `${tp.date.now("yyyy-MM-DD, HH:mm, dddd.")}`;
  prevFmObjTrimd["disabled rules"] = "all";
  let ixTgtDir = ixTgtFiAbp.parent.path;
  let ixTgtDirAbp = await app.vault.getAbstractFileByPath(
    ixTgtFiAbp.parent.path
  );
  await app.vault.trash(ixTgtFiAbp);
  await tp.file.create_new(ixTmt, "index", false, ixTgtDirAbp);
  var newIxFileAbp = await app.vault.getAbstractFileByPath(
    ixTgtDir + "/index.md"
  );
  await app.fileManager.processFrontMatter(newIxFileAbp, (fm) => {
    Object.assign(fm, prevFmObjTrimd);
  });
};

// block below: returns string of MOC path (usable for app.vault.getAbstractFileByPath).
const callAbpStrFix = function (d) {
  // d: string = folder path (which contains MOC file).
  if (d === "/") {
    let fxstr = `MOC.md`;
    return `${fxstr}`;
  } else {
    let fxstr = `${d}/MOC.md`;
    return `${fxstr}`;
  }
};

// block below: makes new MOC files along the tree (including new MOC's as replacements for the previously existing ones).
const mkNewMocFile = async function (d) {
  // TODO : CHECK if array is in proper order or if it needs to get sorted.
  // d: string = directory path (where to put te MOC in).
  var mdabp = await app.vault.getAbstractFileByPath(d);
  if (mdabp.children.map((p) => p.name).some((val) => "MOC.md" === val)) {
    // IF: There is already a MOC in this folder.
    var fixdMocAbpStr = callAbpStrFix(d);
    var mfabp = await app.vault.getAbstractFileByPath(fixdMocAbpStr);
    var previousFmObj = {};
    // It saves the YAML FM (frontmatter) keys/values in the "previousFmObj" var, then restores (most of) them to the replacement MOC.
    await app.fileManager.processFrontMatter(mfabp, (fm) => {
      previousFmObj = fm;
    });
    // It deletes the FM keys with value = null, and FM keys starting with "date" word, as they usually rely on filesystem metadata, which wouldn't be accurate for this use case. However, it preserves other keys/values.
    var prevFmObjTrimd = Object.fromEntries(
      Object.entries(previousFmObj).filter(
        ([k, v]) => v != null && !k.startsWith("date")
      )
    );
    // The new file FM will have key/value for "date updated" (which will get deleted and replaced on the next run), and "disabled rules" (to avoid have the Linter plugin mingling with de FM).
    prevFmObjTrimd["date updated"] = `${tp.date.now(
      "yyyy-MM-DD, HH:mm, dddd."
    )}`;
    prevFmObjTrimd["disabled rules"] = "all";
    await app.vault.trash(mfabp);
    await tp.file.create_new(mocTmt, "MOC", false, mdabp);
    var newMocFileAbp = await app.vault.getAbstractFileByPath(d + "/MOC.md");
    await app.fileManager.processFrontMatter(newMocFileAbp, (fm) => {
      Object.assign(fm, prevFmObjTrimd);
    });
  } else {
    // ELSE: There is no MOC in this folder yet.
    await tp.file.create_new(mocTmt, "MOC", false, mdabp);
    var fixdMocAbpStr = callAbpStrFix(d);
    var newMocFileAbp = await app.vault.getAbstractFileByPath(fixdMocAbpStr);
    await app.fileManager.processFrontMatter(newMocFileAbp, (fm) => {
      fm["date updated"] = `${tp.date.now("yyyy-MM-DD, HH:mm, dddd.")}`;
      fm["disabled rules"] = "all";
    });
  }
};

// block below: main function.
// HELP NEEDED: there is probably a cleaner way to write this, without needing this sleep function. However this "fix" was needed in order to avoid contents of a target moc/index going into another moc/index.
const mkHierStruct = async function () {
  // IF: initial file hierarchy is ok, iterate over index and MOC folder arrays to make/update proper files. 
  if (chkBadSrcHier(ixSrcDirsArr, mocSrcDirsArr)) {
    for (let f of ixSrcDirsArr) {
      await sleep(250);
      mkNewIndexFile(f);
      await sleep(250);
    }
    console.log("Marklisma has finished with index files.")
    mkTgtMocDirArr(allDirsArr, ixSrcDirsArr);
    for (let d of mocTgtDirsArr) {
      await sleep(250);
      mkNewMocFile(d);
      await sleep(250);
    }
    console.log("Marklisma has finished with MOC files.")
    // ELSE: show hierarchy errors on console.log .
  } else {
    console.log("errors found! \n" + badHierLog.map((p) => p + "\n"));
  }
};
mkHierStruct();
%>