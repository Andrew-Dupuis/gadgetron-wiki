## Git Repositories
In migrating from SourceForge to GitHub, the git repository commit history was modified.  To use the new repository, please do NOT pull and merge.  Instead, you should remove your local source code tree and clone a fresh copy from the current [repository](http://github.com/gadgetron/gadgetron).

## SourceForge archive
If you need to access a specific commit in the old SF repository you have two options.

### Option 1: search the current repo for the old hash
If you are looking for the old hash fc9d22984d3bf3460ce0e7c8bdfc4d5fc04dd2c4:
    git log --all -g -grep="fc9d22984d3bf3460ce0e7c8bdfc4d5fc04dd2c4"
will return the commit with its new hash
    commit 6c14b8c20ead81e1b0cc673a197a995395aaa627
    Author: David C Hansen <dch@cs.au.dk>
    Date:   Thu Nov 6 16:28:52 2014 +0100
    
        Change DicomFinishGadget to write multi frame images
        
        Former-commit-id: fc9d22984d3bf3460ce0e7c8bdfc4d5fc04dd2c4

### Option 2: search through the old repo
An archive of the decommissioned source forge repository can be found here:
[](https://s3.amazonaws.com/gadgetron.github.io/gadgetron-repo-20141114.tar.gz)