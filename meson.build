# Copyright 2022 David Robillard <d@drobilla.net>
# SPDX-License-Identifier: 0BSD OR MIT

project('soundio', ['c'],
        version: '2.0.0',
        default_options: [
          'b_ndebug=if-release',
          'buildtype=release',
          'warning_level=3',
          'c_std=c99',
        ])

soundio_major_version = meson.project_version().split('.')[0]
soundio_minor_version = meson.project_version().split('.')[1]
soundio_micro_version = meson.project_version().split('.')[2]
soundio_src_root = meson.current_source_dir()
version_suffix = '-@0@'.format(soundio_major_version)
versioned_name = 'soundio' + version_suffix

#######################
# Compilers and Flags #
#######################

# Required tools
pkg = import('pkgconfig')
cc = meson.get_compiler('c')

# Build as C++ with MSVC
if cc.get_id() == 'msvc'
  add_project_arguments(['/TP'], language: 'c')
endif

# Warning suppressions
warning_level = get_option('warning_level')
c_suppressions = []
if cc.get_id() == 'clang'
  if warning_level == 'everything'
    c_suppressions += [
      '-Wno-alloca',
      '-Wno-bad-function-cast',
      '-Wno-cast-align',
      '-Wno-declaration-after-statement',
      '-Wno-double-promotion',
      '-Wno-float-conversion',
      '-Wno-float-equal',
      '-Wno-implicit-float-conversion',
      '-Wno-implicit-int-float-conversion',
      '-Wno-missing-noreturn',
      '-Wno-missing-variable-declarations',
      '-Wno-padded',
      '-Wno-shadow',
      '-Wno-shorten-64-to-32',
      '-Wno-sign-conversion',
      '-Wno-switch-enum',
      '-Wno-unreachable-code',
      '-Wno-used-but-marked-unused',
    ]
  endif

  if not meson.is_cross_build()
    c_suppressions += [
      '-Wno-poison-system-directories',
    ]
  endif

  if warning_level in ['everything', '3', '2']
    c_suppressions += [
      '-Wno-sign-compare',
      '-Wno-unused-parameter',
    ]
  endif

  if host_machine.system() == 'darwin'
    c_suppressions += [
      '-Wno-conditional-uninitialized',
    ]
  endif

elif cc.get_id() == 'gcc'
  if warning_level == 'everything'
    c_suppressions += [
      '-Wno-alloca',
      '-Wno-bad-function-cast',
      '-Wno-cast-align',
      '-Wno-conversion',
      '-Wno-double-promotion',
      '-Wno-float-conversion',
      '-Wno-float-equal',
      '-Wno-padded',
      '-Wno-shadow',
      '-Wno-sign-conversion',
      '-Wno-stack-protector',
      '-Wno-strict-overflow',
      '-Wno-switch-default',
      '-Wno-switch-enum',
      '-Wno-unsuffixed-float-constants',
      '-Wno-unused-const-variable',
    ]
  endif

  if warning_level in ['everything', '3', '2']
    c_suppressions += [
      '-Wno-sign-compare',
      '-Wno-unused-parameter',
    ]
  endif

elif cc.get_id() == 'msvc'
  c_suppressions += [
    '/experimental:external',
    '/external:W0',
    '/external:anglebrackets',
  ]

  if warning_level == 'everything'
    c_suppressions += [
      '/wd4061', # enumerator is not explicitly handled
      '/wd4242', # conversion with possible loss of data
      '/wd4365', # signed/unsigned mismatch
      '/wd4514', # unreferenced inline function has been removed
      '/wd4577', # noexcept used with no exception handling mode specified
      '/wd4623', # default constructor was implicitly deleted
      '/wd4625', # copy constructor was implicitly deleted
      '/wd4626', # assignment operator was implicitly deleted
      '/wd4800', # implicit conversion from int to bool
      '/wd4820', # padding added after data member
      '/wd5026', # move constructor was implicitly deleted
      '/wd5027', # move assignment operator was implicitly deleted
      '/wd5045', # will insert Spectre mitigation
      '/wd5219', # implicit conversion with possible loss of data
      '/wd5246', # initialization of subobject should be wrapped in braces
    ]
  endif

  if warning_level in ['everything', '3']
    c_suppressions += [
      '/wd4100', # unreferenced formal parameter
      '/wd4211', # nonstandard extension used: redefined extern to static
      '/wd4456', # declaration hides previous local declaration
      '/wd4702', # unreachable code
      '/wd4706', # assignment within conditional expression
    ]
  endif

  if warning_level in ['everything', '3', '2']
    c_suppressions += [
      '/wd4267', # conversion from 'size_t', possible loss of data
      '/wd4996', # POSIX name for this item is deprecated
    ]
  endif

  if warning_level in ['everything', '3', '2', '1']
    c_suppressions += [
      '/wd4244', # float to int conversion, possible loss of data
    ]
  endif
