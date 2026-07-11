# Independent Rollback Validation

**Date**: 2026-07-11

The implementation commit is `b3acfd1be528093a784ec9fe1f8d19b5bd0f884b`.
It was checked out into detached worktree `/tmp/ndnsf-spec086-rollback` and
validated with:

```bash
./waf configure --with-tests --with-examples
./waf build --targets=App_User,App_Provider,unit-tests -j$(nproc)
git revert --no-edit b3acfd1
./waf configure --with-tests --with-examples
./waf build --targets=App_User,App_Provider,unit-tests -j$(nproc)
```

Results:

- implementation checkout: PASS, 73 build steps;
- independent revert commit: `c41c435`;
- reverted checkout: PASS, 76 build steps, including restored V1/BloomFilter
  targets;
- detached worktree status: clean after the revert commit.

The temporary worktree was removed after validation. Build logs were retained
for this run at `/tmp/spec086-rollback-build-current.log` and
`/tmp/spec086-rollback-build-reverted.log`.
