kentSrc = ../..
include ../../inc/common.mk

ifeq (${USE_TABIX},1)
    TABIX_TESTS=tabixTest vcfTest
else
    TABIX_TESTS=
endif

MYLIBDIR = ../../lib/${MACHTYPE}
MYLIBS = ${MYLIBDIR}/jkweb.a
BIN_DIR = bin/${MACHTYPE}

test: errCatchTest htmlPageTest htmlExpandUrlTest pipelineTests dyStringTest \
    mimeTests base64Tests quotedPTests safeTest hashTest fetchUrlTest gff3Test \
    ${TABIX_TESTS} hacTreeTest testSumDoubles jsonQueryTest
	rm -r output fetchUrlTest testSumDoubles
	@echo tested all


mkdirs:
	${MKDIR} output ${BIN_DIR}

testSumDoubles: testSumDoubles.o ${MYLIBS}
	${CC} ${COPT} -o ./testSumDoubles testSumDoubles.o ${MYLIBS} ${L}

errCatchTest: errCatchTest.o ${MYLIBS} mkdirs
	${CC} ${COPT} -o ${BIN_DIR}/errCatchTest errCatchTest.o ${MYLIBS} ${L}
	${STRIP} ${BIN_DIR}/errCatchTest${EXE}
	${BIN_DIR}/errCatchTest secret > output/errCatch.good
	diff expected/errCatch.good output/errCatch.good
	${BIN_DIR}/errCatchTest bad > output/errCatch.bad
	diff expected/errCatch.bad output/errCatch.bad

htmlExpandUrlTest: htmlExpandUrlTest.o ${MYLIBS} mkdirs
	${CC} ${COPT} -o ${BIN_DIR}/htmlExpandUrlTest htmlExpandUrlTest.o ${MYLIBS} ${L}
	${STRIP} ${BIN_DIR}/htmlExpandUrlTest${EXE}
	${BIN_DIR}/htmlExpandUrlTest > output/htmlExpandUrlTest 2>&1
	diff expected/htmlExpandUrlTest output/htmlExpandUrlTest

htmlPageTest: htmlPageTest.o ${MYLIBS} mkdirs
	${CC} ${COPT} -o ${BIN_DIR}/htmlPageTest htmlPageTest.o ${MYLIBS} ${L}
	${STRIP} ${BIN_DIR}/htmlPageTest${EXE}
	${BIN_DIR}/htmlPageTest input/google.html > output/google.out
	diff expected/google.out output/google.out

pipelineTests: 	pipelineWrite pipelineWriteMult pipelineWriteFd \
		pipelineRead pipelineReadMult pipelineReadFd pipelineReadMem \
		pipelineExitCode pipelineWriteErr pipelineExecError pipelineSigpipe \
		pipelineTimeout

pipelineWrite: ${BIN_DIR}/pipelineTester mkdirs
	${BIN_DIR}/pipelineTester -write -pipeData=input/simple1.txt -otherEnd=output/$@.out.gz "gzip -1"
	gunzip -c output/$@.out.gz > output/$@.out
	diff -b input/simple1.txt output/$@.out

# add come junk to make sure output gets truncated
pipelineWriteMult: ${BIN_DIR}/pipelineTester mkdirs
	cat input/google.html > output/$@.wc
	${BIN_DIR}/pipelineTester -write -pipeData=input/simple1.txt -otherEnd=output/$@.wc "gzip -1" "gzip -dc" "wc"
	diff -b expected/simple1.wc output/$@.wc

pipelineWriteFd: ${BIN_DIR}/pipelineTester mkdirs
	${BIN_DIR}/pipelineTester -fdApi -write -pipeData=input/simple1.txt -otherEnd=output/$@.out.gz "gzip -1"
	gunzip -c output/$@.out.gz > output/$@.out
	diff -b input/simple1.txt output/$@.out

pipelineRead: ${BIN_DIR}/pipelineTester mkdirs
	gzip -1c input/simple1.txt >output/$@.in.gz
	${BIN_DIR}/pipelineTester -otherEnd=output/$@.in.gz -pipeData=output/$@.out "gzip -dc"
	diff -b input/simple1.txt output/$@.out

