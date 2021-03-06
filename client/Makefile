# Standard compiler and linker flags.
PKG_CONFIG ?= pkg-config
override CXXFLAGS := -std=gnu++0x -Wall -Wextra -Wold-style-cast -Wconversion -Wundef -O2 -g $(shell $(PKG_CONFIG) --cflags gtkmm-2.4 protobuf | sed 's/-I/-isystem /g') $(CXXFLAGS)
override LDFLAGS := $(shell $(PKG_CONFIG) --libs-only-L --libs-only-other gtkmm-2.4 protobuf)
override LDLIBS := $(shell $(PKG_CONFIG) --libs-only-l gtkmm-2.4 protobuf)

# The default target.
.PHONY : world
world : testclient

# Gather lists of files of various types.
protos := referee.proto
proto_sources := $(patsubst %.proto,%.pb.cc,$(protos))
proto_headers := $(patsubst %.proto,%.pb.h,$(protos))
proto_objs := $(patsubst %.proto,%.pb.o,$(protos))
non_proto_sources := $(filter-out $(proto_sources),$(wildcard *.cc))
non_proto_headers := $(filter-out $(proto_headers),$(wildcard *.h))
non_proto_objs := $(patsubst %.cc,%.o,$(non_proto_sources))
all_sources := $(proto_sources) $(non_proto_sources)
all_headers := $(proto_headers) $(non_proto_headers)
all_objs := $(proto_objs) $(non_proto_objs)

# Normal rule to link the final binary.
testclient : $(all_objs)
	@echo "LD    $@"
	@$(CXX) $(LDFLAGS) -o $@ $+ $(LDLIBS)

# Static pattern rule to compile a protobuf source file (with warnings disabled, as they make no sense here).
$(proto_objs) : %.pb.o : %.pb.cc $(all_headers)
	@echo "CXX   $@"
	@$(CXX) $(CPPFLAGS) $(CXXFLAGS) -w -c $<

# Static pattern rule to compile a non-protobuf source file.
$(non_proto_objs) : %.o : %.cc $(all_headers)
	@echo "CXX   $@"
	@$(CXX) $(CPPFLAGS) $(CXXFLAGS) -c $<

# Pattern rule to run protoc on a message definition file.
%.pb.cc %.pb.h : ../%.proto
	@echo "PROTO $(patsubst ../%.proto,%.pb.cc,$<)"
	@protoc --proto_path=.. --cpp_out=. $<

# Rule to clean intermediates and outputs.
.PHONY : clean
clean :
	$(RM) testclient *.o *.pb.cc *.pb.h
