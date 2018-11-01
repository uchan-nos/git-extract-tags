# git-extract-tags

A tool to support git-based textbook writing.
git-extract-tags extracts files from specified tags into directories.

This tool will work well two-repository model, such as:

- textbook repository which contains source codes for the textbook.
- program repository which contains source codes for the target program.

(Sorry! Japanese version only)
詳しい使い方をブログで紹介しています  
[「30 日でできる！OS 自作入門」のような段階的に進展するプログラムについての解説書の執筆を Git で支援する](http://uchan.hateblo.jp/entry/2018/08/11/191600)

## How to setup

Clone this repository and create symlink in a directory where is included in $PATH variable.
git-extract-tags requires GitPython library.

Example:

    $ cd /path/to/somewhere
    $ git clone https://github.com/uchan-nos/git-extract-tags.git
    $ cd /usr/local/bin
    $ sudo ln -s /path/to/somewhere/git-* ./
    $ sudo pip3 install gitpython

## Typical situation this tool addresses

Typical usage of this tool is for a textbook which explains a large program,
whose codes are evolved step-by-step manner.

Imagine you are going to write an tutorial about implementing an RPG.
This textbook may have 4 sections:
1. A game loop with one player character
2. Implementing events
3. Battle with enemies
4. Making scenarios

If you want to explain evolvements between steps, the repository for the
textbook may look like this:

    /               root directory of this textbook
        Makefile    build script of the textbook
        text/       source codes for this textbook (Markdown, Sphinx, etc.)
        src/        source codes of each snapshot of the target program
            sec1/   snapshot of section 1
            secN/

`src/sec1` is a snapshot of the game program which contains only a game loop
with one player character. The directory contains neither events nor enemies.

`src/sec2` is a snapshot of section 2 which contains a game loop and events.

and so on.

Assume you've written 2 sections: section 1 and section 2.
The status of the game program is between section 2 and 3.

Now you noticed a typo in a player character name, which has been
implemented at very beginning of the game program history.
To fix it, you have to apply a patch to both `src/sec1` and `src/sec2`.
The more evolvements between a bug and the latest version of the game program,
the more files to apply patch.


## Solution this tool provides

You should be released from suffering by using this tool :)

git-extract-tags assumes the following directory structure:

    rpgbook/        root directory of this textbook
        Makefile
        text/
        src/
    myrpg/          root directory of the game program
        source codes ...

And git-extract-tags assumes the `myrpg` repository contains tags like
`rpgbook-sec1` for each snapshot.
A tag must consist of a prefix ( `rpgbook-` ) and a snapshot name ( `sec1` ).
Then you can extract files from tags:

    $ cd rpgbook
    $ git extract-tags rpgbook- ../myrpg

Above commands extract files of `rgpbook-XX` tag to `src/rpgbook-XX` directory.
If there are 2 tags, rpgbook-sec1 and rpgbook-sec2, then 2 directoeis,
src/rpgbook-sec1 and src/rpgbook-sec2, are re-created (removed then made)
with files from corresponding tags.

# git-retag

This command moves tags to each corresponding commit. It will help you to retag
when you have made some fixes to earlier history of the target software.

Imagine following git history of two branches (master and fix-some-bug).

master:
- D "Battle with enemies"                rag: rpgbook-sec3
- C "Add events"                         rag: rpgbook-sec2
- B "Add a game loop with one player"    tag: rpgbook-sec1
- A "Initial commit"

fix-some-bug:
- D' "Battle with enemies"
- C' "Add events"
- E "Fix player name"
- B "Add a game loop with one player"    tag: rpgbook-sec1
- A "Initial commit"

You may want to set tag "rpgbook-sec1" to E, "rpgbook-sec2" to C',
and "rpgbook-sec3" to D'. This can be done by deleting three tags and adding
these to each commit, but this is a bother, isn't it?

git-retag command helps you! The following command will re-tag all three tags
to corresponding commits.

    $ git retag rpgbook- master fix-some-bug