pipelineReadMult: ${BIN_DIR}/pipelineTester mkdirs
	${BIN_DIR}/pipelineTester -pipeData=output/$@.wc -otherEnd=input/simple1.txt "gzip -1" "gzip -dc" "wc"
	diff -b expected/simple1.wc output/$@.wc

pipelineReadFd: ${BIN_DIR}/pipelineTester mkdirs
	gzip -1c input/simple1.txt >output/$@.in.gz
	${BIN_DIR}/pipelineTester -fdApi -otherEnd=output/$@.in.gz -pipeData=output/$@.out "gzip -dc"
	diff -b input/simple1.txt output/$@.out

pipelineReadMem: ${BIN_DIR}/pipelineTester mkdirs
	gzip -1c input/simple1.txt >output/$@.in.gz
	${BIN_DIR}/pipelineTester -memApi -otherEnd=output/$@.in.gz -pipeData=output/$@.out "gzip -dc"
	diff -b input/simple1.txt output/$@.out

# make sure pipe exit code makes it back
pipelineExitCode: ${BIN_DIR}/pipelineTester
	${BIN_DIR}/pipelineTester -exitCode=13 "sh -c 'exit 13'"

# test redirecting stderr, see that two process can write stderr, but only
# the second's stdout should make it to the end of the pipe.  Since order
# of writes to stderr is determined by process scheduling and when a process
# terminates due to SIGPIPE, just check that stderr was not empty, don't
# check contents.
pipelineWriteErr: ${BIN_DIR}/pipelineTester mkdirs
	${BIN_DIR}/pipelineTester -write -otherEnd=output/$@.out -stderr=output/$@.err "sh -c 'echo OUT; echo ERR >&2'"  "sh -c 'echo OUT2; echo ERR2 >&2'"
	diff -b expected/$@.out output/$@.out
	test -s output/$@.err

# exec a non-existent program
pipelineExecError: ${BIN_DIR}/pipelineTester mkdirs
	if ${BIN_DIR}/pipelineTester -write -stderr=output/$@.err "./thatDoesNotCompute" 2> output/$@.parent.err ; then false else true ; fi
	diff -b expected/$@.err output/$@.err
	diff -b expected/$@.parent.err output/$@.parent.err

# test setting SIGPIPE by generating lots of output and then prematurely closing the
# pipe
pipelineSigpipe: ${BIN_DIR}/pipelineTester mkdirs
	${BIN_DIR}/pipelineTester -sigpipe -maxNumLines=3 -pipeData=/dev/null "awk 'BEGIN {while (1) {print "foo"}}'"

pipelineTimeout: ${BIN_DIR}/pipelineTester mkdirs
	if ${BIN_DIR}/pipelineTester -timeout=2 "bash -c 'sleep 20'" 2> output/$@.parent.err ; then false else true ; fi
	diff -b expected/$@.parent.err output/$@.parent.err

${BIN_DIR}/pipelineTester: pipelineTester.o ${MYLIBS}
	${CC} ${COPT} -o ${BIN_DIR}/pipelineTester pipelineTester.o ${MYLIBS} ${L}


dyStringTest: ${BIN_DIR}/dyStringTester mkdirs
	${BIN_DIR}/dyStringTester

${BIN_DIR}/dyStringTester:  mkdirs dyStringTester.o ${MYLIBS}
	${CC} ${COPT} -o ${BIN_DIR}/dyStringTester dyStringTester.o ${MYLIBS} ${L}


mimeTests: mime1 mime2 mime3 mime4 mimeBin mime5 mimeAltHead mimeAutoBoundary mimeBlat

${BIN_DIR}/mimeTester:  mkdirs mimeTester.o ${MYLIBS}
	${CC} ${COPT} -o ${BIN_DIR}/mimeTester mimeTester.o ${MYLIBS} ${L}


mime1: ${BIN_DIR}/mimeTester mkdirs
	${BIN_DIR}/mimeTester < input/$@.txt > output/$@.out
	diff expected/$@.out output/$@.out

