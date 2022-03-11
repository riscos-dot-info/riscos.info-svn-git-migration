# Makefile for ccres conversion using reposurgeon
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
REMOTE_URL = svn://svn.riscos.info/ccres
READ_OPTIONS =
VERBOSITY = "verbose 1"
REPOSURGEON = reposurgeon

# Configuration ends here

.PHONY: local-clobber remote-clobber gitk gc compare clean dist stubmap
# Tell make not to auto-remove tag directories, because it only tries rm 
# and hence fails
.PRECIOUS: ccres-%-checkout ccres-%-git

default: ccres-git

# Build the converted repo from the second-stage fast-import stream
ccres-git: ccres.fi
	rm -fr ccres-git; $(REPOSURGEON) "read <ccres.fi" "prefer git" "rebuild ccres-git"

# Build the second-stage fast-import stream from the first-stage stream dump
ccres.fi: ccres.svn ccres.opts ccres.lift ccres.map $(EXTRAS)
	$(REPOSURGEON) $(VERBOSITY) "script ccres.opts" "read $(READ_OPTIONS) <ccres.svn" "authors read <ccres.map" "sourcetype svn" "prefer git" "script ccres.lift" "legacy write >ccres.fo" "write >ccres.fi"

# Build the first-stage stream dump from the local mirror
ccres.svn: ccres-mirror
	(cd ccres-mirror/ >/dev/null; repotool export) >ccres.svn

# Build a local mirror of the remote repository
ccres-mirror:
	repotool mirror $(REMOTE_URL) ccres-mirror

# Make a local checkout of the source mirror for inspection
ccres-checkout: ccres-mirror
	cd ccres-mirror >/dev/null; repotool checkout ../ccres-checkout

# Make a local checkout of the source mirror for inspection at a specific tag
ccres-%-checkout: ccres-mirror
	cd ccres-mirror >/dev/null; repotool checkout ../ccres-$*-checkout $*

# Force rebuild of first-stage stream from the local mirror on the next make
local-clobber: clean
	rm -fr ccres.fi ccres-git *~ .rs* ccres-conversion.tar.gz ccres-*-git

# Force full rebuild from the remote repo on the next make.
remote-clobber: local-clobber
	rm -fr ccres.svn ccres-mirror ccres-checkout ccres-*-checkout

# Get the (empty) state of the author mapping from the first-stage stream
stubmap: ccres.svn
	$(REPOSURGEON) "read $(READ_OPTIONS) <ccres.svn" "authors write >ccres.map"

# Compare the histories of the unconverted and converted repositories at head
# and all tags.
EXCLUDE = -x CVS -x .svn -x .git
EXCLUDE += -x .svnignore -x .gitignore
headcompare: ccres-mirror ccres-git
	repotool compare $(EXCLUDE) ccres-mirror ccres-git
tagscompare: ccres-mirror ccres-git
	repotool compare-tags $(EXCLUDE) ccres-mirror ccres-git
branchescompare: ccres-mirror ccres-git
	repotool compare-branches $(EXCLUDE) ccres-mirror ccres-git
allcompare: ccres-mirror ccres-git
	repotool compare-all $(EXCLUDE) ccres-mirror ccres-git

# General cleanup and utility
clean:
	rm -fr *~ .rs* ccres-conversion.tar.gz *.svn *.fi *.fo

# Bundle up the conversion metadata for shipping
SOURCES = Makefile ccres.lift ccres.map $(EXTRAS)
ccres-conversion.tar.gz: $(SOURCES)
	tar --dereference --transform 's:^:ccres-conversion/:' -czvf ccres-conversion.tar.gz $(SOURCES)

dist: ccres-conversion.tar.gz

#
# The following productions are git-specific
#

# Browse the generated git repository
gitk: ccres-git
	cd ccres-git; gitk --all

# Run a garbage-collect on the generated git repository.  Import doesn't.
# This repack call is the active part of gc --aggressive.  This call is
# tuned for very large repositories.
gc: ccres-git
	cd ccres-git; time git -c pack.threads=1 repack -AdF --window=1250 --depth=250