endif

add_project_arguments(cc.get_supported_arguments(c_suppressions), language: 'c')

##########################
# Platform Configuration #
##########################

platform_args = []
if host_machine.system() in ['gnu', 'linux']
  platform_args += [
    '-D_POSIX_C_SOURCE=200809L',
  ]
endif

# add_project_arguments(platform_args, language: 'c')

################
# Dependencies #
################

threads_dep = dependency('threads')
m_dep = cc.find_library('m', required: false)

alsa_dep = dependency('alsa', required: get_option('alsa'))
coreaudio_dep = dependency('coreaudio', required: get_option('coreaudio'))
corefoundation_dep = dependency('corefoundation', required: get_option('coreaudio'))
audiounit_dep = dependency('audiounit', required: get_option('coreaudio'))
jack_dep = dependency('jack', required: get_option('jack'))
pulseaudio_dep = dependency('pulseaudio', required: get_option('pulseaudio'))
have_wasapi = cc.has_header('audioclient.h')

###########
# Library #
###########

private_include_dirs = include_directories('.', 'src')

# Generate config.h
conf_data = configuration_data()
conf_data.set('SOUNDIO_HAVE_ALSA', alsa_dep.found())
conf_data.set('SOUNDIO_HAVE_COREAUDIO', coreaudio_dep.found())
conf_data.set('SOUNDIO_HAVE_JACK', jack_dep.found())
conf_data.set('SOUNDIO_HAVE_PULSEAUDIO', pulseaudio_dep.found())
conf_data.set('SOUNDIO_HAVE_WASAPI', have_wasapi)
conf_data.set('SOUNDIO_VERSION_MAJOR', soundio_major_version)
conf_data.set('SOUNDIO_VERSION_MINOR', soundio_minor_version)
conf_data.set('SOUNDIO_VERSION_PATCH', soundio_micro_version)
conf_data.set_quoted('SOUNDIO_VERSION_STRING', meson.project_version())
config_h = configure_file(output: 'config.h', configuration: conf_data)

libsoundio_sources = files(
  'src/channel_layout.c',
  'src/dummy.c',
  'src/os.c',
  'src/ring_buffer.c',
  'src/soundio.c',
  'src/util.c',
)

libsoundio_headers = files('soundio/soundio.h', 'soundio/endian.h')

if jack_dep.found()
  libsoundio_sources += files('src/jack.c')
endif
if pulseaudio_dep.found()
  libsoundio_sources += files('src/pulseaudio.c')
endif
if alsa_dep.found()
  libsoundio_sources += files('src/alsa.c')
endif
if coreaudio_dep.found()
  libsoundio_sources += files('src/coreaudio.c')
endif
if have_wasapi
  libsoundio_sources += files('src/wasapi.c')
endif

libsoundio_deps = [
  alsa_dep,
  audiounit_dep,
  coreaudio_dep,
  corefoundation_dep,
  jack_dep,
  m_dep,
  pulseaudio_dep,
  threads_dep,
]

