project('quickjs', 'c',
  version: '2026-06-04-frida',
  default_options: ['c_std=gnu99,c99'],
)

cc = meson.get_compiler('c')

have_msvc = cc.get_id() == 'msvc'
have_emscripten = cc.get_define('EMSCRIPTEN') != ''

if meson.is_subproject() and not have_msvc
  add_project_arguments('-w', language: 'c')
endif

threads_dep = dependency('threads')
dl_dep = cc.find_library('dl', required: false)
m_dep = cc.find_library('m', required: false)

headers = [
  'quickjs.h',
]

sources = [
  'quickjs.c',
  'dtoa.c',
  'libregexp.c',
  'libunicode.c',
  'cutils.c',
]

cdata = configuration_data()
cdata.set_quoted('CONFIG_VERSION', meson.project_version())

if cc.has_header('unistd.h')
  cdata.set('HAVE_UNISTD_H', 1)
endif

if cc.has_function('malloc_usable_size')
  cdata.set('HAVE_MALLOC_USABLE_SIZE', 1)
endif

if get_option('libc')
  headers += 'quickjs-libc.h'
  sources += 'quickjs-libc.c'
endif

atomics = false
opt = get_option('atomics')
if not opt.disabled()
  supported = not (have_msvc or have_emscripten)
  if opt.auto()
    atomics = supported
  elif supported
    atomics = true
  else
    error('Atomics support was requested but is not available')
  endif
endif
cdata.set('CONFIG_ATOMICS', atomics)

stack_check = false
opt = get_option('stack_check')
if not opt.disabled()
  supported = not (have_msvc or have_emscripten)
  if opt.auto()
    stack_check = supported
  elif supported
    stack_check = true
  else
    error('Stack limitation was requested but is not available')
  endif
endif
cdata.set('CONFIG_STACK_CHECK', stack_check)

install_headers(headers, subdir: 'quickjs')

configure_file(
  output: 'config.h',
  configuration: cdata,
)
add_project_arguments(
  have_msvc ? '/FI' : '-include', meson.current_build_dir() / 'config.h',
  '-D_GNU_SOURCE=1',
  language: 'c',
)

quickjs = library('quickjs', sources,
  c_args: have_msvc ? ['/fp:precise'] : [],
  dependencies: [threads_dep, dl_dep, m_dep],
  install: true,
)
quickjs_dep = declare_dependency(
  link_with: quickjs,
  include_directories: include_directories('.'),
)

pkg = import('pkgconfig')
pkg.generate(quickjs,
  name: 'quickjs',
  description: 'Small and embeddable JavaScript engine',
  url: 'https://bellard.org/quickjs/',
  subdirs: ['quickjs'],
)

meson.override_dependency('quickjs', quickjs_dep)
