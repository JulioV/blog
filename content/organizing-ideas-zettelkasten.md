---
date: "2020-04-03"
tags: ["research", "productivity"]
title: "Organizing ideas with the Zettelkasten method"
---

I came across the Zettelkasten (ZK) method as a flexible way of organizing knowledge. I have read different descriptions, and most of them describe it as a second brain, a single text-based repository where you can dump all your ideas and link them to not only store but also generate knowledge. This is what resonated with me the most, as I know that I am the most creative once I know any topic(s) in-depth and can make connections between its different components.

According to [this website](https://zettelkasten.de/posts/overview/#principles), keeping a ZK repository has multiple benefits like improving your thinking, writing, memory, and learning. I do think that keeping one will help me resurface ideas that I have after reading papers or technical content and retain concepts for longer as you are supposed to add notes in your own words instead of copy and pasting. That said, I need to check if the recommendation of writing long pieces in a ZK works for me. The idea is that you outline your text in a note and then add subsections in other notes linked to the original. I usually follow this iterative approach to writing the difference is that I like to have a quick overview of what I have written to expand and reorganize content, so I have to see if I can get used to not having this.

For the actual implementation of my ZK repo, I chose to type all my notes in markdown files stored in a single directory backed up in Github and [Sublime](https://www.sublimetext.com) with the [ZK plugin]((https://github.com/renerocksai/sublime_zk#zettelkasten-mode)) in Mac OS (it should be cross-platform).

## Sublime as editor

I set up my ZK repository in Mac OS using Sublime and a few extra plugins as suggested [here](https://github.com/renerocksai/sublime_zk#zettelkasten-mode). Most of the steps below are taken from that project's README, but I replicate some of them here for future reference (I find the official docs a bit overwhelming).
1. Install the [ZK plugin]((https://github.com/renerocksai/sublime_zk#zettelkasten-mode)):
 - In Sublime's command palette (`cmd + P`) run *Install Package Control*
 - In Sublime's command palette run *Package Control: Add Repository* and paste this URL when prompted `https://github.com/renerocksai/sublime_zk`
 - In Sublime's command palette run *Package Control: Install Package* and search for `sublime_zk` when prompted
2. Install the Silver Searcher plugin using: `brew install the_silver_searcher`
3. Install Pandoc using: ` brew install pandoc`
4. Re-start Sublime

### Initializing my ZK folder and creating my first note

1. In Sublime's command palette run *ZK: New Zettel Note* and type a name for your new note
1. When prompted choose or create a new folder as your ZK repository (I added mine to git source control)
1. In Sublime's command palette run *ZK: Enter in Zettelkasten Mode*
1. Now you can create a new note by typing `Shift + Enter`

### Configuring my ZK folder
I am using the default file extension (`.md`), link notation (`[[link]]`) and ID precision in minutes (`YYYYMMDDHHMM`)

I configured ZK to insert the title of a note next to its ID when they are linked from other notes. In addition, I changed the color scheme, set the bib citation format to pandoc's, and modified the template for new notes to insert the note ID, title, date, and tags at the beginning of the file. To do all this, go to Sublime's *Preferences* > *Package Settings* > *Sublime ZK* > *Settings User* and add the following code:

```json
{
 // Insert note titles next to links when linking a note frome another
 "insert_links_with_titles": true,
 // Template for new notes
 "new_note_template":
 "---\nnote-id: {id}\ntitle: {title}\ndate: {timestamp: %Y-%m-%d}\ntags: \n---\n",
 "color_scheme": "Packages/sublime_zk/Monokai Extended-zk.tmTheme",
 "citations-mmd-style": false,

}
```

When you create a new note with the config above, its header will look like this: 

```yaml
---
note-id: 202003281428
title: My second note
date: 2020-03-28
tags: 
---
```

### Taking notes in my ZK

I create notes with the following [principles](https://zettelkasten.de/posts/overview/#principles) in mind:

- Each note is atomic and self-contained. This means that a note is related to a single idea, and I don't need anything else to remember what a note means.
- A link is a stronger connection than a tag. In other words, a single idea is developed throughout different notes connected by links, and multiple ideas related to the same broad topic are grouped with tags.
- If a new note is related to an existing note, I link the parent note in the child note (`[[parent ID]]`).
- I try to use specific tags, and before adding a new one, I list all existent tags to make sure I am not duplicating any. Using the `# + ?` shortcut helps me avoid typos.

Finally, I found the following shortcuts the most useful:

- Create a new note `shift + enter`
- Open the note pointed by a link `opt + double-click on link` or `cursor on link + ctrl + enter`
- Insert a link to a note `[ + [`
- Find all notes referencing another note `cursor on link + opt + enter`
- View all tags `# + !`
- View all notes `[ + !`
- Autocomplete tag `# + ?`
- Find all notes tagged by a tag `cursor on tag + ctrl + enter`
- Expand note link inline `ctrl + .`
- Expand tag inline (all referencing notes) `ctrl + .`
- Expand citekey inline (all referencing notes) `ctrl + .`
- Insert pandoc citation `[ + @`

I will update my experience using ZK in a few months.