# Build shared and/or static library
libsoundio = library(
  meson.project_name(),
  libsoundio_sources,
  c_args: platform_args + [
    '-DSOUNDIO_MAJOR_VERSION=@0@'.format(soundio_major_version),
    '-DSOUNDIO_MICRO_VERSION=@0@'.format(soundio_micro_version),
    '-DSOUNDIO_MINOR_VERSION=@0@'.format(soundio_minor_version),
    '-DSOUNDIO_VERSION="@0@"'.format(meson.project_version()),
  ],
  dependencies: libsoundio_deps,
  gnu_symbol_visibility: 'hidden',
  include_directories: private_include_dirs,
  install: true,
  version: meson.project_version(),
)

# Declare dependency for internal meson dependants
soundio_dep = declare_dependency(
  include_directories: include_directories('.'),
  link_with: libsoundio,
  dependencies: libsoundio_deps,
)

# Generage pkg-config file for external dependants
pkg.generate(
  libsoundio,
  description: 'C library providing cross-platform audio input and output',
  name: 'soundio',
  version: meson.project_version(),
)

# Override pkg-config dependency for internal meson dependants
meson.override_dependency(versioned_name, soundio_dep)

# Install headers
install_headers(libsoundio_headers, subdir: 'soundio')

############
# Examples #
############

if not get_option('examples').disabled()
  executable(
    'sio_sine',
    'example/sio_sine.c',
    dependencies: [soundio_dep, m_dep],
  )

  executable(
    'sio_list_devices',
    'example/sio_list_devices.c',
    dependencies: [soundio_dep, m_dep],
  )

  executable(
    'sio_microphone',
    'example/sio_microphone.c',
    dependencies: [soundio_dep],
  )

  executable(
    'sio_record',
    'example/sio_record.c',
    dependencies: [soundio_dep],
  )
endif

#########
# Tests #
#########

if not get_option('tests').disabled()
  test(
    'unit_tests',
    executable(
      'unit_tests',
      files('test/unit_tests.c'),
      libsoundio_sources,
      c_args: platform_args,
      dependencies: [soundio_dep, m_dep],
      include_directories: private_include_dirs,
    ),
  )

  executable(
    'latency',
    files('test/latency.c'),
    libsoundio_sources,
    c_args: platform_args,
    dependencies: [soundio_dep, m_dep],
    include_directories: private_include_dirs,
  )

  executable(
    'underflow',
    'test/underflow.c',
    dependencies: [soundio_dep, m_dep],
    include_directories: private_include_dirs,
  )

  executable(
    'backend_disconnect_recover',
    'test/backend_disconnect_recover.c',
    dependencies: [soundio_dep],
    include_directories: private_include_dirs,
  )

  executable(
    'overflow',
    'test/overflow.c',
    dependencies: [soundio_dep],
    include_directories: private_include_dirs,
  )
endif

#################
# Documentation #
#################

doxygen = find_program('doxygen', required: get_option('docs'))
if doxygen.found()
  subdir('doc')
endif

#########################
# Configuration Summary #
#########################

if not meson.is_subproject()
  summary(
    'Documentation',
    doxygen.found(),
    bool_yn: true,
    section: 'Components',
  )
  summary(
    'Examples',
    not get_option('tests').disabled(),
    bool_yn: true,
    section: 'Components',
  )
  summary(
    'Tests',
    not get_option('tests').disabled(),
    bool_yn: true,
    section: 'Components',
  )

  summary(
    'ALSA',
    alsa_dep.found(),
    bool_yn: true,
    section: 'Backends',
  )
  summary(
    'CoreAudio',
    coreaudio_dep.found(),
    bool_yn: true,
    section: 'Backends',
  )
  summary(
    'JACK',
    jack_dep.found(),
    bool_yn: true,
    section: 'Backends',
  )
  summary(
    'PulseAudio',
    pulseaudio_dep.found(),
    bool_yn: true,
    section: 'Backends',
  )
  summary(
    'WASAPI',
    have_wasapi,
    bool_yn: true,
    section: 'Backends',
  )

  summary(
    'Prefix',
    get_option('prefix'),
    section: 'Installation',
  )
  summary(
    'Headers',
    get_option('prefix') / get_option('includedir'),
    section: 'Installation',
  )
  summary(
    'Libraries',
    get_option('prefix') / get_option('libdir'),
    section: 'Installation',
  )
endif
