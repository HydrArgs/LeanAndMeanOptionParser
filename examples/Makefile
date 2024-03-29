CXX=g++
CXXFLAGS=-W -Wall -g -fmessage-length=0 -I../include
CCFLAGS=$(CXXFLAGS) -fno-exceptions -fno-rtti -nodefaultlibs
OPTIMIZE=-Os -fomit-frame-pointer
#OPTIMIZE=-O0
LD=gcc  # gcc automatically picks the correct crt*.o and helper libraries to link
VERSION=1.7
PACKAGEDIR=optionparser-$(VERSION)
TARBALL=../$(PACKAGEDIR).tar.gz

all: example_arg testparse testprintusage testodr example doc

# .cpp files depend on the C++ standard lib
%: %.cpp ../include/optionparser.h
	$(CXX) $(CXXFLAGS) $(OPTIMIZE) -o $@ $<

# .cc files depend only on libc 
%.o: %.cc ../include/optionparser.h
	$(CXX) $(CCFLAGS) $(OPTIMIZE) -c $<

%: %.cc ../include/optionparser.h
	$(CXX) $(CCFLAGS) $(OPTIMIZE) -lc -o $@ $<

testprintusage: testprintusage.cpp ../include/optionparser.h
	$(CXX) $(CXXFLAGS) $(OPTIMIZE) -Wno-unused-result -o $@ $<

# testodr needs to be linked in a separate step rather than
# just passing both .cpp files to g++, because only this way
# can we be sure that duplicate definitions cause an error message.
testodr: testodr1.o testodr2.o
	$(LD) -lc -o $@ $^

doc:
	cd .. && doxygen
	cd .. && cat Doxyfile | sed '/^INTERNAL_DOCS/s/YES/NO/;/^HTML_OUTPUT/s/-dev//' | doxygen -

clean:
	rm -f testodr testodr1.o testodr2.o example.o example testprintusage testparse example_arg
	rm -rf ../html ../html-dev $(PACKAGEDIR) $(TARBALL)
	rm -f *~ ../*~

package:
	rm -rf $(PACKAGEDIR)
	mkdir -p $(PACKAGEDIR)/examples
	cp ../Doxyfile ../DoxygenLayout.xml ../README_ARGC_ARGV $(PACKAGEDIR)
	cp Makefile example.cpp example_arg.cc ../include/optionparser.h \
	   testodr1.cc testodr2.cc testparse.cpp testprintusage.cpp \
	   printUsage.h $(PACKAGEDIR)/examples
	tar --owner=0 --group=0 -czf $(TARBALL) $(PACKAGEDIR)

upload: all package
	cd .. && scp -r include/optionparser.h html/* optionparser-frs:/home/project-web/optionparser/htdocs
	scp $(TARBALL) optionparser-frs:/home/frs/project/o/op/optionparser
