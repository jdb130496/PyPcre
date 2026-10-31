project('PyPcre', 'c',
  version: '0.5.0',
  default_options: ['c_std=c11']
)

message('pcre2_inc = ' + get_option('pcre2_include_dir'))
message('pcre2_lib = ' + get_option('pcre2_library'))

py = import('python').find_installation(pure: false)
dep_py = py.dependency()

pcre2_inc = get_option('pcre2_include_dir')
pcre2_lib = get_option('pcre2_library')

if pcre2_inc != '' and pcre2_lib != ''
  pcre2_dep = declare_dependency(
    include_directories: include_directories(pcre2_inc),
    link_args: [pcre2_lib]
  )
else
  pcre2_dep = dependency('libpcre2-8',
    required: true,
    method: 'auto'
  )
endif

sources = files(
  'pcre_ext/pcre2.c',
  'pcre_ext/pattern_cache.c',
  'pcre_ext/jit_support.c',
  'pcre_ext/string_helpers.c',
  'pcre_ext/error.c',
  'pcre_ext/cache.c',
  'pcre_ext/flag.c',
  'pcre_ext/util.c',
  'pcre_ext/memory.c',
)
py.extension_module('pcre_ext_c',
  sources,
  dependencies: [dep_py, pcre2_dep],
  c_args: ['-DPCRE2_STATIC'],
  install: true,
)
py.install_sources(
  'pcre/__init__.py',
  'pcre/_stdlib_re.py',
  'pcre/cache.py',
  'pcre/error.py',
  'pcre/flags.py',
  'pcre/pcre.py',
  'pcre/re_compat.py',
  'pcre/threads.py',
  subdir: 'pcre'
)
