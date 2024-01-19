all:

.PHONY: all clean

ifneq ($(wildcard nodepool/.),)
ifneq ($(wildcard depcache/.),)

PACKAGES :=

define pkgname =
$(shell (cd $(1) && git pull) >/dev/null 2>&1 && Rscript -e "\
 cat(read.dcf('$(1)/DESCRIPTION')[,c('Package','Version')], sep = '_'); \
 cat('.tar.gz') \
")
endef

define pkgdef =
$(2):
	R CMD build $(1)
PACKAGES += $(2)
endef

define defpkg =
$(call pkgdef,$(1),$(call pkgname,$(1)))
endef

$(eval $(call defpkg,nodepool))
$(eval $(call defpkg,depcache))
endif
endif

ifdef PACKAGES

.PHONY: check repo commit

all: commit

check: $(PACKAGES)
	R CMD check $(PACKAGES)

repo: PACKAGES.rds
PACKAGES.rds: PACKAGES.gz
PACKAGES.gz: PACKAGES
PACKAGES: $(PACKAGES) check
	Rscript -e 'tools::write_PACKAGES(verbose = TRUE)'

commit: repo
	git checkout -B repo makefile
	git add PACKAGES* *.tar.gz
	git commit -m "Publish $(PACKAGES)"
	git push -f origin repo
	git checkout makefile

else

nodepool:
	git clone https://github.com/aitap/nodepool $@

depcache:
	git clone --branch serialize_canonical https://github.com/aitap/depcache $@

# Cannot set $(PACKAGES) without having cloned the repo
# (thankfully, this is needed only once)
all: nodepool depcache
	$(MAKE)

endif

clean:
	-rm -rf src PACKAGES* *.tar.gz *.Rcheck
