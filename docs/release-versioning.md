# Release and Versioning

## Version format

We use semantic versioning: `vMAJOR.MINOR.PATCH`

- Bump **MAJOR** when you make a breaking change to the mapping format, config keys, or runtime behavior
- Bump **MINOR** for new features that don't break existing setups (new formatters, new transform types, new config options)
- Bump **PATCH** for bug fixes only

## How to cut a release

1. Make sure `main` is green (all tests pass)
2. Update the version in `pom.xml` if needed
3. Tag and push:
   ```bash
   git tag v1.2.3
   git push origin v1.2.3
   ```
4. Create a GitHub release from the tag and attach the shaded JAR

## Writing a changelog

We follow [Conventional Commits](https://www.conventionalcommits.org/) — prefix your commits with `feat:`, `fix:`, `docs:`, `perf:`, `refactor:`, etc.

To get a quick changelog between two releases:

```bash
git log --pretty=format:"- %s (%h)" v1.1.0..v1.2.0
```

Group entries under these headings:

- **Added** — new features
- **Changed** — behavior changes
- **Fixed** — bug fixes
- **Performance** — speed or memory improvements
- **Security** — masking, encryption, or credential handling changes

## CI automation

If you set up a release workflow, have it trigger on `v*` tags. It should:

- Run the full test suite
- Build the shaded JAR
- Generate release notes from commits between the previous tag and this one
- Publish the release with the JAR attached
