# The build file for the gridcentric nova extension


# The path to the nova source code. Keeping it empty will assume that
# nova is already on the python path.
NOVA_PATH ?=

# The path to the vms source code. Keeping it empty will asusme that
# vms is already on the python path
VMS_PATH ?=

# The name of the test report
TEST_REPORT ?= gc-extension.xml

# Allow a python version spec.
PYTHON_VERSION ?= $(shell python -V 2>&1 |cut -d' ' -f2|cut -d'.' -f1,2)

# Where the collect target goes.
COLLECT_DIR ?= .

# Ensure that the correct python version is used.
PYTHON ?= python$(PYTHON_VERSION)

# This command is used to setup the package directories
INSTALL_DIR := install -d -m0755 -p
INSTALL_BIN := install -m0755 -p 
INSTALL_DATA := install -m0644 -p 

# The version of the extension
VERSION ?= 1.0
RELEASE ?= 0

# This matches the Openstack release version
NOVA_RELEASE ?= unknown

# The timestamp release for the extension
TIMESTAMP := $(shell date "+%Y-%m-%dT%H:%M:%S%:z")

# **** TARGETS ****

all : test package pylint
.PHONY : all

# Build the python egg files
build : build-nova-gridcentric
build : build-nova-api-gridcentric
build : build-novaclient-gridcentric
build : build-nova-gridcentric
.PHONY : build

# Package the extension
package : deb tgz
.PHONY : package

# Build the different packages
build-nova-gridcentric : test
	@rm -rf build/ dist/nova-gridcentric
	@PACKAGE=nova-gridcentric VERSION=$(VERSION) \
	    $(PYTHON) setup.py install --prefix=dist/nova-gridcentric/usr
PHONY: build-nova-gridcentric

build-nova-api-gridcentric : test
	@rm -rf build/ dist/nova-api-gridcentric
	@sed -i -e "s/'.*' ##TIMESTAMP##/'$(TIMESTAMP)' ##TIMESTAMP##/" \
	    gridcentric/nova/osapi/gridcentric_extension.py
	@PACKAGE=nova-api-gridcentric VERSION=$(VERSION) \
	    $(PYTHON) setup.py install --prefix=dist/nova-api-gridcentric/usr
.PHONY: build-nova-api-gridcentric

build-novaclient-gridcentric : test
	@rm -rf build/ dist/novaclient-gridcentric
	@PACKAGE=novaclient-gridcentric VERSION=$(VERSION) \
	    $(PYTHON) setup.py install --prefix=dist/novaclient-gridcentric/usr
.PHONY: build-novaclient-gridcentric

build-nova-compute-gridcentric : test
	@rm -rf build/ dist/nova-compute-gridcentric
	@PACKAGE=nova-compute-gridcentric VERSION=$(VERSION) \
	    $(PYTHON) setup.py install --prefix=dist/nova-compute-gridcentric/usr
	@$(INSTALL_DIR) dist/nova-compute-gridcentric/etc/init
	@$(INSTALL_DATA) etc/nova-gridcentric.conf dist/nova-compute-gridcentric/etc/init
.PHONY: build-nova-compute-gridcentric

deb : deb-nova-gridcentric
deb : deb-nova-api-gridcentric
deb : deb-novaclient-gridcentric
deb : deb-nova-compute-gridcentric
deb-% : build-%
	@rm -rf debbuild && $(INSTALL_DIR) debbuild
	@rsync -ruav packagers/deb/$*/ debbuild
	@rsync -ruav dist/$*/ debbuild
	@sed -i "s/\(^Version:\).*/\1 $(VERSION).$(RELEASE)-$(NOVA_RELEASE)py$(PYTHON_VERSION)/" debbuild/DEBIAN/control
	@dpkg -b debbuild/ .
	@LIBDIR=`ls -1d debbuild/usr/lib*/python*`; mv $$LIBDIR/site-packages $$LIBDIR/dist-packages
	@sed -i "s/\(^Version:\).*/\1 $(VERSION).$(RELEASE)-ubuntu$(NOVA_RELEASE)py$(PYTHON_VERSION)/" debbuild/DEBIAN/control
	@dpkg -b debbuild/ .

tgz : tgz-nova-gridcentric
tgz : tgz-nova-api-gridcentric
tgz : tgz-novaclient-gridcentric
tgz : tgz-nova-compute-gridcentric
tgz-% : build-%
	tar -cvzf $*_$(VERSION).$(RELEASE)-$(NOVA_RELEASE)py$(PYTHON_VERSION).tgz -C dist/$* .

# Runs pylint on the code base
pylint:
	@[ -f pylintrc ] && \
	    pylint --rcfile=pylintrc gridcentric 2>&1 > pylint.txt || true
.PHONY: pylint

# Executes the units tests and generated an Junit XML report
test : testenv
	PYTHONPATH=$(NOVA_PATH):$(VMS_PATH)/src/python nosetests \
	    --with-xunit --xunit-file=build/$(TEST_REPORT) gridcentric || true
.PHONY : test

# Builds the test environment
testenv : 
	mkdir -p build
.PHONY : testenv

clean : 
	rm -f vms.db
	rm -rf build
	rm -rf dist
	rm -f pylint.txt
	rm -rf *.deb debbuild
	rm -rf *.tgz
.PHONY : clean

# Note that COLLECT_DIR needs to be set
collect :
	[ -f build/$(TEST_REPORT) ] && \
	    mkdir -p $(COLLECT_DIR)/test-reports/ && \
	    cp build/$(TEST_REPORT) $(COLLECT_DIR)/test-reports/$(TEST_REPORT) || true
	mkdir -p $(COLLECT_DIR)/deb/ && cp *.deb $(COLLECT_DIR)/deb/ || true
	mkdir -p $(COLLECT_DIR)/tgz/ && cp *.tgz $(COLLECT_DIR)/tgz || true

.PHONE : collect