mime2: ${BIN_DIR}/mimeTester mkdirs
	${BIN_DIR}/mimeTester < input/$@.txt > output/$@.out
	diff expected/$@.out output/$@.out

mime3: ${BIN_DIR}/mimeTester mkdirs
	${BIN_DIR}/mimeTester < input/$@.txt > output/$@.out
	diff expected/$@.out output/$@.out

mime4: ${BIN_DIR}/mimeTester mkdirs
	${BIN_DIR}/mimeTester < input/$@.txt > output/$@.out
	diff expected/$@.out output/$@.out

mimeBin: ${BIN_DIR}/mimeTester mkdirs
	${BIN_DIR}/mimeTester < input/$@.txt > output/$@.out
	diff expected/$@.out output/$@.out

mime5: ${BIN_DIR}/mimeTester mkdirs
	${BIN_DIR}/mimeTester < input/$@.txt > output/$@.out
	diff expected/$@.out output/$@.out

mimeAltHead: ${BIN_DIR}/mimeTester mkdirs
	${BIN_DIR}/mimeTester -altHeader='CONTENT_TYPE=multipart/form-data; boundary=----------0xKhTmLbOuNdArY' < input/$@.txt > output/$@.out
	diff expected/$@.out output/$@.out

mimeAutoBoundary: ${BIN_DIR}/mimeTester mkdirs
	${BIN_DIR}/mimeTester -autoBoundary < input/$@.txt > output/$@.out
	diff expected/$@.out output/$@.out

mimeBlat: ${BIN_DIR}/mimeTester mkdirs
	${BIN_DIR}/mimeTester -altHeader='CONTENT_TYPE=multipart/form-data; boundary=----------0xKhTmLbOuNdArY' < input/$@.txt > output/$@.out
	diff expected/$@.out output/$@.out

mimeSeries: ${BIN_DIR}/mimeTester mkdirs
	${BIN_DIR}/mimeTester -sizeSeries=3000

${BIN_DIR}/htmlMimeTest:  mkdirs htmlMimeTest.o ${MYLIBS}
	${CC} ${COPT} -o ${BIN_DIR}/htmlMimeTest htmlMimeTest.o ${MYLIBS} ${L}

htmlMime1: ${BIN_DIR}/htmlMimeTest mkdirs
	${BIN_DIR}/htmlMimeTest https://hgwdev.gi.ucsc.edu/cgi-bin/hgBlat input/htmlMime.txt 3490 3502 > output/$@.out
	diff expected/$@.out output/$@.out


base64Tests: base64Encode base64Decode

${BIN_DIR}/testBase64:  mkdirs testBase64.o ${MYLIBS}
	${CC} ${COPT} -o ${BIN_DIR}/testBase64 testBase64.o ${MYLIBS} ${L}

base64Encode: ${BIN_DIR}/testBase64 mkdirs
	${BIN_DIR}/testBase64 'My Test String' > output/$@.out
	diff expected/$@.out output/$@.out

base64Decode: ${BIN_DIR}/testBase64 mkdirs
	${BIN_DIR}/testBase64 'TXkgVGVzdCBTdHJpbmc=' > output/$@.out
	diff expected/$@.out output/$@.out



quotedPTests: quotedPEncode quotedPDecode

${BIN_DIR}/testQuotedP:  mkdirs testQuotedP.o ${MYLIBS}
	${CC} ${COPT} -o ${BIN_DIR}/testQuotedP testQuotedP.o ${MYLIBS} ${L}

quotedPEncode: ${BIN_DIR}/testQuotedP mkdirs
	${BIN_DIR}/testQuotedP 'taxes are quite high ' > output/$@.out
	diff expected/$@.out output/$@.out

quotedPDecode: ${BIN_DIR}/testQuotedP mkdirs
	${BIN_DIR}/testQuotedP 'taxes=20are=20quite=20high=20=' > output/$@.out
	diff expected/$@.out output/$@.out

${BIN_DIR}/mimeDecodeTest:  mkdirs mimeDecodeTest.o ${MYLIBS}
	${CC} ${COPT} -o ${BIN_DIR}/mimeDecodeTest mimeDecodeTest.o ${MYLIBS} ${L}

