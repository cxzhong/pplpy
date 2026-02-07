project('ppl', 'cpp',
  version: '1.2',
  default_options: ['cpp_std=c++23'],
)

cpp = meson.get_compiler('cpp')

# Find GMP / GMPXX
gmp_dep = dependency('gmp', required: false)
if not gmp_dep.found()
  gmp_dep = cpp.find_library('gmp')
endif
gmpxx_dep = dependency('gmpxx', required: false)
if not gmpxx_dep.found()
  gmpxx_dep = cpp.find_library('gmpxx')
endif

# ---------- sizes (used by ppl-config.h) ----------
sz_char       = cpp.sizeof('char')
sz_short      = cpp.sizeof('short')
sz_int        = cpp.sizeof('int')
sz_long       = cpp.sizeof('long')
sz_long_long  = cpp.sizeof('long long')
sz_float      = cpp.sizeof('float')
sz_double     = cpp.sizeof('double')
sz_long_double = cpp.sizeof('long double')
sz_size_t     = cpp.sizeof('size_t')
sz_intp       = cpp.sizeof('int*')
sz_mp_limb_t  = cpp.sizeof('mp_limb_t', prefix: '#include <gmp.h>', dependencies: gmp_dep)
# fp = function pointer
sz_fp         = cpp.sizeof('void (*)(void)')

# ---------- header checks ----------
have_stdint_h      = cpp.has_header('stdint.h')
have_inttypes_h    = cpp.has_header('inttypes.h')
have_dlfcn_h       = cpp.has_header('dlfcn.h')
have_fenv_h        = cpp.has_header('fenv.h')
have_getopt_h      = cpp.has_header('getopt.h')
have_ieeefp_h      = cpp.has_header('ieeefp.h')
have_memory_h      = cpp.has_header('memory.h')
have_signal_h      = cpp.has_header('signal.h')
have_stdlib_h      = cpp.has_header('stdlib.h')
have_string_h      = cpp.has_header('string.h')
have_strings_h     = cpp.has_header('strings.h')
have_unistd_h      = cpp.has_header('unistd.h')
have_sys_resource_h = cpp.has_header('sys/resource.h')
have_sys_stat_h    = cpp.has_header('sys/stat.h')
have_sys_time_h    = cpp.has_header('sys/time.h')
have_sys_types_h   = cpp.has_header('sys/types.h')
have_glpk_h        = cpp.has_header('glpk.h')
have_glpk_glpk_h   = cpp.has_header('glpk/glpk.h')

# ---------- type checks ----------
have_int_fast16_t  = cpp.has_type('int_fast16_t',  prefix: '#include <stdint.h>')
have_int_fast32_t  = cpp.has_type('int_fast32_t',  prefix: '#include <stdint.h>')
have_int_fast64_t  = cpp.has_type('int_fast64_t',  prefix: '#include <stdint.h>')
have_uint_fast16_t = cpp.has_type('uint_fast16_t', prefix: '#include <stdint.h>')
have_uint_fast32_t = cpp.has_type('uint_fast32_t', prefix: '#include <stdint.h>')
have_uint_fast64_t = cpp.has_type('uint_fast64_t', prefix: '#include <stdint.h>')
have_uintptr_t     = cpp.has_type('uintptr_t',     prefix: '#include <stdint.h>')

# siginfo_t & timeval
have_siginfo_t = cpp.has_type('siginfo_t', prefix: '#include <signal.h>')
have_timeval   = cpp.has_type('struct timeval', prefix: '#include <sys/time.h>')

# ---------- function / symbol declarations ----------
have_decl_ffs      = cpp.has_function('ffs',      prefix: '#include <strings.h>')
have_decl_fma      = cpp.has_function('fma',      prefix: '#include <cmath>')
have_decl_fmaf     = cpp.has_function('fmaf',     prefix: '#include <cmath>')
have_decl_fmal     = cpp.has_function('fmal',     prefix: '#include <cmath>')
have_decl_getenv   = cpp.has_function('getenv',   prefix: '#include <stdlib.h>')
have_decl_rintf    = cpp.has_function('rintf',    prefix: '#include <cmath>')
have_decl_rintl    = cpp.has_function('rintl',    prefix: '#include <cmath>')
have_decl_strtod   = cpp.has_function('strtod',   prefix: '#include <stdlib.h>')
have_decl_strtof   = cpp.has_function('strtof',   prefix: '#include <stdlib.h>')
have_decl_strtold  = cpp.has_function('strtold',  prefix: '#include <stdlib.h>')
have_decl_strtoll  = cpp.has_function('strtoll',  prefix: '#include <stdlib.h>')
have_decl_strtoull = cpp.has_function('strtoull',  prefix: '#include <stdlib.h>')

have_decl_getrusage = cpp.has_header_symbol('sys/resource.h', 'getrusage')
have_decl_setrlimit = cpp.has_header_symbol('sys/resource.h', 'setrlimit')
have_decl_setitimer = cpp.has_header_symbol('sys/time.h', 'setitimer')
have_decl_sigaction = cpp.has_header_symbol('signal.h', 'sigaction')

# RLIMIT defines
_rlimit_prefix = '#include <sys/resource.h>'
have_decl_rlimit_as   = cpp.has_header_symbol('sys/resource.h', 'RLIMIT_AS')
have_decl_rlimit_data = cpp.has_header_symbol('sys/resource.h', 'RLIMIT_DATA')
have_decl_rlimit_rss  = cpp.has_header_symbol('sys/resource.h', 'RLIMIT_RSS')
have_decl_rlimit_vmem = cpp.has_header_symbol('sys/resource.h', 'RLIMIT_VMEM')

# Check for __mpz_struct members
_mpz_prefix = '#include <gmp.h>'
have_mpz_alloc = cpp.has_member('__mpz_struct', '_mp_alloc', prefix: _mpz_prefix, dependencies: gmp_dep)
have_mpz_d     = cpp.has_member('__mpz_struct', '_mp_d',     prefix: _mpz_prefix, dependencies: gmp_dep)
have_mpz_size  = cpp.has_member('__mpz_struct', '_mp_size',  prefix: _mpz_prefix, dependencies: gmp_dep)

# typeof support
have_typeof = cpp.compiles('int x = 0; typeof(x) y = x;', name: 'typeof support')

# char signedness
cxx_plain_char_is_signed = cpp.compiles('''
  static_assert(static_cast<char>(0xFF) < 0, "char is unsigned");
''', name: 'plain char is signed')

# FPU control
can_control_fpu = cpp.compiles('''
  #include <fenv.h>
  int main() { fesetround(FE_UPWARD); return 0; }
''', name: 'can control FPU')

# limiting memory
cxx_supports_limiting_memory = cpp.links('''
  #include <sys/resource.h>
  int main() {
    struct rlimit rl;
    getrlimit(RLIMIT_AS, &rl);
    setrlimit(RLIMIT_AS, &rl);
    return 0;
  }
''', name: 'supports limiting memory')

# IEEE inexact flag
cxx_supports_ieee_inexact_flag = cpp.compiles('''
  #include <fenv.h>
  int main() {
    feclearexcept(FE_INEXACT);
    return fetestexcept(FE_INEXACT);
  }
''', name: 'IEEE inexact flag support')

# __attribute__((weak))
cxx_supports_attribute_weak = cpp.compiles('''
  void __attribute__((weak)) f() {}
''', name: '__attribute__((weak)) support')

# zero length arrays
cxx_supports_zero_length_arrays = cpp.compiles('''
  struct S { int a[0]; };
''', name: 'zero length arrays')

# proper long double
cxx_provides_proper_long_double = cpp.compiles('''
  static_assert(sizeof(long double) > sizeof(double), "no proper long double");
''', name: 'proper long double')

# Float binary formats – on x86_64 Linux these are the standard IEEE formats
# We detect at configure time whether doubles/floats/long doubles are supported
# by checking their size matches known IEEE formats.
float_format_map = {
  '4': 'PPL_FLOAT_IEEE754_SINGLE',
  '8': 'PPL_FLOAT_IEEE754_DOUBLE',
  '16': 'PPL_FLOAT_IEEE754_QUAD',
}

cxx_float_format = float_format_map.get(sz_float.to_string(), '')
cxx_double_format = float_format_map.get(sz_double.to_string(), '')
# long double: 16 bytes on x86_64 but actually 80-bit Intel extended
cxx_long_double_format = ''
if sz_long_double == 16 or sz_long_double == 12
  cxx_long_double_format = 'PPL_FLOAT_INTEL_DOUBLE_EXTENDED'
endif

supported_float = cxx_float_format != '' ? 1 : 0
supported_double = cxx_double_format != '' ? 1 : 0
supported_long_double = cxx_long_double_format != '' ? 1 : 0

# ---------- Build configuration data ----------
cdata = configuration_data()

cdata.set('PPL_HAVE_CXX11', true)

