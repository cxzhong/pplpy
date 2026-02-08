project('pplpy', 'cpp', 'cython',
  version: '0.8.10',
  default_options: [
    'cpp_std=c++17',
  ],
)

# ---------- Python ----------
py = import('python').find_installation(pure: false)

# ---------- PPL ----------
# Try to find system PPL via ppl-config script, fall back to bundled subproject
ppl_config = find_program('ppl-config', required: false)
if ppl_config.found()
  ppl_cppflags = run_command(ppl_config, '--cppflags', check: true).stdout().strip().split()
  ppl_ldflags = run_command(ppl_config, '--ldflags', check: true).stdout().strip().split()
  ppl_dep = declare_dependency(
    compile_args: ppl_cppflags,
    link_args: ppl_ldflags,
  )
  # System PPL 1.2 headers use std::mem_fun_ref, removed in C++17.
  # Compile consumer code with C++11 to stay compatible.
  ppl_cpp_std_override = ['cpp_std=c++11']
else
  ppl_proj = subproject('ppl-1.2')
  ppl_dep = ppl_proj.get_variable('ppl_dep')
  ppl_cpp_std_override = []
endif

# ---------- Build the ppl package ----------
subdir('ppl')

# ---------- Tests ----------
subdir('tests')