mimeDecodeTest: ${BIN_DIR}/mimeDecodeTest mkdirs
	${BIN_DIR}/mimeDecodeTest -cid -autoBoundary output < input/$@.txt
	diff expected/noName1.html output/noName1.html

${BIN_DIR}/safeTester:  mkdirs safeTester.o ${MYLIBS}
	${CC} ${COPT} -o ${BIN_DIR}/safeTester safeTester.o ${MYLIBS} ${L}

safeTest: ${BIN_DIR}/safeTester mkdirs
	${BIN_DIR}/safeTester

hashTest: hashTest1

${BIN_DIR}/testHash:  mkdirs testHash.o ${MYLIBS}
	${CC} ${COPT} -o ${BIN_DIR}/testHash testHash.o ${MYLIBS} ${L}

hashTest1: ${BIN_DIR}/testHash mkdirs
	${BIN_DIR}/testHash input/$@.txt > output/$@.out
	diff expected/$@.out output/$@.out

${BIN_DIR}/testQuotedString: mkdirs testQuotedString.o ${MYLIBS}
	${CC} ${COPT} -o ${BIN_DIR}/testQuotedString testQuotedString.o ${MYLIBS} ${L}

testQuotedString:	${BIN_DIR}/testQuotedString mkdirs
	${BIN_DIR}/testQuotedString -verbose=2 quote this\\ following

miniBlat:  mkdirs miniBlat.o ${MYLIBS}
	${CC} ${COPT} -o miniBlat miniBlat.o ${MYLIBS} ${L}

fetchUrlTest:  mkdirs fetchUrlTest.o ${MYLIBS}
	${CC} ${COPT} -o fetchUrlTest fetchUrlTest.o ${MYLIBS} ${L}

fetchUrlViaUdcTest:  mkdirs fetchUrlViaUdcTest.o ${MYLIBS}
	${CC} ${COPT} -o fetchUrlViaUdcTest fetchUrlViaUdcTest.o ${MYLIBS} ${L}

##
# gff3 tests
##
gff3Tester=${BIN_DIR}/gff3Tester
gff3Test: gff3SacCerTest gff3ErrorCasesTest gff3DiscontiousTest

# FIXME: doesn't work yet
# gff3SpecialCasesTest

gff3SacCerTest: ${gff3Tester} mkdirs
	${gff3Tester} input/sacCerTest.gff3 output/$@.out
	diff expected/$@.out output/$@.out
gff3SpecialCasesTest: ${gff3Tester} mkdirs
	${gff3Tester} input/specialCasesTest.gff3 output/$@.out
	diff expected/$@.out output/$@.out
gff3ErrorCasesTest: ${gff3Tester} mkdirs
	if ${gff3Tester} input/errorCasesTest.gff3 /dev/null >output/$@.err 2>&1 ; then exit 0 else exit 1; fi
	diff expected/$@.err output/$@.err
gff3DiscontiousTest: ${gff3Tester} mkdirs
	${gff3Tester} input/discontinuous.gff3 output/$@.out
	diff expected/$@.out output/$@.out

${BIN_DIR}/gff3Tester: gff3Tester.o ${MYLIBS}
	${MKDIR} ${BIN_DIR}
	${CC} ${COPT} -o ${BIN_DIR}/gff3Tester gff3Tester.o ${MYLIBS} ${L}


# lineFile's tabix support:
tabixTester=${BIN_DIR}/tabixFetch
tabixTest: tabixFetch1kGNoGenotypes tabixFetch1kGWithGenotypes

tabixFetch1kGNoGenotypes: ${tabixTester} mkdirs
	${tabixTester} input/YRI.trio.2010_06.novelsequences.sites.vcf.gz 2:26790860-194631353 > output/$@.out
	diff expected/$@.out output/$@.out

tabixFetch1kGWithGenotypes: ${tabixTester} mkdirs
	${tabixTester} input/YRI.low_coverage.2010_07_excerpt.genotypes.vcf.gz 2:26793738-26794385 > output/$@.out
	diff expected/$@.out output/$@.out

