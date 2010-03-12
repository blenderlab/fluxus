# -*- mode: python; -*-
################################################################################
# Top level SConscript for fluxus
#
# Checks all dependencies needed, builds the fluxus canvas
# application, then calls the sconscripts for libfluxus and
# the fluxus PLT modules

import os, sys, commands, subprocess

MajorVersion = "0"
MinorVersion = "17"
FluxusVersion = MajorVersion+MinorVersion
# remember to change fluxa too...
Target = "fluxus"

# changed prefix and pltprefix so they can be invoked at runtime
# like scons Prefix=/usr PLTPrefix=/usr instead of default /usr/local

DESTDIR = ARGUMENTS.get('DESTDIR', '')
# this makes DESTDIR relative to root of the source tree, no matter
# where we are
if len(DESTDIR)>0 and DESTDIR[0] != "/":
                DESTDIR = "#" + DESTDIR

if sys.platform == 'darwin':
	AddOption('--app', action='store_true', help='Build OSX application')
	file = os.popen('dirname "`which mzscheme`"')
	PLTBin = file.read()
	file.close()
	Prefix = ARGUMENTS.get('Prefix','/opt/local')
	PLTPrefix = ARGUMENTS.get('PLTPrefix', PLTBin[:-5])
	PLTInclude = ARGUMENTS.get('PLTInclude', PLTPrefix + "/include")
	PLTLib = ARGUMENTS.get('PLTLib', PLTPrefix + "/lib")
elif sys.platform == 'win32':
	Prefix = ARGUMENTS.get('Prefix','c:/Program Files/Fluxus')
	PLTPrefix = ARGUMENTS.get('PLTPrefix','c:/Program Files/PLT')
	PLTInclude = ARGUMENTS.get('PLTInclude', PLTPrefix + "/include")
	PLTLib = ARGUMENTS.get('PLTLib', PLTPrefix + "/lib")
else:
	Prefix = ARGUMENTS.get('Prefix','/usr/local')
	PLTPrefix = ARGUMENTS.get('PLTPrefix','/usr/local')
	PLTInclude = ARGUMENTS.get('PLTInclude', PLTPrefix + "/include/plt")
	PLTLib = ARGUMENTS.get('PLTLib', PLTPrefix + "/lib/plt")
BinInstall = DESTDIR + Prefix + "/bin"

DataLocation = Prefix + "/share/fluxus-"+FluxusVersion
DataInstall = DESTDIR + DataLocation
FluxusCollectsLocation = Prefix + "/lib"
CollectsInstall = DESTDIR + FluxusCollectsLocation + "/fluxus-" + FluxusVersion
DocsInstall = DESTDIR + Prefix + "/share/doc/fluxus-" + FluxusVersion

if sys.platform == 'darwin' or sys.platform == 'win32':
        PLTCollectsLocation = PLTPrefix + "/collects/"
else:
        PLTCollectsLocation = PLTLib + "/collects/"

if sys.platform == 'darwin' and GetOption('app'):
        PLTCollectsLocation = '/Applications/Fluxus.app/Contents/Resources/collects/'
        FluxusCollectsLocation = '/Applications/Fluxus.app/Contents/Resources/collects/'
        DataLocation = '/Applications/Fluxus.app/Contents/Resources'

LibPaths     = [
        PLTLib,
        PLTLib+"/..",
        "/usr/lib",
        "../../libfluxus"]

IncludePaths = [
        "/usr/local/include",
        "/usr/include",
        "/usr/local/include/freetype2",  # arg - freetype needs to be
        "/usr/include/freetype2",        # on the include path :(
        PLTInclude,
        "../../libfluxus/src"]

################################################################################
# Make the "one" environment for building and installing

paranoid = ' -W -Wcast-qual -Wwrite-strings -Wcast-align -Wpointer-arith -Wconversion -Wredundant-decls -Wunreachable-code -Winline -Wlarger-than-256'

env = Environment(CCFLAGS = '-ggdb -pipe -Wall -O3 -ffast-math -Wno-unused -fPIC',
                  VERSION_NUM = FluxusVersion)
env.MergeFlags(ARGUMENTS.get('CCFLAGS', ''))

