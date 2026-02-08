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
  ppl_cxxflags = run_command(ppl_config, '--cxxflags', check: true).stdout().strip().split()
  ppl_ldflags = run_command(ppl_config, '--ldflags', check: true).stdout().strip().split()
  ppl_dep = declare_dependency(
    compile_args: ppl_cxxflags,
    link_args: ppl_ldflags,
  )
else
  ppl_proj = subproject('ppl-1.2')
  ppl_dep = ppl_proj.get_variable('ppl_dep')
endif

# ---------- Build the ppl package ----------
subdir('ppl')

# ---------- Tests ----------
subdir('tests')