${BIN_DIR}/tabixFetch: tabixFetch.o ${MYLIBS}
	${MKDIR} ${BIN_DIR}
	${CC} ${COPT} -o ${BIN_DIR}/tabixFetch tabixFetch.o ${MYLIBS} ${L}


# vcf:
vcfTester=${BIN_DIR}/vcfParseTest
vcfTest: vcfParse1kGNoGenotypes vcfParse1kGWithGenotypes vcfParseOldV3

vcfParse1kGNoGenotypes: ${vcfTester} mkdirs
	${vcfTester} input/YRI.trio.2010_06.novelsequences.sites.vcf.gz 2 26790859 194631353 > output/$@.out
	diff expected/$@.out output/$@.out

vcfParse1kGWithGenotypes: ${vcfTester} mkdirs
	${vcfTester} input/YRI.low_coverage.2010_07_excerpt.genotypes.vcf.gz 2 26793737 26794385 > output/$@.out
	diff expected/$@.out output/$@.out

vcfParseOldV3:
	${vcfTester} input/20091110_pilot1_vcf_merged_call_sets_YRI.2and3_way.vcf.gz 1 3000 50000 >& output/$@.out
	diff expected/$@.out output/$@.out

${BIN_DIR}/vcfParseTest: vcfParseTest.o ${MYLIBS}
	${MKDIR} ${BIN_DIR}
	${CC} ${COPT} -o ${BIN_DIR}/vcfParseTest vcfParseTest.o ${MYLIBS} ${L}


# hacTree:
hacTreeTester=${BIN_DIR}/hacTreeTest
hacTreeTest: ${hacTreeTester} mkdirs
	${hacTreeTester} input/$@.txt output/$@.out
	diff expected/$@.out output/$@.out

${BIN_DIR}/hacTreeTest: hacTreeTest.o ${MYLIBS}
	${MKDIR} ${BIN_DIR}
	${CC} ${COPT} -o ${BIN_DIR}/hacTreeTest hacTreeTest.o ${MYLIBS} ${L}

# udc (not part of the top-level test target at this point):
udcTest: udcTest.o ${MYLIBS} mkdirs
	${CC} ${COPT} -o ${BIN_DIR}/udcTest udcTest.o ${MYLIBS} ${L}
	${BIN_DIR}/udcTest

# udc (not part of the top-level test target at this point):
udcCacheSizesCheck: udcCacheSizesCheck.o ${MYLIBS} mkdirs
	${CC} ${COPT} -o ${BIN_DIR}/udcCacheSizesCheck udcCacheSizesCheck.o ${MYLIBS} ${L}
	${BIN_DIR}/udcCacheSizesCheck

testOutOfMem: testOutOfMem.o ${MYLIBS} mkdirs
	${CC} ${COPT} -o ${BIN_DIR}/testOutOfMem testOutOfMem.o ${MYLIBS} ${L}
	# we expect this to errAbort because we allocate one byte too much beyond limit
	-${BIN_DIR}/testOutOfMem 100000 1

clean:
	rm -rf *.o bin output *.tmp mimeTester.tmp mimeTester.out fetchUrlTest fetchUrlViaUdcTest

${BIN_DIR}/testDecodedString: mkdirs testDecodedString.o ${MYLIBS}
	${CC} ${COPT} -o ${BIN_DIR}/testDecodedString testDecodedString.o ${MYLIBS} ${L}

testDecodedString:	${BIN_DIR}/testDecodedString mkdirs
	${BIN_DIR}/testDecodedString -verbose=2 quote this\\ following

# jsonQuery:
jsonQueryTester=${BIN_DIR}/jsonQueryTest
jsonQueryTest: ${jsonQueryTester} mkdirs
	${jsonQueryTester} input/$@Json.txt input/$@Path.txt output/$@.out
	diff expected/$@.out output/$@.out

${BIN_DIR}/jsonQueryTest: jsonQueryTest.o ${MYLIBS}
	${MKDIR} ${BIN_DIR}
	${CC} ${COPT} -o ${BIN_DIR}/jsonQueryTest jsonQueryTest.o ${MYLIBS} ${L}