if env['PLATFORM'] == 'win32':
	IncludePaths += [ "/MinGW/include/freetype2" ]
	LibPaths += [ "/MinGW/lib" ]

if env['PLATFORM'] == 'darwin':
	IncludePaths += ['/opt/local/include', '/opt/local/include/freetype2']
	LibPaths += ['/opt/local/lib']

env.Append(CPPPATH = IncludePaths)
env.Append(LIBPATH = LibPaths)
env.Append(CCFLAGS=' -DFLUXUS_MAJOR_VERSION='+MajorVersion)
env.Append(CCFLAGS=' -DFLUXUS_MINOR_VERSION='+MinorVersion)
env.Append(CCFLAGS=" -DPLT_COLLECTS_LOCATION="+"\"\\\""+PLTCollectsLocation+"\"\\\"")
env.Append(CCFLAGS=" -DFLUXUS_COLLECTS_LOCATION="+"\"\\\""+FluxusCollectsLocation+"\"\\\"")
env.Append(CCFLAGS=" -DDATA_LOCATION="+"\"\\\""+DataLocation+"\"\\\"")

if ARGUMENTS.get("GLSL","1")=="1":
        env.Append(CCFLAGS=' -DGLSL')

if ARGUMENTS.get("STEREODEFAULT","0")=="1":
        env.Append(CCFLAGS=' -DSTEREODEFAULT')

if ARGUMENTS.get("ACCUM_BUFFER","0")=="1":
        env.Append(CCFLAGS=' -DACCUM_BUFFER')

if ARGUMENTS.get("MULTISAMPLE","0")=="1":
        env.Append(CCFLAGS=' -DMULTISAMPLE')

if ARGUMENTS.get("MULTITEXTURE","1")=="0":
        env.Append(CCFLAGS=' -DDISABLE_MULTITEXTURE')

if ARGUMENTS.get("RELATIVE_COLLECTS","0")=="1":
	env.Append(CCFLAGS=' -DRELATIVE_COLLECTS')

static_modules=0
if ARGUMENTS.get("STATIC_MODULES","0")=="1":
	static_modules=1
	env.Append(CCFLAGS=' -DSTATIC_LINK')

static_everything=0
if ARGUMENTS.get("STATIC_EVERYTHING","0")=="1":
	static_everything=1
	static_modules=1
	env.Append(CCFLAGS=' -DSTATIC_LINK')

static_ode=int(ARGUMENTS.get("STATIC_ODE","0"))

# need to do this to get scons to link plt's mzdyn.o
env["STATIC_AND_SHARED_OBJECTS_ARE_THE_SAME"]=1
MZDYN = PLTLib + "/mzdyn.o"

if env['PLATFORM'] == 'win32':
	# need to do this to get scons to link plt's mzdyn.o
	MZDYN = PLTLib + "/gcc/mzdyn.o"

	if ARGUMENTS.get("3M","1")=="1":
		env.Append(CCFLAGS=' -DMZ_PRECISE_GC')
		MZDYN = PLTLib + "/gcc/mzdyn3m.o"
else:
	# need to do this to get scons to link plt's mzdyn.o
	MZDYN = PLTLib + "/mzdyn.o"
	
	if ARGUMENTS.get("3M","1")=="1":
		env.Append(CCFLAGS=' -DMZ_PRECISE_GC')
		MZDYN = PLTLib + "/mzdyn3m.o"

################################################################################
# Figure out which libraries we are going to need

# First member of each list is a library, second - a header or headers list
# to be passed to the CheckLibWithHeader(...) at configure time.
# We may add extra libraries later on per platform basis
LibList = [["m", "math.h"],
                ["pthread", "pthread.h"],
                ["dl", "stdio.h"],
                ["mzscheme3m", PLTInclude + "/scheme.h"],
                ["jpeg", ["stdio.h", "stdlib.h", "jpeglib.h"]],
                ["tiff", "tiff.h"],
                ["freetype", "ft2build.h"],
                ["z", "zlib.h"],
                ["png", "png.h"],
                ["ode", "ode/ode.h"],
                ["sndfile", "sndfile.h"],
                ["fftw3", "fftw3.h"],
                ["lo", "lo/lo.h"],
                ["GLEW", "GL/glew.h"]]

