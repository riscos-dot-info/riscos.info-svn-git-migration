# Makefile for gccsdk conversion using reposurgeon
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
REMOTE_URL = svn://svn.riscos.info/gccsdk
READ_OPTIONS =
VERBOSITY = "verbose 1"
REPOSURGEON = reposurgeon

# Configuration ends here

.PHONY: local-clobber remote-clobber gitk gc compare clean dist stubmap
# Tell make not to auto-remove tag directories, because it only tries rm 
# and hence fails
.PRECIOUS: gccsdk-%-checkout gccsdk-%-git

default: gccsdk-git

# Build the converted repo from the second-stage fast-import stream
gccsdk-git: gccsdk.fi
	rm -fr gccsdk-git; $(REPOSURGEON) "read <gccsdk.fi" "prefer git" "rebuild gccsdk-git"

# Build the second-stage fast-import stream from the first-stage stream dump
gccsdk.fi: gccsdk.svn gccsdk.opts gccsdk.lift gccsdk.map $(EXTRAS)
	$(REPOSURGEON) $(VERBOSITY) "script gccsdk.opts" "read $(READ_OPTIONS) <gccsdk.svn" "authors read <gccsdk.map" "sourcetype svn" "prefer git" "script gccsdk.lift" "legacy write >gccsdk.fo" "write >gccsdk.fi"

# Build the first-stage stream dump from the local mirror
gccsdk.svn: gccsdk-mirror
	(cd gccsdk-mirror/ >/dev/null; repotool export) >gccsdk.svn

# Build a local mirror of the remote repository
gccsdk-mirror:
	repotool mirror $(REMOTE_URL) gccsdk-mirror

# Make a local checkout of the source mirror for inspection
gccsdk-checkout: gccsdk-mirror
	cd gccsdk-mirror >/dev/null; repotool checkout ../gccsdk-checkout

# Make a local checkout of the source mirror for inspection at a specific tag
gccsdk-%-checkout: gccsdk-mirror
	cd gccsdk-mirror >/dev/null; repotool checkout ../gccsdk-$*-checkout $*

# Force rebuild of first-stage stream from the local mirror on the next make
local-clobber: clean
	rm -fr gccsdk.fi gccsdk-git *~ .rs* gccsdk-conversion.tar.gz gccsdk-*-git

# Force full rebuild from the remote repo on the next make.
remote-clobber: local-clobber
	rm -fr gccsdk.svn gccsdk-mirror gccsdk-checkout gccsdk-*-checkout

# Get the (empty) state of the author mapping from the first-stage stream
stubmap: gccsdk.svn
	$(REPOSURGEON) "read $(READ_OPTIONS) <gccsdk.svn" "authors write >gccsdk.map"

# Compare the histories of the unconverted and converted repositories at head
# and all tags.
EXCLUDE = -x CVS -x .svn -x .git
EXCLUDE += -x .svnignore -x .gitignore
headcompare: gccsdk-mirror gccsdk-git
	repotool compare $(EXCLUDE) gccsdk-mirror gccsdk-git
tagscompare: gccsdk-mirror gccsdk-git
	repotool compare-tags $(EXCLUDE) gccsdk-mirror gccsdk-git
branchescompare: gccsdk-mirror gccsdk-git
	repotool compare-branches $(EXCLUDE) gccsdk-mirror gccsdk-git
allcompare: gccsdk-mirror gccsdk-git
	repotool compare-all $(EXCLUDE) gccsdk-mirror gccsdk-git

# General cleanup and utility
clean:
	rm -fr *~ .rs* gccsdk-conversion.tar.gz *.svn *.fi *.fo

# Bundle up the conversion metadata for shipping
SOURCES = Makefile gccsdk.lift gccsdk.map $(EXTRAS)
gccsdk-conversion.tar.gz: $(SOURCES)
	tar --dereference --transform 's:^:gccsdk-conversion/:' -czvf gccsdk-conversion.tar.gz $(SOURCES)

dist: gccsdk-conversion.tar.gz

#
# The following productions are git-specific
#

# Browse the generated git repository
gitk: gccsdk-git
	cd gccsdk-git; gitk --all

# Run a garbage-collect on the generated git repository.  Import doesn't.
# This repack call is the active part of gc --aggressive.  This call is
# tuned for very large repositories.
gc: gccsdk-git
	cd gccsdk-git; time git -c pack.threads=1 repack -AdF --window=1250 --depth=250
