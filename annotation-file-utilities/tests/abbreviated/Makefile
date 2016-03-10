# Very rough testing framework for the annotator that is based on the
# testing framework for the Javarifier.  Running 'make all' will look for
# all myClass.goal files in this directory, run the annotator on the
# corresponding .jaif and .java files, and then output the difference in a
# myClass.diff file in this directory.
#
# To test just one file, use (for example) 'make myClass.diff'.

# Put user-specific changes in your own Makefile.user.
# Make will silently continue if that file does not exist.
-include ../Makefile.user

# Override these in Makefile.user if the java and javac commands are not on
# your execution path.  Example from Makefile.user:
#   JAVA=${JAVA_HOME}/bin/java
#   JAVAC=${JAVA_HOME}/bin/javac
JAVA?=java
JAVAC?=javac

export SHELL=/bin/bash -o pipefail


#filter-out reverses order, so use tac to reverse its input first
DIFFS := $(wildcard *.goal)
DISABLED := $(shell grep -le "@skip-test" $(DIFFS))
FILTERED := $(filter-out $(DISABLED),$(DIFFS))
DIFFS := $(patsubst %.goal, %.diff, $(FILTERED))
AFU_JARS := ../../lib/plume.jar ../../annotation-file-utilities.jar
JAIF := C.jaif
SRC := $(patsubst %.goal, %.java, $(FILTERED))
OUT := $(patsubst %.goal, %.output, $(FILTERED))

DEBUG :=
# Use this to enable some debugging.
# DEBUG := --debug

default : all

.PHONY: all
all : $(DIFFS) results

# Display results of all .diff files.
.PHONY: results
results: ../bin/VerifyDiffs.class
	@rm -rf output
	@echo ""
	@echo "=== RESULTS ==="
	@echo ""
	@$(JAVA) -cp bin:../bin VerifyDiffs --show_all

# Remakes the little java program that checks and compares diffs
../bin/VerifyDiffs.class : ../VerifyDiffs.java
	@$(JAVAC) -g -cp ../../bincompile -d ../bin ../VerifyDiffs.java

# Compiles all the test cases (be verbose about this).
.PHONY: compile
compile : $(SRC)
	mkdir -p bin
	$(JAVAC) -g -cp bin:../../bin -d bin -sourcepath . $(SRC)

# Actually runs the annotator to create the annotated java file.
# We are required to put annotation-file-utilities.jar (and ../bin) on the
# bootclasspath so that the JSR 308 javac classes bundled therein are found
# before the stock javac classes that the Mac OS includes on the bootclasspath
# (other platforms do not make such inclusions)
output: compile $(JAIF) ../../bin $(AFU_JARS)
	$(JAVA) \
        -Xbootclasspath/p:../../bin:../../annotation-file-utilities.jar \
	-cp bin \
	annotator.Main \
	${DEBUG} \
	--abbreviate=true \
	-d output \
	$(JAIF) \
	$(SRC) \
	2>&1 | tee C.log

.PRECIOUS: %.output
%.output: output
	find output -name "$*.java" -print | xargs cat > "$*.output"

# Compare the output of the annotator and the goal file.
%.diff: %.goal %.output
	-diff -u $*.goal $*.output >& $*.diff

# Remove all .diff, .log files from the tests directory.
.PHONY: clean
clean :
	rm -rf bin
	rm -f *.diff
	rm -f *.log
	rm -f *.output