if env['PLATFORM'] == 'win32':
	LibList = [["m", "math.h"],
			["freetype", "ft2build.h"],
			["glew32", "GL/glew.h"],
			["glut32", "GL/glut.h"],
			["glu32", "GL/glu.h"],
			["opengl32", "GL/gl.h"],
			["openal32", "AL/al.h"],
			["libmzsch3m_6ncc9s", PLTInclude + "/scheme.h"]]

if env['PLATFORM'] == 'posix':
        env.Prepend(LINKFLAGS = ["-rdynamic"])
        LibList += [["GL", "GL/gl.h"],
                    ["GLU", "GL/glu.h"],
                    ["glut", "GL/glut.h"],
                    ["asound", "alsa/asoundlib.h"],
                    ["openal", "AL/al.h"]]

elif env['PLATFORM'] == 'darwin':
        # add jack as a library if not making an app
        if not GetOption('app'):
            LibList += [["jack", "jack/jack.h"]]
        env.Append(FRAMEWORKPATH = [PLTLib])

        if GetOption('app'):
            env.Append(CCFLAGS = ' -D__APPLE_APP__ -DRELATIVE_COLLECTS')
            # FIXME: check if Jackmp is available when making an app
            env.Append(FRAMEWORKS = Split("GLUT OpenGL CoreAudio CoreFoundation PLT_MrEd Jackmp"))
        else:
            env.Append(FRAMEWORKS = Split("GLUT OpenGL CoreAudio PLT_MrEd"))


################################################################################
# Make sure we have these libraries availible

if not GetOption('clean'):
        print '--------------------------------------------------------'
        print 'Fluxus: Configuring Build Environment'
        print '--------------------------------------------------------'
        conf = Configure(env)

        # check PLT and OpenAL frameworks on osx
        if env['PLATFORM'] == 'darwin':
                if not conf.CheckHeader('scheme.h'):
                        print "ERROR: 'mzscheme3m' must be installed!"
                        Exit(1)
                LibList = filter(lambda x: x[0] != 'mzscheme3m', LibList)
                # OpenAL should be installed everywhere
                if not conf.CheckHeader('OpenAL/al.h'):
                        print "ERROR: 'OpenAL' must be installed!"
                        Exit(1)

        # all libraries are required, and some of them require each other,
        # hence the order is important, and autoadd=1
        for (lib,headers) in LibList:
                if not conf.CheckLibWithHeader(lib, headers, 'C', autoadd = 1):
                        print "ERROR: '%s' must be installed!" % (lib)
                        Exit(1)

        if not conf.CheckFunc("dInitODE2"):
            env.Append(CCFLAGS=' -DGOODE_OLDE_ODE')

        # the liblo version 0.25 does not include the declaration of lo_arg_size anymore
        # This will be re-included in future version
        if not conf.CheckFunc("lo_arg_size_check", "#include <lo/lo.h>\n#define lo_arg_size_check() lo_arg_size(LO_INT32, NULL)", "C++"):
            env.Append(CCFLAGS=' -DNO_LO_ARG_SIZE_DECL')

        env = conf.Finish()
        # ... but we shouldn't forget to add them to LIBS manually
        env.Replace(LIBS = [rec[0] for rec in LibList])

# replace libs with static libs if building an osx app
if env['PLATFORM'] == 'darwin' and GetOption('app'):
	for l in ['png', 'tiff', 'GLEW', 'z', 'sndfile', 'fftw3', 'freetype', 'ode', 'jpeg']:
		env['LIBS'].remove(l)
		env['LIBS'].append(File('/opt/local/lib/lib%s.a' % l))

# detect ode precision
if not GetOption('clean'):
	try:
		odec = subprocess.Popen(['ode-config', '--cflags', '--libs'], stdout=subprocess.PIPE)
		ode_str = odec.communicate()
		if isinstance(ode_str[0], str):
			env.MergeFlags(ode_str[0])
			if ode_str[0].find('-DdDOUBLE') > -1:
				print 'WARNING: ODE double precision installation detected'
				print 'WARNING: you might encounter problems when using the fluxus physics system'
			else:
				print 'ODE single precision detected'
	except:
		print 'WARNING: unable to run ode-config, cannot detect ODE precision'

