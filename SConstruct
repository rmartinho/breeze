# --- configuration ---

projectName = 'untitled'

language = 'c++11'
ignoredWarnings = [ 'mismatched-tags' ]
noRebuild = [ '-Wfatal-errors' ]

systemLibs = []
otherLibs = []

infoFiles = Split('README.md COPYING.txt')

# --- targets ---

def makeTargets(base):
    debug = makeDebug(base, 'debug')
    release = makeRelease(base, 'release')

    makeTest(debug, 'test-debug')
    makeTest(release, 'test')

    Default('test')

# --- functions ---

def makeOptions():
    options = Variables()
    options.Add(EnumVariable('lib', 'library to build', 'static', allowed_values=('static', 'shared')))
    options.Add(BoolVariable('fatal', 'stop on first error', True))
    return options

import os
import sys
def makeBaseEnvironment(opts):
    if sys.platform == 'win32':
        env = Environment(options = opts, ENV = os.environ, tools = ['mingw'])
    else:
        env = Environment(options = opts, ENV = os.environ)

    Help(opts.GenerateHelpText(env))

    setWarnings(env)
    setLanguage(env)

    setStructure(env)

    setLibs(env)

    setPlatformMacros(env)
    setVisibilityMacros(env)
    setVersionMacros(env)

    return env

def setWarnings(env):
    base = Split('-Wall -Wextra -Werror -pedantic-errors')
    ignored = prefix('-Wno-', ignoredWarnings)
    fatal = [ '-Wfatal-errors' ] if env['fatal'] else []
    
    env.MergeFlags(base + fatal + ignored)

def setLanguage(env):
    env.MergeFlags([ '-std=' + language ])
    env['LINK'] = env['CXX']

def setStructure(env):
    env.Append(CPPPATH = ['include'])
    env.Append(LIBPATH = ['lib'])

def setLibs(env):
    if len(systemLibs) > 0:
        env.ParseConfig('pkg-config --cflags --libs ' + ' '.join(systemLibs))
    env.Append(LIBS = otherLibs)

def setPlatformMacros(env):
    if env['PLATFORM'] == 'win32':
        macros = ['WINDOWS']
    else:
        macros = ['POSIX']
    env.Append(CPPDEFINES = projectMacroList(macros))

def setVisibilityMacros(env):
    if env['lib'] == 'static':
        macros = ['BUILD']
    if env['lib'] == 'shared':
        macros = ['BUILD', 'SHARED']
    env.Append(CPPDEFINES = projectMacroList(macros))

def setVersionMacros(env):
    with open('VERSION', 'r') as f:
        version = f.read().strip()

    major, minor, trail = version.split('.')
    revision, info = trail.split('-')

    macros = [ 'VERSION_MAJOR=' + major
             , 'VERSION_MINOR=' + minor
             , 'VERSION_REVISION=' + revision
             , 'VERSION_INFO=' + info
             ]

    env['version'] = version

    env.Append(CPPDEFINES = projectMacroList(macros))

import os.path
def cloneLib(base, alias, config):
    env = base.Clone()
    env['config'] = config

    target = os.path.join(binDir(config), simpleName(projectName))
    env.VariantDir(objDir(config), '.', duplicate = 0)
    sources = getObjectTargets('src', config)
    if env['lib'] == 'static':
        lib = env.StaticLibrary(target, sources)
    if env['lib'] == 'shared':
        lib = env.SharedLibrary(target, sources)
    env.Alias(alias, lib)

    return env

def getObjectTargets(folder, config):
    return baseFiles(objDir(config), getFiles(folder, '*.c++'))

def makeDebug(base, alias):
    env = cloneLib(base, alias, 'debug')
    env.MergeFlags([ '-g' ])
    env.Append(CPPDEFINES = [ '_GLIBCXX_DEBUG' ])
    return env

def makeRelease(base, alias):
    env = cloneLib(base, alias, 'release')
    env.MergeFlags([ '-O3', '-flto' ])
    env.Append(CPPDEFINES = [ 'NDEBUG' ])
    env.Append(LINKFLAGS = [ '-flto' ])
    return env

def makeTest(base, alias):
    env = base.Clone()
    config = env['config']

    env.Append(LIBS = simpleName(projectName))
    env.Append(LIBPATH = binDir(config))

    env.Append(CPPPATH = [ 'test' ])

    target = os.path.join(binDir(config), 'runtest')
    sources = getObjectTargets('test', config)
    program = env.Program(target, sources)
    env.AlwaysBuild(env.Alias(alias, [program], target))

    return env

# --- generic utils ---

def prefix(base, items):
    return map(lambda e: base + e, items)

import re
def simpleName(s):
    s = s.strip()
    s = re.sub(r'\s+', '_', s)
    s = re.sub(r'[^a-zA-Z0-9]+', '_', s)
    return s

def macroPrefix(s):
    return simpleName(s).upper() + '_'

def projectMacroList(macros):
    return prefix(macroPrefix(projectName), macros)

def baseFiles(base, files):
    return map(lambda e: os.path.join(base, e), files)

def objDir(config):
    return os.path.join('obj', config)
def binDir(config):
    return os.path.join('bin', config)

import fnmatch
def getFiles(root, pattern):
    pattern = fnmatch.translate(pattern)
    for dir, dirs, files in os.walk(root):
        for f in files:
            if re.match(pattern, f):
                yield os.path.join(dir, f)

# --- hacks ---

import SCons
import types
def avoidRebuilds(flags):
    SCons.Action.ActionBase.get_contents_orig = SCons.Action.ActionBase.get_contents
    def get_contents(self, *args, **kw):
        cnt = self.get_contents_orig(*args, **kw).split()
        cnt_norebuild = " ".join(f for f in cnt if f not in flags)
        return cnt_norebuild
    SCons.Action.ActionBase.get_contents = types.MethodType(get_contents, None, SCons.Action.ActionBase)

avoidRebuilds(noRebuild)

# --- go! go! go! ---

makeTargets(makeBaseEnvironment(makeOptions()))

# vim:set ft=python:
