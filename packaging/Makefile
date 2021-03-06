SHELL=/bin/bash

ifndef VERSION
VERSION=$(shell grep version.number= ${PWD}/../version.properties | cut -d= -f 2)
endif
ifndef RELEASE
RELEASE=$(shell grep version.release.number= ${PWD}/../version.properties | cut -d= -f 2)
endif
ifndef TAG
TAG=$(shell grep version.tag= ${PWD}/../version.properties | cut -d= -f 2)
endif

# version number used by java artifacts
ifeq ($(TAG),GA)
  JVERS=$(VERSION)
else
  JVERS=$(VERSION)-$(TAG)
endif

TAG?=snapshot

RPMBUILD_DEFINES = --define "_topdir ${PWD}/rpmdist" 
RPMBUILD_DEFINES+= --define "_version ${VERSION}"
RPMBUILD_DEFINES+= --define "_release ${RELEASE}"

ifndef BUILD_NUMBER
ALPHATAG?=$(TAG)_$(shell date "+%Y%m%d%H%M")
else
ALPHATAG?=$(TAG)
RPMBUILD_DEFINES+= --define "_buildnumber $(BUILD_NUMBER)"
endif

ifndef GA_RELEASE
RPMBUILD_DEFINES+= --define "_alphatag $(ALPHATAG)"
DEB_TAG="-$(ALPHATAG)"
endif

ifndef VNAME
VNAME=${VERSION}${DEB_TAG}
endif
RPMBUILD_DEFINES+= --define "_vname ${VNAME}"

.PHONY: rpm deb

rpm:
	@mkdir -p rpmdist/RPMS/noarch
	@mkdir -p rpmdist/SOURCES
	@mkdir -p rpmdist/SOURCES/usr/bin
	@mkdir -p rpmdist/SOURCES/var/lib/rundeck/.ssh
	@mkdir -p rpmdist/SOURCES/var/lib/rundeck/exp/webapp
	@mkdir -p rpmdist/SOURCES/var/lib/rundeck/{data,work,logs,cli,libext,bootstrap}
	@mkdir -p rpmdist/SOURCES/var/lib/rundeck/libext/cache
	@mkdir -p rpmdist/SOURCES/var/log/rundeck
	@mkdir -p rpmdist/SOURCES/var/lib/rundeck/var/tmp/pluginJars
	@mkdir -p rpmdist/SOURCES/tmp/rundeck
	@mkdir -p rpmdist/SOURCES/var/rundeck/projects
	@mkdir -p rpmdist/SOURCES/var/rundeck/user-assets
	@cp -R root/* rpmdist/SOURCES
	@cp -R ../core/build/tools/lib/*.jar rpmdist/SOURCES/var/lib/rundeck/cli
	@cp ../core/build/libs/rundeck-core-$(JVERS).jar rpmdist/SOURCES/var/lib/rundeck/cli
	@cp ../rundeckapp/build/libs/rundeck-$(JVERS).war rpmdist/SOURCES/var/lib/rundeck/bootstrap
	@cp ../plugins/*-plugin/build/libs/*-plugin-$(JVERS).jar rpmdist/SOURCES/var/lib/rundeck/libext
	@cp ../core/src/main/resources/com/dtolabs/launcher/setup/templates/admin.aclpolicy.template rpmdist/SOURCES/etc/rundeck/admin.aclpolicy
	@cp ../core/src/main/resources/com/dtolabs/launcher/setup/templates/apitoken.aclpolicy.template rpmdist/SOURCES/etc/rundeck/apitoken.aclpolicy
	@find ../core/src/main/resources/com/dtolabs/rundeck/core/cli/templates ! -regex ".*.bat$$" -and -type f | sed -e /rd-check/d -e /rd-setup/d | xargs -I Z cp Z rpmdist/SOURCES/usr/bin
	rpmbuild -bb --target=noarch-linux $(RPMBUILD_DEFINES) --buildroot ${PWD}/rpmdist/SOURCES rundeck.spec


deb:
	@mkdir -p debdist/
	@mkdir -p debdist/usr/bin
	@mkdir -p debdist/var/lib/rundeck/exp/webapp
	@mkdir -p debdist/var/lib/rundeck/{data,work,logs,cli,libext,bootstrap}
	@mkdir -p debdist/var/log/rundeck
	@mkdir -p debdist/var/lib/rundeck/var/tmp/pluginJars
	@mkdir -p debdist/tmp/rundeck
	@mkdir -p debdist/var/rundeck/projects
	@mkdir -p debdist/var/rundeck/user-assets
	@cp -R debroot/* debdist
	@# set version
	@cat debroot/DEBIAN/control | sed -e "s/%VERSION%/$(VERSION)/" > debdist/DEBIAN/control
	@cp -R ../core/build/tools/lib/*.jar debdist/var/lib/rundeck/cli
	@cp ../core/build/libs/rundeck-core-$(JVERS).jar debdist/var/lib/rundeck/cli
	@cp ../rundeckapp/build/libs/rundeck-$(JVERS).war debdist/var/lib/rundeck/bootstrap
	@cp ../plugins/*-plugin/build/libs/*-plugin-$(JVERS).jar debdist/var/lib/rundeck/libext
	@cp ../core/src/main/resources/com/dtolabs/launcher/setup/templates/admin.aclpolicy.template debdist/etc/rundeck/admin.aclpolicy
	@cp ../core/src/main/resources/com/dtolabs/launcher/setup/templates/apitoken.aclpolicy.template debdist/etc/rundeck/apitoken.aclpolicy
	@find ../core/src/main/resources/com/dtolabs/rundeck/core/cli/templates ! -regex ".*.bat$$" -and -type f | sed -e /rd-check/d -e /rd-setup/d | xargs -I Z cp Z debdist/usr/bin
	fakeroot dpkg-deb --build -Zgzip debdist
	@mv debdist.deb debdist/rundeck_$(VERSION)-$(RELEASE)$(DEB_TAG)_all.deb

clean: rpmclean debclean

rpmclean:
	rm -rf rpmdist

debclean:
	rm -rf debdist rundeck*.deb