# link ode statically
if static_ode and not GetOption('clean'):
	if 'ode' in env['LIBS']:
		env['LIBS'].remove('ode')
		env['LIBS'].append(File('%s/lib/libode.a' % Prefix))

################################################################################
# Build the fluxus application
Install = BinInstall

# need to build the bytecode for the base scheme library
# this is the wrong place to do this
if not GetOption('clean'):
	if static_modules:
		# in static mode, we want to embed all of plt we need
		# using popen as it's crossplatform 
		mzc_status = os.popen("mzc --c-mods src/base.c \
			++lib scheme/base  \
			++lib scheme/base/lang/reader  \
			++lib xml/xml \
			++lib compiler \
			++lib mzscheme \
			++lib mzlib/string \
			++lib setup   \
			++lib config  \
			++lib stxclass").close()
	else:
		#mzc_status = os.popen("mzc --c-mods src/base.c ++lib scheme/base").close()
		mzc_status = subprocess.call(['mzc', '--c-mods', 'src/base.c', '++lib', 'scheme/base'])

	if mzc_status != 0:
		print "ERROR: Failed to run command 'mzc'"
		Exit(1)


Source = ["src/GLEditor.cpp",
          "src/GLFileDialog.cpp",
          "src/Interpreter.cpp",
          "src/Repl.cpp",
          "src/Recorder.cpp",
          "src/FluxusMain.cpp",
          "src/PolyGlyph.cpp",
          "src/Unicode.cpp",
          "src/main.cpp"]

app_env = env.Clone()

# statically link all the modules
if not GetOption('clean') and static_modules:
		
	# statically link in all the fluxus modules
	# these pick up the 'libfluxus-engine_ss.a' libs
	# rather than .so due to the 'lib' at the start
	app_env.Append(LIBPATH = ["modules/fluxus-engine/"])
	app_env.Append(LIBPATH = ["modules/fluxus-osc/"])
	app_env.Append(LIBPATH = ["modules/fluxus-audio/"])
	app_env.Append(LIBPATH = ["modules/fluxus-midi/"])
	app_env.Append(LIBPATH = ["libfluxus/"])
	app_env.Append(LIBS = ["fluxus-engine_ss"])
	app_env.Append(LIBS = ["fluxus-osc_ss"])
	app_env.Append(LIBS = ["fluxus-audio_ss"])
	app_env.Append(LIBS = ["fluxus-midi_ss"])
	
	if static_everything:
		# this is all a bit fragile, due to the need to put all the 
		# dependancies of the dynamic libraries here, in the right order
		
		# start off with the first options
		linkcom = " -static-libgcc -Wl,-Bstatic -lstdc++ -ljack -lrt -lasound \
			-lfluxus -lpthread_nonshared "
	
		app_env['LIBS'].remove("pthread")
		app_env['LIBS'].remove("dl")
	
		# now go through the rest of the libs, removing them from 
		# the environment at the same time
		for i in " GLEW GLU glut asound m ode fftw3 mzscheme3m png tiff \
					jpeg freetype sndfile lo z ".split():
			app_env['LIBS'].remove(i)
			linkcom+="-l"+i+" "
	
		# add the remaining dependancies
		app_env.Append(LINKCOM = linkcom + ' -lFLAC -logg -Wl,-Bdynamic')
	else:
		# statically link in mzscheme only
		if env['PLATFORM'] == 'win32':
			app_env['LIBS'].remove('libmzsch3m_6ncc9s')
		else:
			app_env['LIBS'].remove('mzscheme3m')

		app_env.Append(LINKCOM = ' -Wl,-Bstatic -lmzscheme3m -Wl,-Bdynamic')
		
		# have to add the libs needed by the fluxus modules here
		app_env.Append(LIBS = ["fluxus"])
		app_env.Append(LIBS = ["jack"])
		app_env.Append(LIBS = ["asound"])

app_env.Program(source = Source, target = Target)

################################################################################
# Build everything else
# call the core library builder and the scheme modules

if env['PLATFORM'] == 'win32':
	SConscript(dirs = Split("libfluxus modules"),
           exports = ["env", "CollectsInstall", "DataInstall", "MZDYN", "BinInstall", \
				   "static_modules", "static_ode", "Prefix"])