cdata.set('PPL_HAVE_DECL_FFS',      have_decl_ffs ? 1 : 0)
cdata.set('PPL_HAVE_DECL_FMA',      have_decl_fma ? 1 : 0)
cdata.set('PPL_HAVE_DECL_FMAF',     have_decl_fmaf ? 1 : 0)
cdata.set('PPL_HAVE_DECL_FMAL',     have_decl_fmal ? 1 : 0)
cdata.set('PPL_HAVE_DECL_GETENV',   have_decl_getenv ? 1 : 0)
cdata.set('PPL_HAVE_DECL_GETRUSAGE', have_decl_getrusage ? 1 : 0)
cdata.set('PPL_HAVE_DECL_RINTF',    have_decl_rintf ? 1 : 0)
cdata.set('PPL_HAVE_DECL_RINTL',    have_decl_rintl ? 1 : 0)
cdata.set('PPL_HAVE_DECL_RLIMIT_AS',   have_decl_rlimit_as ? 1 : 0)
cdata.set('PPL_HAVE_DECL_RLIMIT_DATA', have_decl_rlimit_data ? 1 : 0)
cdata.set('PPL_HAVE_DECL_RLIMIT_RSS',  have_decl_rlimit_rss ? 1 : 0)
cdata.set('PPL_HAVE_DECL_RLIMIT_VMEM', have_decl_rlimit_vmem ? 1 : 0)
cdata.set('PPL_HAVE_DECL_SETITIMER', have_decl_setitimer ? 1 : 0)
cdata.set('PPL_HAVE_DECL_SETRLIMIT', have_decl_setrlimit ? 1 : 0)
cdata.set('PPL_HAVE_DECL_SIGACTION', have_decl_sigaction ? 1 : 0)
cdata.set('PPL_HAVE_DECL_STRTOD',   have_decl_strtod ? 1 : 0)
cdata.set('PPL_HAVE_DECL_STRTOF',   have_decl_strtof ? 1 : 0)
cdata.set('PPL_HAVE_DECL_STRTOLD',  have_decl_strtold ? 1 : 0)
cdata.set('PPL_HAVE_DECL_STRTOLL',  have_decl_strtoll ? 1 : 0)
cdata.set('PPL_HAVE_DECL_STRTOULL', have_decl_strtoull ? 1 : 0)

cdata.set('PPL_HAVE_DLFCN_H',       have_dlfcn_h)
cdata.set('PPL_HAVE_FENV_H',        have_fenv_h)
cdata.set('PPL_HAVE_GETOPT_H',      have_getopt_h)
cdata.set('PPL_HAVE_GLPK_GLPK_H',   have_glpk_glpk_h)
cdata.set('PPL_HAVE_GLPK_H',        have_glpk_h)
cdata.set('PPL_HAVE_IEEEFP_H',      have_ieeefp_h)
cdata.set('PPL_HAVE_INTTYPES_H',    have_inttypes_h)
cdata.set('PPL_HAVE_INT_FAST16_T',  have_int_fast16_t)
cdata.set('PPL_HAVE_INT_FAST32_T',  have_int_fast32_t)
cdata.set('PPL_HAVE_INT_FAST64_T',  have_int_fast64_t)
cdata.set('PPL_HAVE_MEMORY_H',      have_memory_h)
cdata.set('PPL_HAVE_SIGINFO_T',     have_siginfo_t)
cdata.set('PPL_HAVE_SIGNAL_H',      have_signal_h)
cdata.set('PPL_HAVE_STDINT_H',      have_stdint_h)
cdata.set('PPL_HAVE_STDLIB_H',      have_stdlib_h)
cdata.set('PPL_HAVE_STRINGS_H',     have_strings_h)
cdata.set('PPL_HAVE_STRING_H',      have_string_h)
cdata.set('PPL_HAVE_SYS_RESOURCE_H', have_sys_resource_h)
cdata.set('PPL_HAVE_SYS_STAT_H',    have_sys_stat_h)
cdata.set('PPL_HAVE_SYS_TIME_H',    have_sys_time_h)
cdata.set('PPL_HAVE_SYS_TYPES_H',   have_sys_types_h)
cdata.set('PPL_HAVE_TIMEVAL',        have_timeval)
cdata.set('PPL_HAVE_TYPEOF',         have_typeof)
cdata.set('PPL_HAVE_UINTPTR_T',      have_uintptr_t)
cdata.set('PPL_HAVE_UINT_FAST16_T',  have_uint_fast16_t)
cdata.set('PPL_HAVE_UINT_FAST32_T',  have_uint_fast32_t)
cdata.set('PPL_HAVE_UINT_FAST64_T',  have_uint_fast64_t)
cdata.set('PPL_HAVE_UNISTD_H',       have_unistd_h)
cdata.set('PPL_HAVE___MPZ_STRUCT__MP_ALLOC', have_mpz_alloc)
cdata.set('PPL_HAVE___MPZ_STRUCT__MP_D',     have_mpz_d)
cdata.set('PPL_HAVE___MPZ_STRUCT__MP_SIZE',  have_mpz_size)

