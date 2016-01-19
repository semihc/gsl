#!python
# -*-python-*-

import os
import sys
import atexit
import subprocess
import fnmatch
#from scons_support import *
import SConsLib


def findHeaders(rdir, filter, bld_dir):
    """
    Find GSL header files in the source tree excluding build directory
    """
    headers = []
    for root, dirs, files in os.walk(rdir):
        if os.path.abspath(root).startswith(os.path.abspath(bld_dir)):
            continue
        for fn in fnmatch.filter(files, filter):
            afn  = os.path.join(root, fn)
            headers.append((afn, fn))
    return headers


# Construct variables object by merging command-line settings and
# configuration file.
vars = SConsLib.constructVariables('SConsCfg.py')

# Instantiate Scons environment
env = Environment(
    variables = vars,
    MSVC_VERSION='11.0', # VMSVC 2012 choose any version you prefer
    TARGET_ARCH='x86',   # x86 -> 32bit or x86_64 => 64bit
    PREFIX = GetOption('prefix')
)

# Check for unrecognized variables and warn
SConsLib.checkUnknownVariables(vars)

# Setup Scons environment to be used during build
SConsLib.setupEnvironment(env)

# Further refinements to the environment
Help(vars.GenerateHelpText(env))
env.Decider('timestamp-newer')
env.SetOption('implicit_cache', 1)


# Find GSL headers in source tree
gsl_headers = findHeaders('.', 'gsl_*.h', str(env['PRJ_BLD_DIR']))

# Copy the headers
for sh, h in gsl_headers:
    dh = '%s/gsl/%s' % (env['PRJ_BLD_DIR'], h)
    #print dh, "<-", sh
    env.Command(dh, sh, Copy("$TARGET", "$SOURCE"))



# Extra flags for compiler, specific to this project
env.AppendUnique(CPPPATH = ['#', '.'])
env.AppendUnique(CPPDEFINES = ['HAVE_CONFIG_H'])
env.AppendUnique(CPPPATH = '#/%s' % env['PRJ_BLD_DIR'])
env.AppendUnique(LIBPATH = env['PRJ_BLD_LIB'])

if sys.platform == 'win32':
    if 'cl' in env['CC']:
        env.AppendUnique(CPPPATH = ['#/plat/win'])
    else:
        print "Unrecognized compiler: %s" % env['CC']
        Exit(1)
elif 'linux' in sys.platform:
    env.AppendUnique(CPPPATH = ['#/plat/linux'])
else:
    print "Unrecognized platform: %s" % platform
    Exit(1)


# Source directories that we expect to find SConscript files:
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

# Read and process SConscript files
SConsLib.readSConscriptFiles(env, src_dirs)

# Use progress indicator to get feedback from SCons
SConsLib.useProgress()


# Combine all compiled object files of GSL in a single list
gsllib_objs = []
excluded_libs = ['utils', 'gslcblas']
for lib, objs in env['PRJ_OBJS'].iteritems():
    if lib in excluded_libs:
        continue
    gsllib_objs += objs

# Then define the single GSL library uniting them in a single library
gsllib_fn = env.subst("$PRJ_BLD_LIB/gsl")
gsllib = env.StaticLibrary(gsllib_fn, gsllib_objs)

# Add a 'install' target for two special libraries
env.Alias('install', env['PREFIX'])
env.Install("$PRJ_INST_LIB", env['PRJ_LIBS']['gslcblas'])
env.Install("$PRJ_INST_LIB", gsllib)

# All executables in this project are tests also
env['PRJ_TSTS'] = env['PRJ_EXES']
if env['run_tests']:
    SConsLib.runTests(env)

