# Git Tutorial

- [Git Tutorial](#git-tutorial)
  - [How to write a good commit message](#how-to-write-a-good-commit-message)
  - [Merge vs Rebase vs Cherry Picking](#merge-vs-rebase-vs-cherry-picking)
    - [Merge](#merge)
    - [Rebase](#rebase)
      - [Editing](#editing)
      - [Squashing](#squashing)
    - [Cherry picking](#cherry-picking)
  - [Interactive Staging](#interactive-staging)
  - [Recover deleted commits](#recover-deleted-commits)
  - [Hooks](#hooks)
  - [References](#references)

## How to write a good commit message

I'm not going to say too much on this, there are 101 blogs out there on this topic. I would just ask you this one question, `Look through your old commits, can you tell, just from the message alone what you did and why?`. The answer is likely no, not to worry, even though I think I write pretty good commit messages, I still don't always understand what I did, but usually I can tell why. Read the referenced links on this topic. Suffice to say, most people write the bare minimum in a commit, because they want to move on and do something else, that is the biggest mistake everyone makes. Commit messages are the first place to look for when a bug has been introduced, so you grep the history for some keywords, so treat commit messages with respect, it doesn't hurt BAs/Managers, but they do hurt you and other devs. The next mistake most of us make is in detailing what you did, but I can usually tell that by reading the code, so what is your message actually adding? The message should state why you did something, the code tells us how, the message tells us what and why. So, you add a **title**, written in the present tense e.g. `Fix bug with downloading date ranges`, the body of the commit should be something like 
```
I found the `get_query_params` function, to be ignoring the date ranges passed in. I have since updated that function to read the params being passed in and use them in the query.
```

## Merge vs Rebase vs Cherry Picking

First off, there is no right answer, these are 3 tools we can use to sync 2 branches, each has pros/cons and it is understanding these, that should help decide, which to use and when.


### Merge

Most people are familiar with and use `merge`, mostly cause it is easy, you go to the branch you want to update and run `git merge <branch to mege into this one>`. There can be **Merge Conflicts**, so you fix those, run `git merge --continue` or if your lucky and there are no conflicts, you write a new commit, which is pre-populated to say what your doing e.g.

```text
commit c106ba59995893d0d5ba9a8ea63e4688c88482dd
Merge: fe7c91bd7 3bfbac17f
Author: Mark Small <mark.small@astrosat.net>
Date:   Thu Nov 24 11:24:15 2022 +0000

    Merge branch 'master' into 1415-crossfiltering
```

Okay that's fine, so why do we need anything else? Well, honestly, you don't, but what are the implications of `merging`. The primary one, is first you create an **Empty Commit**, so all the commits you created get added at the top of the branch you are in and then an empty one is added on top. Why? I honestly do not know the answer to that, it just does. Another possible con is that the commits you merge in are placed at the top of the list, so even if they were committed before your latest ones, they sit on top, so you loose temporal linearness (is that a real word or did I just make that up??). That might not matter, but if you have to go back and look at the commit history and you know something was done in the last month, if they are in temporal order, that is easy, if not, it makes it harder, maybe not much, but it does make it harder.

### Rebase

Many people are **scared** of `rebase`, they hear the term `it re-writes the history` and that freaks them out, but what does that really mean? All it means is that it will change the unique IDs|Commit Hashes|SHA sums of the commits you've rebased. Now, if your developing a new commit on your machine that no-one has access to e.g. you haven't pushed it, then that doesn't matter, so think of rebasing as valid until you push code. I would take it further, often I push code, then realize I've broken tests or something. Now, devs don't drop everything to review my code, so I have a window, or I can say, ignore PR XXX until I say so, or I can put the PR into `draft` mode. Then I am still free to amend my previous commits, to add fixes to the tests, in the commit I should have done it in. There is also nothing to stop me adding another commit, but really, should tests be committed separately from the code they are testing? there is no right answer to that, personally I think not, mostly for the reason that if I do have to go back in history, it's generally easier to find the commit message to add the thing, that the commit that might come a few commits later, that test the thing, so I prefer to commit the code and it's associated tests, as one thing.

Anyway, rebasing gives you the ability to edit previous commits, that is all, yes you can get it wrong, but so too can you get it wrong fixing merge conflicts with `merge`. The plus side is you have a nice neat termporally linear commit history, no extraneous, I forgot to do this, so have not done so commits, you just updated the commit you should have done what your forgot to do in the first place as if you hadn't forgot. The thing to remember is, if anyone has pulled your code, before you pushed cahnges using rebase, you have to tell them to remove their copy of your old branch and re-check it out, it's very unlikely they are also making changes to that same branch. So the main trouble mostly comes around having got the commits into the `master` branch and then looking to rebase, that is a big no-no, in this case, just add a new commit.

This is kinda idealised as there is no code-base that is perfect, things will get forgotten and it won't be for another week they are realized, therefore extra commits are needed. Rebase is mostly good for you developing your feature branch, prior to it being merged into master.

The 2 primary rebasing options I tend to use are `editing` and `squashing`, I'll talk about them separately.

#### Editing

This is my primary reason for using **rebase**, it allows me to edit an existing commit, to add/remove/update the contents, or the message itself.

#### Squashing

I use this less, but it can be useful, if I have 2 or more smaller commits that really are one thing, then I can squash them down into however many I deem to be the right number.

### Cherry picking

Cherry-picking as the name suggests is saying take commit x and z, but not y and add them to the top of my current branch. I do this less, mostly if a dev is having some trouble with some code, they push a temp branch for me to look at, I may make a couple of commits to that and push it. They can then choose to merge|rebase all of what I did, or only cherry-pick some commits. I also do this when my feature branch is old, in comparison to master. In safers, I added a branch to use tiled raster images, but issues were found around how long the tiles took and it was decided to not merge it. 2 months passed and the conversation arose again. Now, master had moved on a lot, I could have merged|rebased master into my feature branch, but that could mean lots of merge conflict resolution, or an empty commit, so I chose to create a new branch from master, and cherry-pick the 2-3 commits I made in my old branch, fixing the errors as I went, so it looked as if I had only just worked on it all.

## Interactive Staging

What does **Interactive Staging** mean? well I hope you understand that when you add files e.g. `git add .` or `git add textfile.txt`, you are **staging them to be committed**. GIT, unlike some of it's predecessors, works on what it calls the object level. What that means is you can commit a single `character`, `word`, `paragraph`, `file`, `directory of files`. While others like **Subversion** you could only commit `file` and `directory of files`. So what does this mean in practice? Often I am working on some code to provide some functionality, to provide that functionality though, I might code some different logical areas e.g. data fetching, data transformation, or data displaying. It might be that I don't want to commit that all as one big commit, maybe I want to break that up. However, I kinda need to have developed a lot of the code first, to ensure it works together, before I commit any. I don't want lots of small commits after saying, I fixed the data fetching after I did the data transformation, cause I noticed I could make a small adjustment that makes the transformation easier. So, I have a bunch of code, possibly split across several files and I now want to commit it. I could commit as one big commit and that would be valid, but what if I had to edit some shared utility function, maybe it doesn't make sense to commit that with the rest, maybe it should be committed separately, maybe I've edited some existing utility function but added 2 new ones, well again, these should be commited separately and the commit messages focus on those specific changes. This is where **Interactive Staging** comes in, the command is `git add -p <file>`, you could interactively add many files, but I tend to find it easier to focus on one file at a time. So maybe I want to add only the code to fix a big I found in an existing function. I can add that on it's own and leave the other changed code in the file, uncommitted (at this time). Maybe I notice I've left a print statement, well I don't want to commit that so I can just not commit that line etc. `This is one of those, it's easier to seen than explain ones`.

## Recover deleted commits

I'm sure at some point, we have all deleted a committed branch/file that we shouldn't have, if we are lucky, we have yet to commit this deletion, but imagine you've deleted a branch where you added a feature accidentally. No you've lost all that work, haven't you? Well no, nothing committed in GIT is every deleted, just the reference to it is. Think of it this way, GIT is just a bucket of individual commits, just thrown into the bargain basement bucket, for us to sift through. Branches make it easier as each commit has a reference to their parent (not really how it works, but conceptually I think this explains how it seems). So, branches are just lists of commits, that all reference each other, the branch references the top|HEAD commit, it references it's parent, the parent references it's parent etc etc.

So, if we have a big bucket and nothing committed is never actually deleted, how can I recover a branch of commits, I just accidentally deleted? This is where `git reflog` comes into effect. It lists all commits in the bucket, now, if you've written **good commit messages**, it should be fairly easy to find the top commit of the branch you just deleted. Now that pre-supposes you know kinda what the top level commit was, often, you might not. There are too many possible situations for me to state here, so all I can say is `reflog` is the tool to use, it is an ordered list of commits most recent first. Look at some blogs on recovering branches/commits, they may be able to provide exact steps, for your situation. I've added at least one URL to the **References** section of this document to help get you started.

## Hooks

**Hooks** are basically scripts that run on specific GIT commands e.g. `add`, `pre-push`, `push`, `post-push` etc. They are mostly client-side, which means they are specific to each individual, you can't share hooks, other than saying, here is a hook, add it to this file in your git repository. There are some tools that provide server-side hooks, but I don't know much about them, I've never used them. What use are hooks, well, they can protect devs from themselves, thing of a hook that runs tests on the `pre-push`, so you run `git push`, the tests are run, you notice you've just broken a test, so you can fix it without looking silly in front of everyone else. There are many possibilities, if you can script it, it can be done, we had one that validated metadata files against the JSON schema in our dataset registry, that was run before committing, for instance. Hooks can be written in any scripting language and be reference from the hook file, under `$PROJ_ROOT/.git/hooks/` directory, or you can inline the script code, if it is a bash script.

## References

- [How to write good commit messages](https://www.freecodecamp.org/news/how-to-write-better-git-commit-messages/)
- [GIT commit format](https://github.com/angular/angular/blob/main/CONTRIBUTING.md#-commit-message-format)
- [Advanced Tips](https://www.atlassian.com/git/tutorials)
- [Recover deleted branch](https://rewind.com/blog/how-to-restore-deleted-branch-commit-git-reflog/)
- [Hooks](https://www.atlassian.com/git/tutorials/git-hooks)