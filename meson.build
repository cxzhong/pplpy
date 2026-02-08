project('pplpy', 'cpp', 'cython',
  version: '0.8.10',
  default_options: [
    'cpp_std=c++17',
  ],
)

# ---------- Python ----------
py = import('python').find_installation(pure: false)

# ---------- PPL ----------
# Try to find system PPL first, fall back to bundled subproject
ppl_dep = dependency('ppl', required: false)
if not ppl_dep.found()
  ppl_proj = subproject('ppl-1.2')
  ppl_dep = ppl_proj.get_variable('ppl_dep')
endif

# ---------- Build the ppl package ----------
subdir('ppl')

# ---------- Tests ----------
subdir('tests')
