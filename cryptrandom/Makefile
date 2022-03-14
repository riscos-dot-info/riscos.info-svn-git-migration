# Makefile for cryptrandom conversion using reposurgeon
#
# Steps to using this:
# 1. Make sure reposurgeon and repotool are on your $PATH.
# 2. For svn, set REMOTE_URL to point at the remote repository
#    you want to convert.
# 3. For cvs, set CVS_HOST to the repo hostname and CVS_MODULE to the module,
#    then uncomment the line that builds REMOTE_URL 
#    Note: for CVS hosts other than Sourceforge or Savannah you will need to 
#    include the path to the CVS modules directory after the hostname.
# 4. Set any required read options, such as --user-ignores or --nobranch,
#    by setting READ_OPTIONS.
# 5. Run 'make stubmap' to create a stub author map.
# 6. (Optional) set REPOSURGEON to point at a faster cython build of the tool.
# 7. Run 'make' to build a converted repository.
#
# The reason both first- and second-stage stream files are generated is that,
# especially with Subversion, making the first-stage stream file is often
# painfully slow. By splitting the process, we lower the overhead of
# experiments with the lift script.
#
# For a production-quality conversion you will need to edit the map
# file and the lift script.  During the process you can set EXTRAS to
# name extra metadata such as a comments mailbox.
#
# Afterwards, you can use the headcompare and tagscompare productions
# to check your work.
#

EXTRAS = 
REMOTE_URL = svn://svn.riscos.info/cryptrandom
READ_OPTIONS =
VERBOSITY = "verbose 1"
REPOSURGEON = reposurgeon

# Configuration ends here

.PHONY: local-clobber remote-clobber gitk gc compare clean dist stubmap
# Tell make not to auto-remove tag directories, because it only tries rm 
# and hence fails
.PRECIOUS: cryptrandom-%-checkout cryptrandom-%-git

default: cryptrandom-git

# Build the converted repo from the second-stage fast-import stream
cryptrandom-git: cryptrandom.fi
	rm -fr cryptrandom-git; $(REPOSURGEON) "read <cryptrandom.fi" "prefer git" "rebuild cryptrandom-git"

# Build the second-stage fast-import stream from the first-stage stream dump
cryptrandom.fi: cryptrandom.svn cryptrandom.opts cryptrandom.lift cryptrandom.map $(EXTRAS)
	$(REPOSURGEON) $(VERBOSITY) "script cryptrandom.opts" "read $(READ_OPTIONS) <cryptrandom.svn" "authors read <cryptrandom.map" "sourcetype svn" "prefer git" "script cryptrandom.lift" "legacy write >cryptrandom.fo" "write >cryptrandom.fi"

# Build the first-stage stream dump from the local mirror
cryptrandom.svn: cryptrandom-mirror
	(cd cryptrandom-mirror/ >/dev/null; repotool export) >cryptrandom.svn

# Build a local mirror of the remote repository
cryptrandom-mirror:
	repotool mirror $(REMOTE_URL) cryptrandom-mirror

# Make a local checkout of the source mirror for inspection
cryptrandom-checkout: cryptrandom-mirror
	cd cryptrandom-mirror >/dev/null; repotool checkout ../cryptrandom-checkout

# Make a local checkout of the source mirror for inspection at a specific tag
cryptrandom-%-checkout: cryptrandom-mirror
	cd cryptrandom-mirror >/dev/null; repotool checkout ../cryptrandom-$*-checkout $*

# Force rebuild of first-stage stream from the local mirror on the next make
local-clobber: clean
	rm -fr cryptrandom.fi cryptrandom-git *~ .rs* cryptrandom-conversion.tar.gz cryptrandom-*-git

# Force full rebuild from the remote repo on the next make.
remote-clobber: local-clobber
	rm -fr cryptrandom.svn cryptrandom-mirror cryptrandom-checkout cryptrandom-*-checkout

# Get the (empty) state of the author mapping from the first-stage stream
stubmap: cryptrandom.svn
	$(REPOSURGEON) "read $(READ_OPTIONS) <cryptrandom.svn" "authors write >cryptrandom.map"

# Compare the histories of the unconverted and converted repositories at head
# and all tags.
EXCLUDE = -x CVS -x .svn -x .git
EXCLUDE += -x .svnignore -x .gitignore
headcompare: cryptrandom-mirror cryptrandom-git
	repotool compare $(EXCLUDE) cryptrandom-mirror cryptrandom-git
tagscompare: cryptrandom-mirror cryptrandom-git
	repotool compare-tags $(EXCLUDE) cryptrandom-mirror cryptrandom-git
branchescompare: cryptrandom-mirror cryptrandom-git
	repotool compare-branches $(EXCLUDE) cryptrandom-mirror cryptrandom-git
allcompare: cryptrandom-mirror cryptrandom-git
	repotool compare-all $(EXCLUDE) cryptrandom-mirror cryptrandom-git

# General cleanup and utility
clean:
	rm -fr *~ .rs* cryptrandom-conversion.tar.gz *.svn *.fi *.fo

# Bundle up the conversion metadata for shipping
SOURCES = Makefile cryptrandom.lift cryptrandom.map $(EXTRAS)
cryptrandom-conversion.tar.gz: $(SOURCES)
	tar --dereference --transform 's:^:cryptrandom-conversion/:' -czvf cryptrandom-conversion.tar.gz $(SOURCES)

dist: cryptrandom-conversion.tar.gz

#
# The following productions are git-specific
#

# Browse the generated git repository
gitk: cryptrandom-git
	cd cryptrandom-git; gitk --all

# Run a garbage-collect on the generated git repository.  Import doesn't.
# This repack call is the active part of gc --aggressive.  This call is
# tuned for very large repositories.
gc: cryptrandom-git
	cd cryptrandom-git; time git -c pack.threads=1 repack -AdF --window=1250 --depth=250
