SVN_SERVER := svn.riscos.info
# Path to local mirror of upstream repository
GCCSDK_REPO_PATH := $(CURDIR)/gccsdk-mirror
# Path to user identity mapping
USERS_MAP := $(realpath $(CURDIR)/gccsdk.map)

.PHONY: all clean

all: gccsdk.git autobuilder.git

clean:
	rm -rf $(GCCSDK_REPO_PATH)
	rm -rf gccsdk.git
	rm -rf autobuilder.git

# reposurgeon cannot cope with the hierarchical developers
# branches within the SVN repository.  Deal with this by
# rewriting the branch names before processing (the lift
# script will map them back to reality once everything
# has been ingested).
gccsdk.git: gccsdk.lift shared.lift $(GCCSDK_REPO_PATH) $(USERS_MAP) Makefile
	rm -rf $@
	svnadmin dump $(GCCSDK_REPO_PATH) | \
	repocutter pathrename \
	    '^branches/developers/([^/]+)/([^/]+)' \
	    'branches/developers+$${1}+$${2}' | \
	reposurgeon "script gccsdk.lift"
	(cd $@ && git checkout -f main)

autobuilder.git: autobuilder.lift shared.lift $(GCCSDK_REPO_PATH) $(USERS_MAP) Makefile
	rm -rf $@
	svnadmin dump $(GCCSDK_REPO_PATH) | \
	repocutter pathrename \
	    '^branches/developers/([^/]+)/([^/]+)' \
	    'branches/developers+$${1}+$${2}' | \
	reposurgeon "script autobuilder.lift"
	(cd $@ && git checkout -f main)

# Use svnsync to mirror the upstream repository
$(GCCSDK_REPO_PATH):
	rm -rf $@
	svnadmin create $@
	echo "#!/bin/sh\nexit 0" >$@/hooks/pre-revprop-change
	chmod 755 $@/hooks/pre-revprop-change
	svnsync init file://$@ svn://$(SVN_SERVER)/gccsdk
	svnsync sync file://$@
