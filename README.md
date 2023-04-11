# marklisma

A Markdown Link Structure Maker, for use with Obsidian.md .

# WARNINGS

- This code will probably not properly work without hacking templater's max_depth variable, in the running .obsidian folder. help is needed to maybe make the hack unnecessary.
- This is experimental code, not thoroughly tested, and it may lead do data leak, damage, or loss.
- I wrote this by myself and i have no professional expertise in software engineering or coding. ("by myself" meaning 1% me and 99% documentation and forums.)
- This code is intented to run as a script (with the templater plugin) in the obsidian.md application.
- Check for more info, license and updates in the repository page at https://github.com/samysberg/marklisma .
- Remember to change (in the main script) the path of the variables "ixtmt" and "moctmt" to suit your use case.

# Other comments
- It requires the Templater and Dataview plugins enabled.
- This script logic could be ported to a standalone plugin, but I have no intention to do so for now.
- Pull requests and issues are welcome; there are many potential improvements to do.

# Usage
- Just put the 3 files in your Templater's template folder, make your edits, and run the main template from the "open insert template modal" command. This command has to be invoked from an opened file, and in principle any file should work for this and it shouldn't itself get modifications.
