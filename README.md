# Example Python package to demo mishandling extra names like `v0`

This repo contains a Python package that pip crashes when installing. It uses an
extra named `v0` which is mishandled by the `packaging` library used by pip when
it parses the requirement in the package metadata that uses an extra name that's
also a valid version

```
Requires-Dist: typing-extensions; extra == 'v0'
```

I've contributed a fix for this `packaging` bug in
https://github.com/pypa/packaging/pull/883

Pip crashes when installing this package:

```console
$ pip --version
pip 25.0.1 from /workspaces/test-env/lib/python3.13/site-packages/pip (python 3.13)
$ pip install 'v-extra-example@git+https://github.com/h4l/v-extra-package-issue.git@main'
Collecting v-extra-example@ git+https://github.com/h4l/v-extra-package-issue.git@main
  Cloning https://github.com/h4l/v-extra-package-issue.git (to revision main) to /tmp/pip-install-39_iyhp3/v-extra-example_84a0e79200514316a5eabe83450d607d
  Running command git clone --filter=blob:none --quiet https://github.com/h4l/v-extra-package-issue.git /tmp/pip-install-39_iyhp3/v-extra-example_84a0e79200514316a5eabe83450d607d
  Resolved https://github.com/h4l/v-extra-package-issue.git to commit b79cc5327a64c6f5cae1e486ff9b9ee74378ecc8
  Installing build dependencies ... done
  Getting requirements to build wheel ... done
  Preparing metadata (pyproject.toml) ... done
ERROR: Exception:
Traceback (most recent call last):
  File "/workspaces/test-env/lib/python3.13/site-packages/pip/_internal/cli/base_command.py", line 106, in _run_wrapper
    status = _inner_run()
  File "/workspaces/test-env/lib/python3.13/site-packages/pip/_internal/cli/base_command.py", line 97, in _inner_run
    return self.run(options, args)
           ~~~~~~~~^^^^^^^^^^^^^^^
  File "/workspaces/test-env/lib/python3.13/site-packages/pip/_internal/cli/req_command.py", line 67, in wrapper
    return func(self, options, args)
  File "/workspaces/test-env/lib/python3.13/site-packages/pip/_internal/commands/install.py", line 386, in run
    requirement_set = resolver.resolve(
        reqs, check_supported_wheels=not options.target_dir
    )
  File "/workspaces/test-env/lib/python3.13/site-packages/pip/_internal/resolution/resolvelib/resolver.py", line 95, in resolve
    result = self._result = resolver.resolve(
                            ~~~~~~~~~~~~~~~~^
        collected.requirements, max_rounds=limit_how_complex_resolution_can_be
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    )
    ^
  File "/workspaces/test-env/lib/python3.13/site-packages/pip/_vendor/resolvelib/resolvers.py", line 546, in resolve
    state = resolution.resolve(requirements, max_rounds=max_rounds)
  File "/workspaces/test-env/lib/python3.13/site-packages/pip/_vendor/resolvelib/resolvers.py", line 427, in resolve
    failure_causes = self._attempt_to_pin_criterion(name)
  File "/workspaces/test-env/lib/python3.13/site-packages/pip/_vendor/resolvelib/resolvers.py", line 239, in _attempt_to_pin_criterion
    criteria = self._get_updated_criteria(candidate)
  File "/workspaces/test-env/lib/python3.13/site-packages/pip/_vendor/resolvelib/resolvers.py", line 229, in _get_updated_criteria
    for requirement in self._p.get_dependencies(candidate=candidate):
                       ~~~~~~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^
  File "/workspaces/test-env/lib/python3.13/site-packages/pip/_internal/resolution/resolvelib/provider.py", line 247, in get_dependencies
    return [r for r in candidate.iter_dependencies(with_requires) if r is not None]
                       ~~~~~~~~~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^
  File "/workspaces/test-env/lib/python3.13/site-packages/pip/_internal/resolution/resolvelib/candidates.py", line 253, in iter_dependencies
    for r in requires:
             ^^^^^^^^
  File "/workspaces/test-env/lib/python3.13/site-packages/pip/_internal/metadata/importlib/_dists.py", line 225, in iter_dependencies
    elif not extras and req.marker.evaluate({"extra": ""}):
                        ~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^
  File "/workspaces/test-env/lib/python3.13/site-packages/pip/_vendor/packaging/markers.py", line 319, in evaluate
    return _evaluate_markers(
        self._markers, _repair_python_full_version(current_environment)
    )
  File "/workspaces/test-env/lib/python3.13/site-packages/pip/_vendor/packaging/markers.py", line 225, in _evaluate_markers
    groups[-1].append(_eval_op(lhs_value, op, rhs_value))
                      ~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/workspaces/test-env/lib/python3.13/site-packages/pip/_vendor/packaging/markers.py", line 183, in _eval_op
    return spec.contains(lhs, prereleases=True)
           ~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^
  File "/workspaces/test-env/lib/python3.13/site-packages/pip/_vendor/packaging/specifiers.py", line 552, in contains
    normalized_item = _coerce_version(item)
  File "/workspaces/test-env/lib/python3.13/site-packages/pip/_vendor/packaging/specifiers.py", line 28, in _coerce_version
    version = Version(version)
  File "/workspaces/test-env/lib/python3.13/site-packages/pip/_vendor/packaging/version.py", line 202, in __init__
    raise InvalidVersion(f"Invalid version: {version!r}")
pip._vendor.packaging.version.InvalidVersion: Invalid version: ''
```
