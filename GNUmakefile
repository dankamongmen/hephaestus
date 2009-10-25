.DELETE_ON_ERROR:

.PHONY: test clean

SHELL:=bash -o pipefail

ifeq ($(shell uname),Linux)
XML:=$(shell which xmlstarlet)
else
ifeq ($(shell uname),FreeBSD)
XML:=$(shell which xml)
endif
endif

XSLTDIR:=xslt
CGIBIN:=cgi-bin
RNGDIR:=clients
HEPHDIR:=clients
COMMON:=$(HEPHDIR)/common

XMLTESTIN:=$(wildcard testing/*xml) testing/itinerary.xml
REPORTS:=$(notdir $(wildcard $(CGIBIN)/*))

XMLTESTOUT:=$(foreach report,$(REPORTS),$(subst .xml,.$(report).html,$(XMLTESTIN)))

test: $(XMLTESTOUT)

# The Relax-NG schema used to validate the XML input
RELAXNG:=$(RNGDIR)/hephaestus.rng

COMMONDEPS:=$(RELAXNG) $(COMMON) $(XSLTDIR)/build-xslt.xml

ENV:=XSLTDIR=$(XSLTDIR) HEPHDIR=$(HEPHDIR) RNGDIR=$(RNGDIR) COMMON=$(COMMON)

# FIXME -- apply "$(XML) val -E $@"
%.itinerary.html: %.xml $(CGIBIN)/itinerary $(XSLTDIR)/itinerary-xslt.xml $(COMMONDEPS)
	@$(XML) val -e -r $(RELAXNG) $<
	env $(ENV) $(CGIBIN)/itinerary $< | sed -e '1,2d' > $@

%.users.html: %.xml $(CGIBIN)/users $(XSLTDIR)/users-xslt.xml $(COMMONDEPS)
	@$(XML) val -e -r $(RELAXNG) $<
	env $(ENV) $(CGIBIN)/users $(USER) $< | sed -e '1,2d' > $@

%.repos.html: %.xml $(CGIBIN)/repos $(XSLTDIR)/repos-xslt.xml $(COMMONDEPS)
	@$(XML) val -e -r $(RELAXNG) $<
	env $(ENV) $(CGIBIN)/repos /svn/hephaestus $< | sed -e '1,2d' > $@

%.stats.html: %.xml $(CGIBIN)/stats $(XSLTDIR)/stats-xslt.xml $(COMMONDEPS)
	@$(XML) val -e -r $(RELAXNG) $<
	env $(ENV) $(CGIBIN)/stats $< | sed -e '1,2d' > $@

testing/itinerary.xml:
	wget -O $@ http://svn.research.sys/hephaestus/itinerary.xml

clean:
	svn --xml --no-ignore status | $(XML) sel -t -m //entry -i "wc-status[@item='ignored']" -v @path -n | sudo xargs rm -rf