else:
	SConscript(dirs = Split("libfluxus modules fluxa addons"),
           exports = ["env", "CollectsInstall", "DataInstall", "MZDYN", "BinInstall", \
			   "static_modules", "static_ode", "Prefix"])


################################################################################
# documentation

Docs = ["docs/fluxus-documentation-en.pdf",
	"docs/fluxus-documentation-fr.pdf",
	"AUTHORS", "CHANGES", "COPYING", "LICENCE", "README"]

Examples = [ "examples" ]

################################################################################
# packaging / installing
if env['PLATFORM'] == 'darwin' and GetOption('app'):
        from macos.osxbundle import *
        TOOL_BUNDLE(env)
        # add dynamic libs
        frameworks = [PLTLib + '/PLT_MrEd.framework',
                     '/Library/Frameworks/Jackmp.framework']
        dylibs = [ '/opt/local/lib/liblo.dylib']

        env.Alias('app', env.MakeBundle('Fluxus.app',
                                        Target,
                                        'key',
                                        'packages/macos/fluxus-Info.plist',
                                        dylibs = dylibs,
                                        frameworks = frameworks,
                                        resources=[['modules/material/fonts/', 'material/fonts/'],
                                                   ['modules/material/meshes/', 'material/meshes/'],
                                                   ['modules/material/shaders/', 'material/shaders/'],
                                                   ['modules/material/textures/', 'material/textures/'],
                                                   ['modules/scheme/', 'collects/fluxus-%s/' % FluxusVersion],
                                                   ['modules/fluxus-engine/fluxus-engine_ss.dylib',
                                                       'collects/fluxus-' + FluxusVersion + '/compiled/native/i386-macosx/3m/fluxus-engine_ss.dylib'],
                                                   ['modules/fluxus-audio/fluxus-audio_ss.dylib',
                                                       'collects/fluxus-' + FluxusVersion + '/compiled/native/i386-macosx/3m/fluxus-audio_ss.dylib'],
                                                   ['modules/fluxus-midi/fluxus-midi_ss.dylib',
                                                       'collects/fluxus-' + FluxusVersion + '/compiled/native/i386-macosx/3m/fluxus-midi_ss.dylib'],
                                                   ['modules/fluxus-osc/fluxus-osc_ss.dylib',
                                                       'collects/fluxus-' + FluxusVersion + '/compiled/native/i386-macosx/3m/fluxus-osc_ss.dylib'],
												   ['modules/fluxus-openal/fluxus-openal_ss.dylib',
													   'collects/fluxus-' + FluxusVersion + '/compiled/native/i386-macosx/3m/fluxus-openal_ss.dylib'],
												   ['addons/video/fluxus-video_ss.dylib',
													   'collects/fluxus-' + FluxusVersion + '/compiled/native/i386-macosx/3m/fluxus-video_ss.dylib'],
												   ['addons/artkp/fluxus-artkp_ss.dylib',
													   'collects/fluxus-' + FluxusVersion + '/compiled/native/i386-macosx/3m/fluxus-artkp_ss.dylib']],

                                        typecode='APPL',
                                        icon_file='packages/macos/fluxus.icns'))
        # build dmg
        '''
        env['BUILDERS']['DiskImage'] = Builder(action = BuildDmg)
        DmgFiles = [File('AUTHORS'), File('CHANGES'), File('LICENCE'), \
                File('README'), File('COPYING'), Dir("Fluxus.app"), \
                Dir("examples")]
        env.Alias("dmg", env.DiskImage('Fluxus-' + FluxusVersion + '.dmg',
                                       DmgFiles))
        '''

        # fluxa
        frameworks = [ '/Library/Frameworks/Jackmp.framework']
        dylibs = [ '/opt/local/lib/liblo.dylib']
        env.Alias('app', env.MakeBundle('Fluxa.app',
                                        'fluxa/fluxa',
                                        'key',
                                        'packages/macos/fluxa-Info.plist',
                                        dylibs = dylibs,
                                        frameworks = frameworks,
                                        typecode='APPL',
                                        icon_file='packages/macos/fluxa.icns'))

if (env['PLATFORM'] == "win32"):
	Target+=".exe"

env.Install(DocsInstall, Docs)
env.Install(DocsInstall, Examples)
env.Install(Install, Target)
env.Alias('install', [DESTDIR + Prefix, CollectsInstall])