cdata.set_quoted('PPL_PACKAGE_BUGREPORT', 'bagnara@cs.unipr.it')
cdata.set_quoted('PPL_PACKAGE_NAME',      'the Parma Polyhedra Library')
cdata.set_quoted('PPL_PACKAGE_STRING',    'the Parma Polyhedra Library 1.2')
cdata.set_quoted('PPL_PACKAGE_TARNAME',   'ppl')
cdata.set_quoted('PPL_PACKAGE_VERSION',   '1.2')

cdata.set('PPL_CAN_CONTROL_FPU',    can_control_fpu ? 1 : 0)
cdata.set('PPL_COEFFICIENT_BITS',   0)
# PPL_COEFFICIENT_TYPE must be bare token mpz_class (not a quoted string)
# We handle this with a #define directly in the template.
cdata.set_quoted('PPL_CONFIGURE_OPTIONS', '')

# Float format macros: these must expand to the PPL_FLOAT_* constants (integers)
if cxx_float_format != ''
  if cxx_float_format == 'PPL_FLOAT_IEEE754_SINGLE'
    cdata.set('PPL_CXX_FLOAT_BINARY_FORMAT', 1)
  elif cxx_float_format == 'PPL_FLOAT_IEEE754_DOUBLE'
    cdata.set('PPL_CXX_FLOAT_BINARY_FORMAT', 2)
  endif
endif
if cxx_double_format != ''
  if cxx_double_format == 'PPL_FLOAT_IEEE754_SINGLE'
    cdata.set('PPL_CXX_DOUBLE_BINARY_FORMAT', 1)
  elif cxx_double_format == 'PPL_FLOAT_IEEE754_DOUBLE'
    cdata.set('PPL_CXX_DOUBLE_BINARY_FORMAT', 2)
  endif
endif
if cxx_long_double_format != ''
  cdata.set('PPL_CXX_LONG_DOUBLE_BINARY_FORMAT', 4)
endif

cdata.set('PPL_CXX_PLAIN_CHAR_IS_SIGNED', cxx_plain_char_is_signed ? 1 : 0)
cdata.set('PPL_CXX_PROVIDES_PROPER_LONG_DOUBLE', cxx_provides_proper_long_double ? 1 : 0)
cdata.set('PPL_CXX_SUPPORTS_ATTRIBUTE_WEAK', cxx_supports_attribute_weak ? 1 : 0)
cdata.set('PPL_CXX_SUPPORTS_IEEE_INEXACT_FLAG', cxx_supports_ieee_inexact_flag ? 1 : 0)
cdata.set('PPL_CXX_SUPPORTS_LIMITING_MEMORY', cxx_supports_limiting_memory ? 1 : 0)
cdata.set('PPL_CXX_SUPPORTS_ZERO_LENGTH_ARRAYS', cxx_supports_zero_length_arrays ? 1 : 0)

cdata.set('PPL_GMP_INTEGERS', true)
cdata.set('PPL_NDEBUG', true)

cdata.set('PPL_SUPPORTED_DOUBLE', supported_double)
cdata.set('PPL_SUPPORTED_FLOAT',  supported_float)
cdata.set('PPL_SUPPORTED_LONG_DOUBLE', supported_long_double)

cdata.set('PPL_SIZEOF_CHAR',        sz_char)
cdata.set('PPL_SIZEOF_DOUBLE',      sz_double)
cdata.set('PPL_SIZEOF_FLOAT',       sz_float)
cdata.set('PPL_SIZEOF_FP',          sz_fp)
cdata.set('PPL_SIZEOF_INT',         sz_int)
cdata.set('PPL_SIZEOF_INTP',        sz_intp)
cdata.set('PPL_SIZEOF_LONG',        sz_long)
cdata.set('PPL_SIZEOF_LONG_DOUBLE', sz_long_double)
cdata.set('PPL_SIZEOF_LONG_LONG',   sz_long_long)
cdata.set('PPL_SIZEOF_MP_LIMB_T',   sz_mp_limb_t)
cdata.set('PPL_SIZEOF_SHORT',       sz_short)
cdata.set('PPL_SIZEOF_SIZE_T',      sz_size_t)
cdata.set('PPL_STDC_HEADERS', true)

