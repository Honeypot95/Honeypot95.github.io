---
layout: post
title:  "Creating phantom commits with git rebase"
date:   2022-01-04 17:44:02 +0200
categories: jekyll update
---
<div class="ABSTRACT" id="org31fce8f">
<p>
Git rebase is probably the first git feature you should learn after mastering the basics. There are good
articles on the web about why, how, and when you should use it. If you are short on time, you should stop
reading this and instead learn about <a href="https://hackernoon.com/git-merge-vs-rebase-whats-the-diff-76413c117333">git rebase</a>.
</p>

<p>
If you are still here, read on.
</p>
<!--more-->
</div>


# Problem

While reading the above article, I noticed something in the diagrams. With `git merge`, things are ugly and
straightforward. You make divergent changes, you bring them together in an opaque merge commit, and solve
whatever conflicts arise.

![img](./images/git-merge.png "`git merge`")

With `git rebase` however, new commits are created in order to have a linear history, just as if you wrote the
commits yourself.

![img](./images/git-rebase.png "`git rebase`")

The thing is, `git rebase` seems to create commits that have never been on anyone&rsquo;s computer before. In the
above diagram, the state ***a*** *b* never existed as such when writing the code, so it&rsquo;s not tested. Can this
be used to make a malicious commit? Let&rsquo;s find out!


# Example

Note: This is a toy example that would probably never happen in real life. I still think it is a valuable
exercise, though.

Let&rsquo;s create a script that cleans up some of our scripts in either `/bin/` or `~/bin/`. Of course, we want
this to be version controlled.

We want either to:

-   delete all scripts in `~/bin/` (safe)
-   delete a specific script in `~/bin/` (safe)
-   delete a specific script in `/bin/` (safe)

We certainly do not want to:

-   delete all scripts in `/bin/` (unsafe)

Create the script and initialize the git repo as usual:
    {% highlight bash %}
    mkdir -p ~/test/test-git-rebase/ && cd ~/test/test-git-rebase/
    touch cleanup.sh {% endhighlight %}
We have the following contents in the `cleanup.sh` file:

    ROOTDIR="~/bin/"
    FILES="*"

This is basically just configuration, no commands yet. We now commit this as our initial commit.

    git init
    git add cleanup.sh
    git commit -m "Initial commit"

Observing git best practices, we create a branch for our feature, called `feature`.

    git checkout -b feature

This is our basic setup. We now make the following changes, trying to mimic a normal workflow.

-   initial commit: `500d3a1`
-   on master, we change the `ROOTDIR` to point to the system `/bin/` (commit `8178a85`)
-   on feature, add the `rm` command, with the `-rf` flags (commit `661e350`)
-   on feature, we change the the `FILES` to point to a specific file (commit `d1c8943`)
-   on feature, we remove the `rm` flags. (commit `8b610bc`)

As the `ROOTDIR` points to `/bin/` only on master, and since master has no `rm` commands, there is no state in
which we could accidentally delete all our `/bin/` scripts.

For now, our `git log` s look like thse:

feature:

    Recent commits
    8b610bc feature Remove the rm flags.
    d1c8943 FILES now point to a specific script.
    661e350 Added rm -rf command
    500d3a1 Initial commit

master:

    Recent commits
    8178a85 master ROOTDIR now points to /bin/
    500d3a1 Initial commit

We continue to observe git best practices, and we rebase our `feature` branch onto `master`.

    git rebase master

This prompts us to resolve the following conflict:

    <<<<<<< HEAD
    ROOTDIR="/bin/"
    FILES="*"
    =======
    ROOTDIR="~/bin/"
    FILES="that-problematic-script.sh"
    >>>>>>> d1c8943... FILES now point to a specific script.
    rm -rf "$ROOTDIR""$FILES"

We resolve it by keeping the `ROOTDIR` from master, and the rest from `feature`. The file now looks like this:

    ROOTDIR="/bin/"
    FILES="that-problematic-script.sh"
    rm -rf "$ROOTDIR""$FILES"

Our `git log` for feature now looks like this:

feature:

    Recent commits
    c6725e6 feature Remove the rm flags.
    40c8fce FILES now point to a specific script.
    37e5678 Added rm -rf command
    8178a85 master ROOTDIR now points to /bin/
    500d3a1 Initial commit

We see that the hash for the commit that adds `rm` has changed (because rebase changed our commit). So let&rsquo;s take a look at what exactly is there:

    ROOTDIR="/bin/"
    FILES="*"
    rm -rf "$ROOTDIR""$FILES"

Aha! This is the commit we were trying to create. This finally violates our only rule, that we do not want
to delete all the scripts from `/bin/`. This commit has never been on our computer, in this state. We also
did not see this during our conflict, because after we resolved the conflict, the state was deleting the
specified script from the `/bin/` directory, not all scripts.

We still need a way to naively execute this script. Luckily, our malicious phantom commit is now right in
the middle of our `git log`. Suppose we continue to observe git best practices, and we have a bug on our
last commit. We know the first commit is good. So we do `git bisect` to find out exactly what went wrong.

    git bisect start
    git bisect good # our last commit is good
    git bisect bad 500d3a1 # Initial commit

`git bisect` leaves us right in the middle, on our malicious phantom commit. Executing the script now to see
if it bugs out or not, we delete all the scripts from our `/bin/` directory.


# Conclusion

This is clearly a ridiculous example, used to point out that intermediary commits can be very, very
dangerous. The commit is never on anyone&rsquo;s computer until the rebase. Using `git bisect` to trigger the
commit may seem ludicrous, but it it the only example I could think of. I also think this does not happen
with interactive rebase.

Can this be used in practice to introduce malicious commits? I don&rsquo;t think so, with good reviews. Even if
you manage to generate the commit, it still needs to run, and I don&rsquo;t see how our small `git bisect`
exercise could be used in practice.

Regardless, this was a fun experiment!
