# Makefile for DeskLib conversion using reposurgeon
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
REMOTE_URL = svn://svn.riscos.info/DeskLib
READ_OPTIONS =
VERBOSITY = "verbose 1"
REPOSURGEON = reposurgeon

# Configuration ends here

.PHONY: local-clobber remote-clobber gitk gc compare clean dist stubmap
# Tell make not to auto-remove tag directories, because it only tries rm 
# and hence fails
.PRECIOUS: DeskLib-%-checkout DeskLib-%-git

default: DeskLib-git

# Build the converted repo from the second-stage fast-import stream
DeskLib-git: DeskLib.fi
	rm -fr DeskLib-git; $(REPOSURGEON) "read <DeskLib.fi" "prefer git" "rebuild DeskLib-git"

# Build the second-stage fast-import stream from the first-stage stream dump
DeskLib.fi: DeskLib.svn DeskLib.opts DeskLib.lift DeskLib.map $(EXTRAS)
	$(REPOSURGEON) $(VERBOSITY) "script DeskLib.opts" "read $(READ_OPTIONS) <DeskLib.svn" "authors read <DeskLib.map" "sourcetype svn" "prefer git" "script DeskLib.lift" "legacy write >DeskLib.fo" "write >DeskLib.fi"

# Build the first-stage stream dump from the local mirror
DeskLib.svn: DeskLib-mirror
	(cd DeskLib-mirror/ >/dev/null; repotool export) >DeskLib.svn

# Build a local mirror of the remote repository
DeskLib-mirror:
	repotool mirror $(REMOTE_URL) DeskLib-mirror

# Make a local checkout of the source mirror for inspection
DeskLib-checkout: DeskLib-mirror
	cd DeskLib-mirror >/dev/null; repotool checkout ../DeskLib-checkout

# Make a local checkout of the source mirror for inspection at a specific tag
DeskLib-%-checkout: DeskLib-mirror
	cd DeskLib-mirror >/dev/null; repotool checkout ../DeskLib-$*-checkout $*

# Force rebuild of first-stage stream from the local mirror on the next make
local-clobber: clean
	rm -fr DeskLib.fi DeskLib-git *~ .rs* DeskLib-conversion.tar.gz DeskLib-*-git

# Force full rebuild from the remote repo on the next make.
remote-clobber: local-clobber
	rm -fr DeskLib.svn DeskLib-mirror DeskLib-checkout DeskLib-*-checkout

# Get the (empty) state of the author mapping from the first-stage stream
stubmap: DeskLib.svn
	$(REPOSURGEON) "read $(READ_OPTIONS) <DeskLib.svn" "authors write >DeskLib.map"

# Compare the histories of the unconverted and converted repositories at head
# and all tags.
EXCLUDE = -x CVS -x .svn -x .git
EXCLUDE += -x .svnignore -x .gitignore
headcompare: DeskLib-mirror DeskLib-git
	repotool compare $(EXCLUDE) DeskLib-mirror DeskLib-git
tagscompare: DeskLib-mirror DeskLib-git
	repotool compare-tags $(EXCLUDE) DeskLib-mirror DeskLib-git
branchescompare: DeskLib-mirror DeskLib-git
	repotool compare-branches $(EXCLUDE) DeskLib-mirror DeskLib-git
allcompare: DeskLib-mirror DeskLib-git
	repotool compare-all $(EXCLUDE) DeskLib-mirror DeskLib-git

# General cleanup and utility
clean:
	rm -fr *~ .rs* DeskLib-conversion.tar.gz *.svn *.fi *.fo

# Bundle up the conversion metadata for shipping
SOURCES = Makefile DeskLib.lift DeskLib.map $(EXTRAS)
DeskLib-conversion.tar.gz: $(SOURCES)
	tar --dereference --transform 's:^:DeskLib-conversion/:' -czvf DeskLib-conversion.tar.gz $(SOURCES)

dist: DeskLib-conversion.tar.gz

#
# The following productions are git-specific
#

# Browse the generated git repository
gitk: DeskLib-git
	cd DeskLib-git; gitk --all

# Run a garbage-collect on the generated git repository.  Import doesn't.
# This repack call is the active part of gc --aggressive.  This call is
# tuned for very large repositories.
gc: DeskLib-git
	cd DeskLib-git; time git -c pack.threads=1 repack -AdF --window=1250 --depth=250