# Endianness
if host_machine.endian() == 'big'
  cdata.set('PPL_WORDS_BIGENDIAN', true)
endif

# Generate ppl-config.h
ppl_config_h = configure_file(
  input: 'ppl-config.h.mesondefine',
  output: 'ppl-config.h',
  configuration: cdata,
)

# Generate version.hh
vdata = configuration_data()
vdata.set('PPL_VERSION_MAJOR', 1)
vdata.set('PPL_VERSION_MINOR', 2)
vdata.set('PPL_VERSION_REVISION', 0)
vdata.set('PPL_VERSION_BETA', 0)
vdata.set('VERSION', '1.2')

version_hh = configure_file(
  input: 'src/version.hh.in',
  output: 'version.hh',
  configuration: vdata,
)

# ---------- PPL static library ----------

# Copy ppl.hh.dist to ppl.hh (this is the pre-generated all-in-one header)
ppl_hh = configure_file(
  input: 'src/ppl.hh.dist',
  output: 'ppl.hh',
  copy: true,
)

ppl_sources = files(
  'src/assertions.cc',
  'src/Box.cc',
  'src/checked.cc',
  'src/Checked_Number.cc',
  'src/Float.cc',
  'src/fpu-ia32.cc',
  'src/BDS_Status.cc',
  'src/Box_Status.cc',
  'src/Og_Status.cc',
  'src/Concrete_Expression.cc',
  'src/Constraint.cc',
  'src/Constraint_System.cc',
  'src/Congruence.cc',
  'src/Congruence_System.cc',
  'src/Generator_System.cc',
  'src/Grid_Generator_System.cc',
  'src/Generator.cc',
  'src/Grid_Generator.cc',
  'src/Handler.cc',
  'src/Init.cc',
  'src/Coefficient.cc',
  'src/Linear_Expression.cc',
  'src/Linear_Expression_Impl.cc',
  'src/Linear_Expression_Interface.cc',
  'src/Linear_Form.cc',
  'src/Scalar_Products.cc',
  'src/MIP_Problem.cc',
  'src/PIP_Tree.cc',
  'src/PIP_Problem.cc',
  'src/Poly_Con_Relation.cc',
  'src/Poly_Gen_Relation.cc',
  'src/BHRZ03_Certificate.cc',
  'src/H79_Certificate.cc',
  'src/Grid_Certificate.cc',
  'src/Partial_Function.cc',
  'src/Polyhedron_nonpublic.cc',
  'src/Polyhedron_public.cc',
  'src/Polyhedron_chdims.cc',
  'src/Polyhedron_widenings.cc',
  'src/C_Polyhedron.cc',
  'src/NNC_Polyhedron.cc',
  'src/Grid_nonpublic.cc',
  'src/Grid_public.cc',
  'src/Grid_chdims.cc',
  'src/Grid_widenings.cc',
  'src/BD_Shape.cc',
  'src/Octagonal_Shape.cc',
  'src/Pointset_Powerset.cc',
  'src/CO_Tree.cc',
  'src/Sparse_Row.cc',
  'src/Dense_Row.cc',
  'src/Bit_Matrix.cc',
  'src/Bit_Row.cc',
  'src/Ph_Status.cc',
  'src/Grid_Status.cc',
  'src/Variable.cc',
  'src/Variables_Set.cc',
  'src/Grid_conversion.cc',
  'src/Grid_simplify.cc',
  'src/set_GMP_memory_alloc_funcs.cc',
  'src/stdiobuf.cc',
  'src/c_streambuf.cc',
  'src/globals.cc',
  'src/mp_std_bits.cc',
  'src/Weight_Profiler.cc',
  'src/version.cc',
  'src/termination.cc',
  'src/wrap_string.cc',
  'src/Time.cc',
  'src/Watchdog.cc',
  'src/Threshold_Watcher.cc',
)

ppl_config_inc = include_directories('.')   # for generated ppl-config.h / version.hh
ppl_src_inc    = include_directories('src') # for all the _defs.hh / _inlines.hh

ppl_lib = static_library('ppl', ppl_sources,
  include_directories: [ppl_config_inc, ppl_src_inc],
  dependencies: [gmp_dep, gmpxx_dep],
  cpp_args: ['-DNDEBUG'],
  pic: true,
)

# ---------- Declare dependency for consumers ----------
ppl_dep = declare_dependency(
  link_with: ppl_lib,
  include_directories: [ppl_config_inc, ppl_src_inc],
  dependencies: [gmp_dep, gmpxx_dep],
)
