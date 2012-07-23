---
layout: post
title: Rolling back to an old revision in Mercurial (like git reset)
---

Recently I've wanted the functionality of `git reset --hard` several times in Mercurial. Though Mercurial doesn't provide an easy alternative and shies away from the idea of mutable history, there are a few ways to achieve something close. In this post, I'll outline a few of them.

## hg rollback

If you specifically want to undo your last commit, you can use

    hg rollback

which will undo the commit (and doesn't touch the working tree -- you should be able to follow it with a `hg revert --all` or `hg update -C` if you want to throw away all your changes as well).

## hg clone -r

Suppose you want to reset your repository called `giraffe` to revision 77182fb7451f. If you `cd` into the parent directory and run

    hg clone -r 77182fb7451f giraffe new-giraffe
    cp giraffe/.hg/hgrc new-giraffe/.hg/

then you'll end up with new-giraffe, a repository at revision 77182fb7451f. (The `cp` is necessary to ensure that new-giraffe `paths` (like git's "remotes") correctly point to the origin repo instead of to the `giraffe` folder as it would by default.)

This is faster than recloning from the internet because it only copies files locally on your disk (and on many systems, hard-links them to save even more time and space), but can still be really time-consuming if your repository is large.

## hg strip
                                                                                
If you want to do more complicated things with modifying the commit history (which is somewhat frowned on in the hg world, by the way), first enable the [Mercurial Queues extension](http://mercurial.selenic.com/wiki/MqExtension/) by adding the following lines to your `.hgrc` (there's no need to specify a path after the equals sign):

    [extensions]                                                                    
    mq =

This gives you (among other useful things) the `hg strip` command to completely remove a changeset and all of its descendants from a repository. Thus you can use a command like

    hg strip 1cc72d33ea76

if 1cc72d33ea76 is the first "bad" changeset in your repository that you want to remove.

Unfortunately, it's often hard to remove exactly the right changesets using this method and it's all too easy to end up with multiple heads (which you can view using `hg heads`), requiring tedious repeated application of `hg strip` before you can get to where you want to be with no extra heads.

## hg strip using revsets

Using [hg revsets](http://selenic.com/hg/help/revsets), a Mercurial feature that I just learned about, you can delete all the ancestors of `tip` which are _not_ ancestors of the revision you want to revert to, using a command like:

    hg strip "ancestors(tip) and not ancestors(77182fb7451f)"

Make sure to first run

    hg log -r "ancestors(tip) and not ancestors(77182fb7451f)"

which will show you all the changesets that `hg strip` would remove, so you can make sure you don't irreparably harm your commit history (because you have backups… right?).

Hopefully this was helpful -- let me know if there's any way to make this easier that I'm missing.
