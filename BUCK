#Dependencies
remote_file(
    name = 'sbe-tool',
    url = 'mvn:uk.co.real-logic:sbe-tool:jar:1.6.0',
    out = 'sbe-tool-1.6.0.jar',
    sha1 = '4545e5cad049191d6ca449ca6d32c5008231459b',
)

remote_file(
    name = 'agrona',
    url = 'mvn:org.agrona:agrona:jar:0.9.3',
    out = 'agrona-0.9.3.jar',
    sha1 = '3d2cb221f620b043cc20bde4004834b9d3496a81',
)

remote_file(
    name = 'agrona-source',
    url = 'mvn:org.agrona:agrona:sources:0.9.3',
    out = 'agrona-sources-0.9.3.jar',
    sha1 = 'ec87ca0b9700838d2577c465cff19396043916f4',
)

prebuilt_jar(
    name = 'agrona-jar',
    binary_jar = ':agrona',
    source_jar = ':agrona-source',
)

prebuilt_jar(
    name = 'sbe-tool-jar',
    binary_jar = ':sbe-tool',
)

java_binary(
    name = 'sbe-tool-bin',
    deps = [':sbe-tool-jar', ':agrona-jar' ],
    main_class = 'uk.co.real_logic.sbe.SbeTool'
)

#Where the magic happens
def generate_sbe_stubs(name, namespace, srcs=[]):
    genrule(
        name = name + '-sbe-tool-gen',
        cmd = 'java -jar -Dsbe.target.namespace='
              + namespace +
              ' -Dsbe.output.dir=$OUT/../ $(location :sbe-tool-bin) $SRCS',
        srcs = srcs,
        out = namespace.split('.')[0]
    )

    zip_file(
        name = name + '-sources',
        srcs = [':' + name + '-sbe-tool-gen'],
        out = name + '-sources.src.zip'
    )

    java_library(
        name = name + '-build-jar',
        srcs = [':' + name +'-sources'],
        deps = [':agrona-jar']
    )

    prebuilt_jar(
        name = name,
        visibility = ['PUBLIC' ],
        binary_jar = ':' + name + '-build-jar',
        source_jar = ':' + name + '-sources'
    )

#Use that magic
generate_sbe_stubs('sbe-build', 'sbe_build', ['example-schema.xml'])

#Bundle the magic and our sources
java_library(
    name = 'result',
    srcs = glob(['src/main/java/**']),
    deps = [':sbe-build', ':agrona-jar' ],
)

java_binary(
    name = 'result-bin',
    deps = [':result'],
    main_class = 'net.lfn3.buck_sbe_tool.Core'
)
