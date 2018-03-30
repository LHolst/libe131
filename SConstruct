#!python

import os, subprocess, platform

def add_sources(sources, dir, extension):
  for f in os.listdir(dir):
      if f.endswith('.' + extension):
          sources.append(dir + '/' + f)

AddOption('--prefix',
          dest='prefix',
          type='string',
          nargs=1,
          action='store',
          metavar='DIR',
          help='installation prefix')

version = "1.3.1"
host_platform = platform.system()
target_platform = ARGUMENTS.get('p', ARGUMENTS.get('platform', 'linux'))
target_arch = ARGUMENTS.get('a', ARGUMENTS.get('arch', '64'))
result_path = 'build'
result_name = ARGUMENTS.get('n', ARGUMENTS.get('name', os.path.relpath('.', '..')))
env = Environment(SHLIBVERSION=version, PREFIX=GetOption('prefix'))

if target_platform == 'linux':
    #result_name += '.linux.' + target_arch

    env['CXX']='gcc-5'
    if ARGUMENTS.get('use_llvm', 'no') == 'yes':
        env['CXX'] = 'clang++'

    env.Append(CCFLAGS = [ '-fPIC', '-g', '-O3', '-std=c++14', '-Wwrite-strings' ])
    env.Append(LINKFLAGS = [ '-Wl,-R,\'$$ORIGIN\'' ])

    if target_arch == '32':
        env.Append(CCFLAGS = [ '-m32' ])
        env.Append(LINKFLAGS = [ '-m32' ])
    elif target_arch == '64':
        env.Append(CCFLAGS = [ '-m64' ])
        env.Append(LINKFLAGS = [ '-m64' ])

elif target_platform == 'windows':
    # This makes sure to keep the session environment variables on windows,
    # that way you can run scons in a vs 2017 prompt and it will find all the required tools
    if (target_arch == '64'):
        env = Environment(ENV = os.environ, TARGET_ARCH='amd64')
    else:
        env = Environment(ENV = os.environ, TARGET_ARCH='x86')

    #result_name += '.windows.' + target_arch

    if host_platform == 'Windows':
        result_name += '.lib'

        env.Append(LINKFLAGS = [ '/WX' ])
        if target == 'debug':
            env.Append(CCFLAGS = ['/EHsc', '/D_DEBUG', '/MDd' ])
        else:
            env.Append(CCFLAGS = ['/O2', '/EHsc', '/DNDEBUG', '/MD' ])
    else:
        if target_arch == '32':
            env['CXX']='i686-w64-mingw32-g++'
        elif target_arch == '64':
            env['CXX']='x86_64-w64-mingw32-g++'

        env.Append(CCFLAGS = [ '-g', '-O3', '-std=c++14', '-Wwrite-strings' ])
        env.Append(LINKFLAGS = [ '--static', '-Wl,--no-undefined', '-static-libgcc', '-static-libstdc++' ])

elif target_platform == 'osx':
    if ARGUMENTS.get('use_llvm', 'no') == 'yes':
        env['CXX'] = 'clang++'

    env.Append(CCFLAGS = [ '-g','-O3', '-std=c++14', '-arch', 'x86_64' ])
    env.Append(LINKFLAGS = [ '-arch', 'x86_64', '-framework', 'Cocoa', '-Wl,-undefined,dynamic_lookup' ])

sources = []
add_sources(sources, 'src', 'c')

idir_prefix = '$PREFIX'
idir_lib    = '$PREFIX/lib'
idir_bin    = '$PREFIX/bin'
idir_inc    = '$PREFIX/include'
idir_data   = '$PREFIX/share'

library = env.SharedLibrary(target=result_path + '/' + result_name, source=sources)
ilib =  env.InstallVersionedLib(idir_lib, library)
ihead = env.Install(idir_inc, ['src/e131.h'])

env.Alias('install', [ilib, ihead])
env.Command("uninstall", None, Delete(FindInstalledFiles()))
Default(library)