#!python
# -*-python-*-

import os
import sys
import atexit
import subprocess
from scons_support import *

def print_build_failures():
    from SCons.Script import GetBuildFailures
    for bf in GetBuildFailures():
        print "%s failed: %s" % (bf.node, bf.errstr)

atexit.register(print_build_failures)
        
def run_tests(env):
    for exe in env['PRJ_EXES'].values():
        cmd = exe[0].abspath
        print "Executing: %s" % cmd
        rv = subprocess.call(cmd)
        if rv == 0:
            print "PASS: %s" % os.path.basename(cmd)
        else:
            print "FAIL: %s" % os.path.basename(cmd)
        
        


def unitTestAction(target, source, env):
    print "PASS: %s", str(source[0].path)


# Get the mode flag from command line
# Default is the release
mode = ARGUMENTS.get('mode','debug')

allowed_modes = {'debug':'dbg', 'release':'rel'}
if not (mode in allowed_modes.keys()):
    print "Error: Expected %s, found %s: " % (allowed_modes.keys(), mode)
    Exit(1)

print "Will build for %s mode..." % mode


use_plat = ARGUMENTS.get('use_plat',0)
use_test = ARGUMENTS.get('use_test',None)

bld_root  = 'build'
platform  = sys.platform
if use_plat:
    print "Using platform '%s' in build output..." % platform
    bld_plat  = '%s/%s' % (bld_root, platform)
else:
    bld_plat = bld_root
    
bld_dir   = '%s/%s' % (bld_plat, allowed_modes[mode])

dump = ARGUMENTS.get('dump',0)


AddOption('--prefix',
          dest='prefix',
          type='string',
          default=bld_root,
          nargs=1,
          action='store',
          metavar='DIR',
          help='installation prefix')

env = Environment(
    MSVC_VERSION='11.0', # VMSVC 2012 choose any version you prefer
    TARGET_ARCH='x86',   # x86 -> 32bit or x86_64 => 64bit
    PREFIX = GetOption('prefix')
)

env.Decider('timestamp-newer')
env.SetOption('implicit_cache', 1)


env.AppendUnique(
    PRJ_BLD_ROOT = Dir(bld_root),
    PRJ_BLD      = Dir(bld_dir),
    PRJ_BLD_BIN  = Dir('%s/bin' % bld_dir),
    PRJ_BLD_LIB  = Dir('%s/lib' % bld_dir),
    PRJ_EXES     = {},
    PRJ_LIBS     = {},
    PRJ_OBJS     = {}
)

env.AppendUnique(
    PRJ_INST_ROOT = Dir(env['PREFIX']),
    PRJ_INST_BIN  = Dir('%s/bin' % env['PREFIX']),
    PRJ_INST_LIB  = Dir('%s/lib' % env['PREFIX'])
)


gsl_headers = FindHeaders('.', 'gsl_*.h', bld_root)
#print gsl_headers

for sh, h in gsl_headers:
    dh = '%s/gsl/%s' % (bld_root, h)
    #print dh, "<-", sh
    env.Command(dh, sh, Copy("$TARGET", "$SOURCE"))


env.AppendUnique(CPPPATH = ['#', '.'])
env.AppendUnique(CPPDEFINES = ['HAVE_CONFIG_H'])
env.AppendUnique(CPPPATH = '#/%s' % env['PRJ_BLD_ROOT'])
env.AppendUnique(LIBPATH = env['PRJ_BLD_LIB'])

if platform == 'win32':
    if 'cl' in env['CC']:
        env.AppendUnique(CPPPATH = ['#/plat/win'])
        env.AppendUnique(CCFLAGS = ['-MT' ])
        #extra compile flags for debug
        #debugcflags = ['-W1', '-GX', '-EHsc', '-D_DEBUG', '/MDd']
        #extra compile flags for release    
        #releasecflags = ['-O2', '-EHsc', '-DNDEBUG', '/MD']         
        if mode == 'debug':
            env.MergeFlags('-W1 -D_DEBUG -RTCs -Zi')
        else:
            # Strangely -O2 resulted in compiler errors
            env.MergeFlags('-O1 -DNDEBUG')
    else:
        print "Unrecognized compiler: %s" % env['CC']
        Exit(1)
elif 'linux' in platform:
    # Replace LINKCOM to position LINKFLAGS at the very end of link command line
    env.Replace(LINKCOM='$LINK -o $TARGET $__RPATH $SOURCES $_LIBDIRFLAGS $_LIBFLAGS $LINKFLAGS')
    env.AppendUnique(LINKFLAGS = ['-lm' ])
    env.AppendUnique(CCFLAGS = ['-fPIC','-DPIC'])
    env.AppendUnique(CPPPATH = ['#/plat/linux'])
    if mode == 'debug':
        env.MergeFlags('-g')
    else:
        env.MergeFlags('-O2 -w')
else:
    print "Unrecognized platform: %s" % platform
    Exit(1)


src_dirs = [
    'utils', 'sys', 'test', 'err', 'ieee-utils', 'const', 'complex', 'cheb',
    'block', 'vector', 'matrix', 'permutation', 'combination', 'multiset',
    'sort', 'cblas', 'blas', 'linalg', 'rng', 'eigen', 'specfunc', 'dht',
    'qrng', 'integration', 'randist', 'fft', 'poly', 'fit', 'statistics',
    'multifit', 'rstat', 'multilarge', 'siman', 'sum', 'interpolation',
    'histogram', 'ode-initval', 'ode-initval2', 'roots', 'multiroots', 'min',
    'multimin', 'monte', 'ntuple', 'diff', 'deriv', 'cdf', 'wavelet',
    'bspline', 'spblas', 'spmatrix', 'splinalg'
]

for sd in src_dirs:
    lib = env.SConscript(
        '%s/SConscript' % sd,
        variant_dir='%s/%s' % (bld_dir,sd),
        duplicate=0,
        exports='env'
    )

gsllib_objs = []
excluded_libs = ['utils', 'gslcblas']
for lib, objs in env['PRJ_OBJS'].iteritems():
    if lib in excluded_libs:
        continue
    gsllib_objs += objs

gsllib_fn = env.subst("$PRJ_BLD_LIB/gsl")
gsllib = env.StaticLibrary(gsllib_fn, gsllib_objs)

for lib in env['PRJ_LIBS'].values():
    env.Install("$PRJ_BLD_LIB", lib)

for exe in env['PRJ_EXES'].values():
    env.Install("$PRJ_BLD_BIN", exe)

    
env.Alias('install', env['PREFIX'])
env.Install("$PRJ_INST_LIB", env['PRJ_LIBS']['gslcblas'])
env.Install("$PRJ_INST_LIB", gsllib)


#TODO: Review later
#env['BUILDERS']['UnitTest'] = env.Builder(action=unitTestAction)

# Create a "test" target for all the unit tests 
for exe in env['PRJ_EXES'].values():
    ta = env.Alias('test', exe, exe[0].abspath)
    #TODO: Review later
    #ta = env.Alias('test', exe, unitTestAction)
    env.AlwaysBuild(ta)
    #print "=>%s" % exe[0].path


Progress(['-\r', '\\\r', '|\r', '/\r'], interval=5)
#Progress('Evaluating $TARGET\r')


if use_test:
    for exe in env['PRJ_EXES'].values():
        cmd = exe[0].abspath
        print "Executing: %s" % cmd
        rv = subprocess.call(cmd)
        if rv == 0:
            print "PASS: %s" % os.path.basename(cmd)
        else:
            print "FAIL: %s" % os.path.basename(cmd)
        

Help("""
Type: 'scons' to build all libraries and tests.
      'scons mode=<mode>' where mode can be debug or release.
      'scons dump=1' to dump construction environment values.
      'scons use_test=1' to run all unit tests (after building everything).

""")

if dump:
    print env.Dump()


